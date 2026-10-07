# Partiu

Partiu é uma plataforma que conecta **turistas**, **guias turísticos** e
**experiências**, do primeiro contato até a avaliação pós-passeio.

A primeira operação comercial é a marca **"Partiu Chapada"**, lançada na
**Chapada Diamantina — Bahia — Brasil**. A plataforma, no entanto, é
projetada desde a arquitetura para ser **nacional**: nenhuma regra de
negócio ou estrutura de dados assume que existe apenas uma região.

## Fluxo principal

```
Destino → Experiência → Data → Guias disponíveis → Reserva → Pagamento → Avaliação
```

Um turista procura um destino/experiência (ex.: "Cachoeira da Fumacinha"),
escolhe uma data, vê os guias habilitados e disponíveis para aquela
experiência naquela data, compara preço/avaliação/idiomas, reserva e paga.
Depois da experiência, avalia o guia.

Uma mesma experiência pode ter vários guias associados, cada um com seu
próprio preço, agenda, idiomas e avaliação — ver detalhes em
[`docs/product-domain.md`](docs/product-domain.md).

## Status atual: PR0 — Foundation

Este repositório está na sua primeira etapa: auditoria do projeto de
referência [GuideGo](https://github.com/bhagyabratagantayat/Guide-Go),
estrutura inicial e documentação. **Nenhum código-fonte do GuideGo foi
copiado para este repositório nesta PR** — o repositório original declara a
licença `ISC` no `package.json`, mas não contém um arquivo `LICENSE`. Até
essa questão ser esclarecida com o autor original, o GuideGo é usado apenas
como referência arquitetural já auditada.

Detalhes completos da auditoria: [`docs/guidego-audit.md`](docs/guidego-audit.md).

## Documentação

- [`docs/architecture.md`](docs/architecture.md) — princípios arquiteturais e restrições (independência regional, fluxo de domínio).
- [`docs/product-domain.md`](docs/product-domain.md) — domínio do produto, perfis (Turista/Guia/Admin), modelo de dados preliminar.
- [`docs/guidego-audit.md`](docs/guidego-audit.md) — auditoria técnica completa do projeto de referência GuideGo.
- [`docs/roadmap.md`](docs/roadmap.md) — roadmap de PRs (PR0 a PR8).

## Fluxo de Git

```
main
└── pr0-foundation
```

Todo trabalho desta etapa ocorre na branch `pr0-foundation`. Nenhuma
alteração é feita diretamente na `main`.

## Licenciamento deste repositório

A definir. Como nenhum código do GuideGo foi incorporado nesta PR, não há
ainda obrigação de atribuição de terceiros a cumprir — isso será revisitado
quando/se código de referência for reaproveitado.
