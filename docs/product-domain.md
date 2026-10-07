# Domínio do Produto — Partiu

## O que é o Partiu

Partiu é uma plataforma brasileira que conecta **turistas** a **guias
turísticos locais**. A primeira operação comercial é a marca **"Partiu
Chapada"**, lançada na Chapada Diamantina (Bahia). O produto, porém, é
desenhado desde o início para expansão nacional sob o nome **"Partiu"** — a
arquitetura não pode ficar acoplada à Chapada Diamantina.

## Fluxo principal de domínio

```
Destino → Experiência → Data → Guias disponíveis → Reserva → Pagamento → Avaliação
```

Exemplo de uso:

1. Turista procura o destino/experiência "Cachoeira da Fumacinha".
2. Seleciona uma data.
3. O sistema encontra guias habilitados para essa experiência, naquela data.
4. Apresenta os guias disponíveis (preço, avaliação, idiomas, capacidade).
5. Turista compara e escolhe um guia.
6. Realiza a reserva → pagamento → (depois da experiência) avaliação.

## Modelo "Uber-like" (futuro — PR8)

Além da reserva agendada, o produto deve suportar, em etapa futura, um modelo
de disponibilidade imediata:

```
Localização do turista → guias online/próximos → solicitação → aceite → acompanhamento em tempo real
```

Isso implica, desde já, que o modelo de dados de Guia precisa comportar um
estado de disponibilidade em tempo real (online/offline, localização), além
da agenda programada — mesmo que essa funcionalidade só seja implementada na
PR8.

## Decisão de domínio central: Experiência não pertence a um guia

Uma **Experience** (ex.: "Cachoeira da Fumacinha") é uma entidade da
plataforma, independente de qualquer guia específico. Vários guias podem
estar associados à mesma experiência, cada um com seus próprios atributos:

```
Fumacinha (Experience)
├── João  — R$ 350 — guia associado (GuideExperience)
├── Carlos — R$ 320 — guia associado (GuideExperience)
└── Ana    — R$ 380 — guia associado (GuideExperience)
```

Ou seja, a relação entre `Guide` e `Experience` é **muitos-para-muitos**,
mediada por uma entidade de associação (`GuideExperience` ou equivalente) que
carrega os atributos que variam por guia:

- preço;
- agenda/disponibilidade para aquela experiência;
- idiomas oferecidos;
- capacidade de pessoas;
- avaliação (derivada das reviews daquele guia naquela experiência, ou do
  guia em geral — a decidir na PR7).

Essa decisão é estrutural e deve ser respeitada desde a modelagem inicial
(PR1), mesmo que a implementação completa de preços/agenda por guia só
avance nas PRs seguintes (PR2/PR3).

## Perfis principais

### 1. Turista

- cadastro/login;
- pesquisar destinos;
- pesquisar experiências;
- escolher data;
- encontrar guias;
- comparar guias;
- reservar;
- pagar;
- conversar com o guia;
- acompanhar reservas;
- avaliar.

### 2. Guia

- cadastro profissional;
- verificação documental;
- foto/perfil, bio, idiomas;
- regiões onde trabalha;
- experiências que realiza (via `GuideExperience`);
- preços, capacidade de pessoas;
- agenda/disponibilidade;
- ficar online/offline (modelo tempo real, PR8);
- receber, aceitar/rejeitar solicitações de reserva;
- acompanhar ganhos;
- avaliações recebidas.

### 3. Admin

- usuários;
- guias (aprovação/verificação);
- destinos;
- experiências;
- reservas;
- pagamentos e comissões;
- avaliações;
- indicadores operacionais.

## Entidades candidatas (visão preliminar, a confirmar na PR1)

- `Destination` — ex.: Chapada Diamantina. Não é exclusivo de uma marca/região fixa.
- `Experience` — ex.: Cachoeira da Fumacinha. Pertence a um `Destination`.
- `Guide` — perfil profissional do guia.
- `GuideExperience` — associação guia↔experiência com preço, idiomas, capacidade, agenda.
- `Booking` (Reserva) — turista + `GuideExperience` + data.
- `Payment` — associado a uma `Booking`.
- `Review` — avaliação associada a uma `Booking` concluída.
- `User` — base para `Tourist`/`Guide`/`Admin` (papéis).

Esta lista é preliminar e será refinada com dados reais na PR1. Não deve ser
tratada como schema definitivo.
