# 10 — Backlog para o Linear (G9)

> Documento 10 de 10. Desdobramento estruturado do blueprint para importação no **Linear.app**: módulos → **Projects**, capacidades → **Issues** (com critérios de aceite e prioridade), marcos da v1 → **Milestones**. A criação efetiva no Linear ocorre quando a integração (MCP/API) estiver conectada. As issues referenciam os IDs do BP (`PLT-/PES-/ACA-/MAT-/ROT-/AVA-/FIN-/COM-/POR-/LGP-/UI-/API-/INT-`).
>
> **Prioridades** (herdadas da análise §10): **P0** fundações transversais · **P1** valor imediato (perfil infantil) · **P2** condicionais resolvidos · **P3** por segmento (seriado).

---

## 1. Milestones da v1

| Milestone | Conteúdo | Depende de |
|---|---|---|
| **M0 — Fundações** | Multi-tenancy, auth, auditoria, parametrização, framework de lote, filas, eventos, design system + shell | — |
| **M1 — Cadastros e acadêmico base** | Pessoas, estrutura acadêmica configurável, situações, turmas, hub do aluno | M0 |
| **M2 — Matrícula e contratos** | Captação, matrícula online, requerimentos, assinatura, renovação | M1 |
| **M3 — Financeiro** | Título/cotas/planos, Regra de Cobrança, CNAB+PIX, régua, descontos, caixa, contábil, NFS-e, impedimentos | M1 |
| **M4 — Rotina e comunicação** | Diário de rotina, frequência, comunicados/circulares/ocorrências/fotos, caixa de saída, notificações, LGPD | M1 |
| **M5 — API e app** | Contrato unificado, adaptação do app, portais, virada coordenada | M1–M4 |
| **M6 — Pedagógico seriado (P3)** | Motor de avaliação, diário de classe, boletim, histórico, portais de aluno/professor | M1 |
| **M7 — Plataforma e go-live** | Backoffice do SaaS, white-label, Educacenso, central de relatórios, KPIs, hardening de segurança/a11y | M0–M5 |

---

## 2. Projects e Issues

> Formato de cada issue: **[ID-BP] título** — *(Prioridade · Milestone)* — critério de aceite resumido. No Linear, a descrição recebe o link para a seção do BP e os critérios de aceite completos (das invariantes de cada doc).

### Project: Plataforma & Administração (doc 01)
- **[PLT-01..03] Multi-tenancy Instituição→Unidade→Empresa** — *(P0 · M0)* — isolamento por tenant em todas as camadas; tenant por subdomínio; ciclo de vida do tenant.
- **[PLT-04] Onboarding de escolas (wizard)** — *(P2 · M7)* — provisiona tenant/unidade/empresa/seeds/usuário com simulação.
- **[PLT-05] Feature flags (3 escopos)** — *(P0 · M0)* — flags global/tenant/perfil auditadas.
- **[PLT-06] Impersonação auditada** — *(P0 · M0)* — substitui senha-mestre; permissão+justificativa+prazo+trilha.
- **[PLT-07] Monitoramento operacional por tenant** — *(P2 · M7)* — filas/rotinas/integrações; reprocessar/reexecutar.
- **[PLT-09/10] White-label (web + app tema dinâmico)** — *(P1 · M7)* — logo/favicon/paleta/subdomínio; contraste validado.
- **[PLT-11] Framework de operações em lote** — *(P0 · M0)* — filtrar→editar→simular→processar com log por item.
- **[PLT-12] Fila de processamento visível** — *(P0 · M0)* — estado/retry/log; rotinas como serviços nomeados.
- **[PLT-13] Eventos de domínio** — *(P0 · M0)* — catálogo publicável (anti-obstáculo v2).
- **[PLT-14] Auditoria onipresente** — *(P0 · M0)* — trilha por entidade; autoria não-humana prevista.
- **[PLT-15/16] Central de relatórios + KPIs acionáveis** — *(P1 · M7)*.
- **[PLT-18..21] Parametrização em camadas, cadastros auxiliares, campos dinâmicos, sequenciais** — *(P0/P1 · M0–M1)*.

