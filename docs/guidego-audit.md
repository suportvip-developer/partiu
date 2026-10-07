# Auditoria Técnica — GuideGo

Fonte: [bhagyabratagantayat/Guide-Go](https://github.com/bhagyabratagantayat/Guide-Go)
(clonado em modo shallow apenas para leitura/análise; **nenhum código-fonte
foi copiado para este repositório** — ver seção "Licença e Atribuição").

Esta auditoria é baseada em leitura direta do código (controllers, models,
routes, middleware, utils, frontend), não no README do projeto original.

## Stack real

- **Backend**: Node.js + Express 5, MongoDB via Mongoose 9
  (`backend/server.js`, `backend/config/db.js`). JWT para autenticação,
  Socket.IO 4.8 para tempo real, Helmet, `express-rate-limit`, sanitização
  contra XSS, Cloudinary (upload de imagens), Nodemailer/Brevo (SMTP),
  `otp-generator`. `razorpay` está instalado como dependência mas não é
  usado em nenhum controller (ver "Pagamentos").
- **Frontend**: React 18 + Vite 6 + Tailwind 4. Mapas via `leaflet` /
  `react-leaflet` (reais). i18n via `i18next`. Usa também
  `@supabase/supabase-js` no cliente — ou seja, **coexistem duas fontes de
  dados/autenticação** (MongoDB via backend Express + Supabase client-side).
  Esse acoplamento precisa ser resolvido/entendido antes da PR1; não deve
  ser herdado sem decisão explícita.
- Sem TypeScript. **Sem testes automatizados em nenhuma camada.**

## Entidades de domínio reais (confirmadas no código)

- `User` — role (user/guide/admin), senha com bcrypt, OTP para
  verificação/reset, campo `supabaseId`.
- `Guide` — 1:1 com `User` via `userId`. KYC (documento de identidade
  indiano "Aadhaar" em Base64 dentro do próprio documento Mongo), `isLive`,
  `status` (pending/approved/rejected/blocked), `location` como **string
  livre** (não normalizada), `pricePerHour/Day`.
- `Place` — name, lat/lng, category, city, `audioGuideText`. É a entidade
  mais próxima de "Destino", mas **sem relação com guias nem com uma
  entidade "Experience"**.
- `Booking` — userId, guideId, `location` (string livre), status
  (searching→accepted→ongoing→completed/cancelled), OTP de início de
  viagem, review embutido.
- `Review`, `Message` (chat), `Report` (denúncias).
- `Hotel`, `Restaurant`, `AudioGuide` — conteúdo turístico adicional
  específico do mercado indiano original, fora do domínio núcleo do Partiu.

**Decisão de domínio mais importante para a PR1**: o GuideGo **não tem uma
entidade "Experience" nem "Destination" desacoplada de guia** — hoje
"localização" é texto livre duplicado em `Guide` e `Booking`, sem
normalização. O modelo `Guide × Experience` muitos-para-muitos descrito em
[`product-domain.md`](./product-domain.md) **não existe no GuideGo** e
precisa ser desenhado do zero na PR1.

## Autenticação / Autorização

JWT (cookie httpOnly ou header Bearer) + middleware `authorizeRole(...)` por
papel, com bypass automático para `admin` (`backend/middleware/auth.js`).
Padrão correto e reutilizável como referência conceitual.

## Socket.IO — real vs. mock

- **Real e funcional**:
  - Chat persistido: `sendMessage` salva `Message` e emite para a room do
    destinatário, com controle de acesso baseado no status da reserva.
  - Matching em tempo real (`backend/utils/matchingManager.js`): notifica
    guias online/aprovados filtrando por idioma e localização, com timeout
    de 3 minutos. É, conceitualmente, o fluxo "Uber-like" que o Partiu
    planeja para a PR8.
- **Mock explícito** (não deve ser reaproveitado):
  - `backend/utils/socket.js` roda um `startSimulator()` que emite
    coordenadas falsas (jitter aleatório) em torno de pontos fixos de
    **Bhubaneswar, Odisha, Índia** (Tribal Museum, Udayagiri, Lingaraj) a
    cada 5 segundos.
- **Falha de design identificada**: o evento `updateLocation` real existe,
  mas faz `io.emit` global (broadcast para todos os clientes conectados),
  sem escopo por sala/reserva. Precisa ser corrigido antes de qualquer
  reaproveitamento.

## Geolocalização / Mapas — real vs. mock

Leaflet no frontend com lat/lng reais em `Place`, `Booking` e fluxo de SOS —
real. Não há integração com Google Maps ou Mapbox. O único componente mock é
o simulador de posição de guia descrito acima.

## Pagamentos — real vs. mock

**Mock/placeholder.** `razorpay` está instalado e citado em
`backend/config/env.js`, mas nenhum controller cria ordem, verifica
assinatura ou processa pagamento Razorpay de fato. `Booking.paymentMethod`
(cash/upi/card) e `paymentStatus` (pending/paid/refunded) são campos
atualizados manualmente, sem qualquer gateway real por trás.

## Admin — real vs. mock

Real e funcional: dashboard de estatísticas
(`adminController.getDashboardStats`), listagem/exclusão de guias, página
`AdminKycPage.jsx` para aprovação de KYC. É a parte mais madura/aproveitável
como referência conceitual.

## Frontend — stack e estrutura

React Router com ~35 páginas (`frontend/src/pages`), incluindo onboarding de
guia completo (`GuideOnboardingPage`, `GuideSetupPage`, `GuideVerifyPage`),
reservas, chat, SOS/emergência e um chat com IA (Gemini).

`frontend/src/data/` contém **dados mockados hardcoded e específicos da
Índia**: `mockHomeData.js`, `mockHotels.js`, `mockRestaurants.js`,
`mockAgencies.js`, `indiaKnowledge.js`. Todo esse conteúdo precisaria ser
substituído integralmente por conteúdo da Chapada Diamantina, caso o código
venha a ser reaproveitado.

## Scripts de debug/manutenção (dívida técnica)

`backend/{checkDB,check_arjun_status,cleanup,fix_arjun,fix_audio_data,
force_repair,repair,seed*,super_seed}.js` e os arquivos em `/scratch/*.js`
são scripts ad-hoc de correção manual de dados de produção — por exemplo,
`fix_arjun.js` atualiza um `userId` específico hardcoded, e
`force_repair.js` corrige em massa guias com KYC aprovado mas status
desalinhado. São **artefatos de debug/incidentes pontuais, não parte da
aplicação**. Recomendação: não portar para o Partiu; se o código-base vier a
ser adotado, descartar esses arquivos (documentando a decisão).

## Testes existentes

**Nenhum.** O script `test` do `package.json` (raiz e backend) é um
placeholder que falha propositalmente (`exit 1`). Não há Jest, Vitest, Mocha
ou Cypress em nenhuma dependência. O frontend tem apenas ESLint configurado
(lint, não teste automatizado).

## Scripts NPM

- Raiz: `build` (delega ao frontend), `test` (placeholder que falha).
- Backend: `start` (`node server.js`), `dev` (`nodemon`), `test` (placeholder).
- Frontend: `dev`, `build`, `lint`, `preview` (padrão Vite).

## Dependências principais e riscos

- Backend: `express@5`, `mongoose@9`, `jsonwebtoken@9`, `bcryptjs@3`,
  `socket.io@4.8`, `helmet@8`, `express-rate-limit@8` — versões recentes,
  sem red flags óbvios de CVE conhecido no momento desta auditoria.
- Frontend: `react@18`, `vite@6`, `tailwind@4`, `leaflet@1.9`, `axios@1.7` —
  stack atual, sem sinais de abandono.
- **Risco de segurança real a não herdar**: `backend/config/env.js` define
  `refreshTokenSecret` com fallback hardcoded
  (`'guidego_refresh_secret_key_2026'`) quando `REFRESH_TOKEN_SECRET` não
  está definido no ambiente. Qualquer deploy que esqueça essa env var fica
  com um segredo de refresh-token público, pois está no código-fonte aberto.

## Outros secrets encontrados

- `frontend/env_backup.env` (rastreado no git do GuideGo): contém
  `VITE_SUPABASE_URL` e `VITE_SUPABASE_ANON_KEY` reais de um projeto
  Supabase de terceiros. Risco baixo — é uma *anon key* pensada para uso
  público no cliente — mas é específica do projeto original e não deve ser
  reaproveitada.
- Nenhuma credencial de alta severidade encontrada: sem private keys, sem
  JSON de service account, sem connection string real de MongoDB (o
  `.env.example` só tem placeholders).

## Licença e atribuição — achado crítico

Há uma **inconsistência real e não resolvida** no repositório original:

- O `README.md` do GuideGo declara: *"Licensed under the MIT License - 2026
  GuideGo Team"*.
- O `package.json` (raiz e backend) declara `"license": "ISC"`.
- **Não existe nenhum arquivo `LICENSE` em lugar nenhum do repositório**
  (confirmado em toda a árvore, não só na raiz).
- Nenhum copyright holder individual é nomeado além de "GuideGo Team"; o
  autor real identificável no GitHub é o handle `bhagyabratagantayat`.

Duas declarações de licença diferentes, nenhuma delas formalizada em um
arquivo `LICENSE`, é juridicamente ambíguo. **Por decisão explícita nesta
PR0, nenhum código-fonte do GuideGo foi copiado para o Partiu** até essa
questão ser esclarecida diretamente com o autor original.

## O que podemos reaproveitar (conceitualmente, após esclarecimento de licença)

- Separação backend (Express/Mongo) / frontend (React/Vite).
- Padrão de autenticação JWT + autorização por papel.
- Fluxo de matching em tempo real via Socket.IO (base conceitual para a PR8).
- Estrutura de aprovação/KYC de guias no admin (base conceitual para a PR2).

## O que devemos substituir ou descartar

- Qualquer dado/conteúdo específico da Índia (`indiaKnowledge.js`, mocks de
  hotéis/restaurantes/agências, documento KYC "Aadhaar").
- Simulador de localização fake (`startSimulator`).
- Broadcast global de localização (`io.emit` sem escopo).
- Fallback hardcoded de secret em `config/env.js`.
- Scripts de debug/manutenção ad-hoc (`fix_arjun.js` e similares).
- Modelagem sem entidades `Experience`/`Destination` — precisa ser desenhada
  do zero conforme [`product-domain.md`](./product-domain.md).
- Dependência de pagamento (`razorpay`) não implementada — decidir gateway
  na PR6, sem herdar a dependência não usada.
