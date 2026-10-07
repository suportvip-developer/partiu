# Arquitetura — Partiu

## Status desta PR (PR0 — Foundation)

Esta PR **não introduz uma stack de código própria** nem copia código-fonte
do GuideGo (ver [`guidego-audit.md`](./guidego-audit.md) para o motivo:
ausência de arquivo `LICENSE` no repositório original). O conteúdo abaixo
descreve restrições e decisões arquiteturais que devem orientar a PR1 em
diante — é uma diretriz, não uma implementação.

## Princípio central: independência regional

A Chapada Diamantina é a **primeira operação comercial** ("Partiu Chapada"),
não uma premissa estrutural. Nenhuma entidade, rota, tabela ou regra de
negócio pode assumir implicitamente que existe apenas uma região/destino.

Na prática, isso significa:

- `Destination` é uma entidade de dados, não uma constante ou configuração fixa.
- Guias são vinculados a destinos/experiências por dados, não por código
  (nenhum `if region === 'chapada'` no domínio).
- Branding ("Partiu Chapada") é uma camada de apresentação/operação
  comercial sobre uma plataforma nacional ("Partiu"), nunca o contrário.

## Fluxo de domínio → implicações técnicas

```
Destino → Experiência → Data → Guias disponíveis → Reserva → Pagamento → Avaliação
```

- **Busca** precisa resolver "quais guias atendem esta experiência, nesta
  data" — isso é uma consulta sobre a associação `Guide × Experience`
  (muitos-para-muitos, ver [`product-domain.md`](./product-domain.md)), não
  uma relação 1:1 guia→experiência.
- **Disponibilidade** tem duas camadas que devem ser modeladas desde o
  início, mesmo que só uma seja implementada nas primeiras PRs:
  1. Agenda programada (PR3) — guia define janelas de disponibilidade.
  2. Presença em tempo real / "online agora" (PR8) — modelo estilo Uber,
     com localização do turista e dos guias.
- **Reserva → Pagamento → Avaliação** formam um pipeline sequencial por
  reserva; avaliação só deve ser possível após reserva concluída.

## O que aproveitamos do GuideGo como referência

A auditoria técnica (ver [`guidego-audit.md`](./guidego-audit.md)) identifica
o que no GuideGo é real (funcional) versus mock/placeholder, e o que pode
servir de referência arquitetural — ex.: separação backend/frontend, uso de
Socket.IO para tempo real, autenticação baseada em JWT. Nenhum código foi
copiado nesta PR; a referência é conceitual até a confirmação de
licenciamento.

## Decisões explicitamente adiadas

- Escolha final de banco de dados (o GuideGo usa MongoDB; não decidimos
  migrar para PostgreSQL nesta etapa — isso exige análise de impacto
  própria, conforme instrução explícita para a PR0).
- Escolha de gateway de pagamento real.
- Escolha de provedor de mapas/geolocalização.
- Framework de frontend definitivo (a avaliar junto da decisão de
  reaproveitamento ou não do GuideGo).

Essas decisões ficam para as PRs correspondentes (PR1 em diante), quando
houver escopo funcional concreto para justificá-las.
