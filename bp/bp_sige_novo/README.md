# Blueprint do SIGE (novo) — v1

> Blueprint funcional do novo SIGE: plataforma SaaS de gestão educacional, da creche ao ensino médio. Descreve **o que** o sistema faz — domínio, regras, fluxos, integrações e requisitos — de forma independente de tecnologia, servindo de base para o SSD (Documento de Especificação de Software).
>
> Construído a partir de `bp/bp_sige_atual` (base fixa — nada descontinuado), `bp/bp_activesoft` (repertório de evolução) e `analise_comparativa.md` (matriz + 13 decisões D + 10 conflitos C + 11 garantias G validadas com o cliente).

## Estrutura

| Doc | Conteúdo | Uso no SSD |
|---|---|---|
| [00 — Visão e princípios](00-visao-e-principios.md) | Posicionamento, personas, 14 pilares, requisitos de qualidade, glossário | Contexto, escopo, NFRs |
| [01 — Plataforma e administração do SaaS](01-plataforma-e-administracao.md) | Multi-tenancy, backoffice do operador, white-label, flags, lotes, filas, eventos, auditoria | Requisitos de plataforma |
| [02 — Domínio acadêmico](02-dominio-academico.md) | Pessoas, estrutura configurável, matrícula, rotina infantil, frequência, avaliação, boletim, histórico | Requisitos funcionais acadêmicos |
| [03 — Domínio financeiro](03-dominio-financeiro.md) | Título/cotas/planos, Regra de Cobrança, CNAB+PIX, caixa, contábil, NFS-e, impedimentos | Requisitos financeiros |
| [04 — Comunicação e portais](04-comunicacao-e-portais.md) | Canais, caixa de saída, fotos, permissões em camadas, portais, app, LGPD | Casos de uso de comunicação/portais |
| [05 — Interface e design system](05-interface-e-design-system.md) | shadcn/ui, shell de 5 áreas, tokens, dark mode, gates de a11y/CSS | Especificação de UI/UX |
| [06 — API e integrações](06-api-e-integracoes.md) | Contrato unificado, migração do app, API pública, conectores | Especificação de API e conectores |
| [07 — Diretrizes tecnológicas](07-diretrizes-tecnologicas.md) | Constrangimentos duros e arquitetura de referência (não-vinculante) | Insumo para decisão de stack |
| [08 — Visão v2](08-visao-v2.md) | SIGE Intelligence (agentes) e diferimentos | Roadmap pós-v1 |
| [09 — Rastreabilidade da base](09-rastreabilidade-da-base.md) | Auditoria: cada item do sistema atual → onde vive no novo BP | Garantia de cobertura (G1) |
| [10 — Backlog para o Linear](10-backlog-linear.md) | Projects/Issues/Milestones com prioridades P0–P3 | Importação no Linear (G9) |
| [inventário da API atual](inventario_api_atual.md) | 135 endpoints legados catalogados (engenharia reversa) | Base do contrato novo e da rastreabilidade |

## Como usar

1. Comece pelo **00** para enquadrar visão, escopo e princípios.
2. Para cada módulo do SSD, cruze o doc de domínio (01–04) + **05** (UI) + **06** (API).
3. **07** orienta (sem fixar) a decisão de stack.
4. **09** é a garantia auditável de que nada da base se perdeu; **10** alimenta o Linear.

## Decisões-chave (resumo)

- **SaaS multi-escola, creche→médio**, configurável por tenant com seeds protegidos.
- **Nada da base é descontinuado** — só os defeitos do §12 da base são corrigidos (auditado no doc 09).
- v1 inclui: caixa físico, multi-empresa, retaguarda contábil, impedimentos configuráveis, secretaria digital, PIX+CNAB, assinatura dual (Autentique+Clicksign), NFS-e via Focus, API pública, white-label.
- Fora da v1 (mapeado no doc 08): chat, BI embarcado, biblioteca, filantropia, ranking, catraca, GFE, gerador SQL, garantidora, registro bancário online, e a **SIGE Intelligence** (camada de agentes — v2, com anti-obstáculos obrigatórios na v1).
