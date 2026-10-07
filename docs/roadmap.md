# Roadmap

Este roadmap organiza a evolução do Partiu em PRs incrementais. Cada PR deve
ser pequena, auditável e reversível — sem grandes refactors acumulados.

A ordem reflete o fluxo de domínio principal:

```
Destino → Experiência → Data → Guias disponíveis → Reserva → Pagamento → Avaliação
```

| PR  | Nome                              | Escopo resumido |
|-----|------------------------------------|-----------------|
| PR0 | Foundation                         | Auditoria do GuideGo, estrutura do repositório, documentação inicial, git flow. Sem funcionalidades novas. |
| PR1 | Destinos e Experiências            | Modelagem de `Destination` e `Experience` (independentes de guia específico). |
| PR2 | Guias e Verificação                | Cadastro profissional de guias, documentos, verificação/aprovação. |
| PR3 | Agenda e Disponibilidade           | Agenda do guia, disponibilidade por data, capacidade. |
| PR4 | Busca e Matching                   | Turista busca destino/experiência/data → sistema encontra guias habilitados e disponíveis. |
| PR5 | Reservas                           | Fluxo de reserva (solicitação, aceite/rejeição, confirmação). |
| PR6 | Pagamentos e Comissões             | Processamento de pagamento e comissão da plataforma (sem gateway real nesta fase inicial). |
| PR7 | Avaliações e Reputação             | Avaliação do guia pelo turista (e vice-versa, se aplicável). |
| PR8 | Tempo real / Guias disponíveis agora | Modelo "Uber-like": localização do turista, guias online/próximos, solicitação em tempo real. |

## Status atual

Estamos na **PR0 — Foundation**. Nenhuma PR subsequente deve ser iniciada
antes da revisão e aprovação explícita da PR0.

## Observação sobre o código-fonte do GuideGo

A PR0 **não copiou código-fonte do GuideGo** para este repositório. O
`package.json` do GuideGo declara a licença `ISC`, mas o repositório original
não contém um arquivo `LICENSE`. Antes de reaproveitar código real do
GuideGo (estrutura de backend, modelos, rotas, frontend), é necessário
confirmar com o autor original que o código pode ser usado, modificado e
distribuído comercialmente, e obter/adicionar o arquivo `LICENSE`
correspondente no repositório de origem.

Até essa confirmação, o Partiu avança com documentação, arquitetura e domínio
próprios, usando o GuideGo apenas como **referência de arquitetura já
auditada** (ver [`guidego-audit.md`](./guidego-audit.md)), não como base de
código copiada.