### Project: Pessoas & Acesso (doc 02 §1)
- **[PES-01] Pessoa única (PF/PJ) + endereço + nome social + censitários** — *(P0 · M1)*.
- **[PES-02/03] Parente, atribuições, papéis, responsável legal (regra fixa)** — *(P0 · M1)*.
- **[PES-04] Colaborador + permissões + (seriado) formação/alocação/quadro** — *(P1/P3 · M1/M6)*.
- **[PES-06] Autenticação (social, recuperação, convite, troca de perfil)** — *(P0 · M0)* — token assinado, valida antes.

### Project: Estrutura Acadêmica (doc 02 §3)
- **[ACA-03/04] Estrutura configurável por escola + templates** — *(P0 · M1)*.
- **[ACA-05] Período letivo + calendário + feriados com efeitos/cópia** — *(P0 · M1)*.
- **[ACA-06..09] Curso, agrupamento/nível, série, turma (vagas reg./inclusivas, restrições)** — *(P1/P3 · M1)*.
- **[ACA-10/11] Disciplina, grade, itinerários, tipo de horário** — *(P3 · M6)*.

### Project: Matrícula & Captação (doc 02 §2/§4)
- **[ACA-01/02] Prospect, funil, agendamento de visita público** — *(P1 · M2)*.
- **[MAT-01/02] Enturmação + atendimento + override auditado** — *(P0 · M1)*.
- **[MAT-03] Matrícula online + procedimentos documentais** — *(P1 · M2)*.
- **[MAT-04/05] Efetivação por gatilho configurável + contrato + assinatura dual** — *(P1 · M2)*.
- **[MAT-06] Requerimentos (8 tipos, workflow, implementação automática)** — *(P1 · M2)*.
- **[MAT-07] Cancelamento (reativável, motivos seed)** — *(P1 · M1)*.
- **[MAT-08] Renovação anual em lote** — *(P1 · M2)*.
- **[MAT-09] Promoção/finalização em lote (seriado)** — *(P3 · M6)*.
- **[MAT-10] Situação do aluno (meta-modelo, seeds protegidos)** — *(P0 · M1)*.

### Project: Rotina Diária Infantil (doc 02 §6)
- **[ROT-01..03] Diário de rotina (refeições, sono, medicação, fralda, presença parcial, fechamento)** — *(P1 · M4)*.
- **[ROT-07] Ficha médica estruturada + edição pelo responsável** — *(P1 · M4)*.
- **[ACA-13] Frequência em dois modos por turma** — *(P1/P3 · M4/M6)*.

### Project: Avaliação (seriado) (doc 02 §8)
- **[AVA-01..03] Sistema de avaliação, fases, fórmulas (DSL), snapshot** — *(P3 · M6)*.
- **[AVA-04] Aprovação por nota E frequência** — *(P3 · M6)*.
- **[AVA-05] Conceitos, avaliação por relatório, competência, institucional** — *(P1/P3 · M6)* — *avaliação por relatório é P1 (infantil)*.
- **[AVA-06/07] Diário de classe + lançamento/confirmação/fila** — *(P3 · M6)*.
- **[AVA-08/09] Boletim (regras de publicação) + histórico escolar** — *(P3 · M6)*.

### Project: Financeiro (doc 03)
- **[FIN-01..04] Título + Serviço/Valor + Cota (seeds) + Plano de Pagamento** — *(P0 · M3)*.
- **[FIN-05] Regra de Cobrança (conceito de 1ª classe)** — *(P1 · M3)*.
- **[FIN-06] Régua de cobrança configurável (seeds)** — *(P1 · M3)*.
- **[FIN-07] Multa/juros nomeados (seed) + vencimento em dia útil + dia por responsável** — *(P1 · M3)*.
- **[FIN-08/09] Agente de cobrança + CNAB Santander/Sicoob + PIX** — *(P1 · M3)*.
- **[FIN-10] Descontos e bolsas** — *(P1 · M3)*.
- **[FIN-11/12/13] Negociação, pagamento a menor/maior, telecobrança** — *(P1 · M3)*.
- **[FIN-14/15] Caixa físico completo** — *(P2 · M3)*.
- **[FIN-16..19] Contas a pagar, plano de contas hierárquico, conciliação, DFC, exportação contábil** — *(P2 · M3)*.
- **[FIN-20/21] NFS-e via hub Focus (conector fiscal)** — *(P1 · M3)*.
- **[FIN-22..24] Impedimentos configuráveis (+disclaimers legais) + bloqueio contábil** — *(P1 · M3)*.

