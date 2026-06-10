# Blueprint — Novo Sistema de Gestão Escolar

> Blueprint extraído por engenharia reversa do **ActiveSoft SIGA** (spec Janus em `_janus_spec/`),
> destinado à construção do **SDD de um novo sistema de gestão escolar**.
> Gerado em 2026-06-10 a partir de 318 telas capturadas, 246+ endpoints catalogados,
> 310 schemas inferidos, 931 release notes mineradas, 15 integrações e 18 canais de comunicação.

## Estrutura

| Doc | Conteúdo | Uso no SDD |
|---|---|---|
| [00 — Visão geral e princípios](00-visao-geral.md) | Posicionamento, escopo, personas, pilares arquiteturais, requisitos de qualidade, nível de confiança | Seções de contexto, escopo e NFRs |
| [01 — Domínio funcional](01-dominio-funcional.md) | 14 módulos com telas, regras de negócio, configurabilidade e perfis de usuário | Requisitos funcionais por módulo |
| [02 — Modelo de dados](02-modelo-de-dados.md) | Catálogo de entidades com campos, ER diagram (mermaid), padrões de modelagem, vocabulário de domínio | Modelo conceitual/lógico de dados |
| [03 — API e integrações](03-api-e-integracoes.md) | Mapa da API por domínio, API pública de parceiros, 15 integrações, aprendizados do changelog, NFRs | Especificação de API e conectores |
| [04 — Jornadas e comunicação](04-jornadas-e-comunicacao.md) | 8 jornadas críticas, matriz dos 18 canais escola↔família, anti-padrões e forças, confiabilidade da spec | Casos de uso e fluxos ponta-a-ponta |
| [05 — Design system e qualidade de UI](05-design-system-e-qualidade.md) | Tokens, paleta, componentes compostos, requisitos de a11y e CSS health | Especificação de UI/UX e gates de qualidade |
| [06 — Regras de negócio finas](06-regras-de-negocio-finas.md) | Validações, fórmulas, máquinas de estado, processos em lote, documentos emitidos, 20 invariantes | Regras de negócio detalhadas e critérios de aceite |

## Como usar

1. Comece pelo **00** para enquadrar visão, escopo e princípios do novo sistema.
2. Para cada módulo do SDD, cruze **01** (funcional) + **06** (regras finas) + **02** (entidades).
3. Use **04** para escrever os casos de uso ponta-a-ponta e **03** para o contrato de API/integrações.
4. **05** define o design system e os gates de qualidade (a11y/CSS) como requisitos não-funcionais.

## Avisos de confiabilidade

- 97% das afirmações marcadas na spec de origem são **observadas em captura** (🟣); as sequências ponta-a-ponta das jornadas de negócio e todo o lado família dos portais são **inferidos** (🟡) — sinalizado em cada documento.
- Os portais aluno/responsável/professor **nunca foram capturados por dentro** (apenas a visão admin).
- Valores concretos citados (percentuais, datas, planos) são do tenant observado (CEMP/Belém-PA) e servem como **exemplo de parametrização**, não como default do novo sistema.