### Project: Comunicação (doc 04 §1)
- **[COM-01..03] Comunicados, circulares (confirmação/reenvio), ocorrências (tipadas)** — *(P1 · M4)*.
- **[COM-04] Fotos com dupla aprovação + consentimento de imagem** — *(P1 · M4)*.
- **[COM-05] Caixa de saída (trilha universal)** — *(P1 · M4)*.
- **[COM-06] Notificações internas + push + campanhas** — *(P1 · M4)*.
- **[COM-07] Templates de e-mail (seeds)** — *(P1 · M4)*.
- **[COM-08/09] Documentos/porta-arquivos + avaliação institucional** — *(P1/P2 · M4)*.

### Project: Portais & App (doc 04 §2/§3 + doc 06)
- **[POR-01] Permissões em camadas (atribuição×situação×config)** — *(P0 · M1)*.
- **[POR-02/03] Portal/App e portal web do responsável** — *(P1 · M5)*.
- **[POR-04] App do colaborador (rotina de sala)** — *(P1 · M5)*.
- **[POR-05/06] Portais de professor e aluno (seriado)** — *(P3 · M6)*.
- **[POR-07] Matriz de configuração dos portais** — *(P1 · M5)*.
- **[POR-08] App próprio (versão mínima, deep links, tema dinâmico)** — *(P1 · M5)*.
- **[API-01..23] Contrato unificado + adaptação do app (virada coordenada)** — *(P0/P1 · M5)*.

### Project: LGPD & Privacidade (doc 04 §4)
- **[LGP-01/02] Termos de consentimento e uso de imagem (aceite datado)** — *(P1 · M4)*.
- **[LGP-03] Dados sensíveis segregados, CPF mascarado, exportação condicionada** — *(P1 · M4)*.

### Project: Design System & Interface (doc 05)
- **[UI-01/02] shadcn/ui + orçamentos de CSS** — *(P0 · M0)*.
- **[UI-03..08] Shell de 5 áreas** — *(P0 · M0)*.
- **[UI-09] Tokens 3 camadas + white-label + dark mode + tokens de domínio** — *(P0 · M0)*.
- **[UI-10..13] Componentes compostos + hub 360º do aluno** — *(P0/P1 · M1)*.
- **[RQ-A11Y] Gate de acessibilidade no CI** — *(P0 · M0)*.

### Project: Integrações (doc 06 §4)
- **[INT-01] Conector de assinatura (Autentique + Clicksign)** — *(P1 · M2)*.
- **[INT-02] Conector fiscal (Focus NFe)** — *(P1 · M3)*.
- **[INT-03/04] Adaptadores bancários CNAB + PIX** — *(P1 · M3)*.
- **[INT-05..09] Push, e-mail/SMTP, SMS, CEP, login social** — *(P0/P1 · M0–M4)*.
- **[PUB-01..03] API pública de parceiros (OAuth2 + escopos)** — *(P1 · M5)*.

### Project: Compliance (docs 01/02/03)
- **[Educacenso] Exportação INEP (blocos 00–60) + dados censitários/códigos** — *(P1 · M7)*.
- **[Secretaria digital] Documentos escolares assinados (seriado)** — *(P3 · M6)*.

### Project: Visão v2 (doc 08) — não desenvolvida na v1
- **[V2-07] Anti-obstáculos na v1** — *(P0 · M0)* — rotinas como serviços, eventos, painel reservado, gate de aprovação, auditoria de agente. *(Issues da SIGE Intelligence em si ficam no backlog v2.)*

---

## 3. Observações de importação

- Cada issue herda, na importação, os **critérios de aceite** das invariantes do documento correspondente (ex.: as 14 invariantes do doc 03 viram critérios das issues FIN).
- As issues **P3 (seriado)** compartilham o Milestone M6 e podem ser desenvolvidas em paralelo às de operação infantil após M1.
- Issues **P0** são bloqueadoras: nenhuma issue de domínio (M1+) inicia antes das fundações M0 correspondentes.
- A coluna de prioridade mapeia para o campo *Priority* do Linear; os Milestones para *Projects/Milestones* conforme a convenção do workspace.
