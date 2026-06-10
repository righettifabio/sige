# 00 — Visão Geral e Princípios do Novo Sistema

> Síntese estratégica do blueprint. Fontes: `produto-estrategico.md`, `confidence-report.md`,
> `review-senior.md` e consolidação dos dossiês 01–06.

---

## 1. O que o sistema de referência é

O ActiveSoft SIGA é o **ERP escolar multi-tenant do Grupo Arco Educação**, voltado ao colégio
particular brasileiro de pequeno e médio porte (200–1500 alunos), cobrindo o ciclo completo:

```
captação → ficha de inscrição pública → matrícula online → contrato (assinatura eletrônica)
→ enturmação → diário/frequência/notas → boletim/histórico → cobrança (boleto/PIX/ISAAC)
→ comunicação família (18 canais) → compliance (Educacenso, NFS-e, LGPD, exportação contábil)
```

**Proposta de valor central:** transformar a secretaria de papel ("Excel + boleto manual +
bilhete impresso") em back-office digital com cobrança automatizada e comunicação multicanal,
sem a escola precisar de TI interno.

**Diferenciais comprovados no produto de referência (a preservar):**
- BR-first com BNCC, Novo Ensino Médio e Educacenso embutidos
- Cobrança PIX/boleto registrado nativa + garantidora de recebíveis (ISAAC)
- Hub 360° do aluno (um registro, 7+ visões em abas)
- Caixa de saída como audit trail universal da comunicação
- Configurabilidade profunda por escola (motor de avaliação, vocabulários, boletim)
- API pública para parceiros (editoras, catracas, plataformas de matrícula)

---

## 2. Escopo funcional (14 módulos)

1. **Alunos** — cadastro mestre 360° (cadastro, ficha médica, enturmação, ocorrências, histórico, descontos, cobranças)
2. **Responsáveis** — PF/PJ, papéis financeiro × pedagógico, acesso ao portal derivado do vínculo
3. **Colaboradores** — cadastro, formação, alocação didática (que ativa o acesso do professor), quadro de horários
4. **Estrutura acadêmica** — Período → Curso → Série → Turma → Disciplina/Grade, itinerários formativos
5. **Diários de classe** — aulas, conteúdo, tarefas, chamada (com ~25 parâmetros por escola)
6. **Avaliação/Notas/Boletim** — motor fórmula-dirigido com fases, aprovação nota E frequência, fila assíncrona, boletim hipercustomizável, histórico escolar
7. **Comunicação** — comunicados, chat, templates de e-mail, push, caixa de saída
8. **Financeiro** — títulos, boletos/PIX/CNAB, régua de cobrança, negociação, caixa físico, contas a pagar, contábil, NFS-e, filantropia
9. **Captação & Matrícula** — funil comercial, ficha pública, pré-matrícula dirigida por pagamento, procedimentos documentais, contrato/assinatura
10. **LGPD/Termos** — consentimento, uso de imagem, privacidade, com aceite datado
11. **Calendário & Agendamentos** — feriados com efeitos (financeiro/biblioteca), reuniões marcadas pelo responsável
12. **Portais** (aluno/responsável/professor) — matriz de permissões por persona × situação do aluno × recurso
13. **Configurações & Administração** — multi-unidade, multi-empresa, auditoria, relatórios, gerador de consultas, filas, Educacenso
14. **Integrações** — 15 conectores (pagamentos, comunicação, editoras, censo, fiscal, BI, suporte)

Detalhe completo no **doc 01**; regras finas no **doc 06**.

---

## 3. Personas (9)

| # | Persona | Característica-chave |
|---|---|---|
| 1 | Admin/Secretaria | acesso total, opera logado em uma unidade |
| 2 | Operador de caixa/tesouraria | abertura/fechamento nominal, permissões de liquidação |
| 3 | Coordenador | recebe fluxo de confirmação de notas, assina boletim |
| 4 | Professor | acesso **derivado da alocação** turma×disciplina; portal próprio |
| 5 | Responsável | acesso **derivado do vínculo** com aluno ativo elegível; papéis financeiro vs pedagógico |
| 6 | Aluno | boletim, atividades, carteirinha digital, porta-arquivos |
| 7 | Vendedor de captação | funil comercial, WhatsApp do consultor |
| 8 | Suporte do fornecedor | usuários internos no ambiente do cliente → exigir impersonação auditada |
| 9 | Parceiro de API | não-humano; hoje Bearer por instituição → migrar para OAuth2 com escopos |

---

## 4. Pilares arquiteturais (derivados da observação)

1. **Multi-tenancy em 3 níveis** — Instituição → Unidade (tenant operacional) → Empresa (CNPJ de faturamento). Tenant resolvido por subdomínio/claim, **nunca por path da URL** (anti-padrão observado: `/api/v1064840/aluno_ficha_medica/103/`).
2. **Status como meta-modelo** — "Situação do aluno na turma" é cadastro configurável pela escola, mapeado a domínios fixos de sistema (ativo/inativo), acadêmico (aprovado/reprovado), Educacenso e flags de comportamento (digita nota, assina contrato). Quase todo módulo filtra por ela. **É a peça de estado central do domínio.**
3. **Motor de avaliação fórmula-dirigido** — fases de nota encadeadas por fórmulas (DSL), aprovação = nota E frequência, recuperação roteada por fórmula, cálculo assíncrono em fila com confirmação professor → coordenação. **Não hardcodar bimestres.**
4. **Acoplamento acadêmico↔financeiro por desenho** — pagamento confirma pré-matrícula; inadimplência gera impedimento que bloqueia boletim/matrícula/caixa/portal (por canal, configurável); inativação na turma cancela títulos; série carrega o serviço de mensalidade.
5. **Impedimento como mecanismo transversal** — pendências (financeiras, biblioteca, documentais, ocorrências) geram bloqueios configuráveis por canal, com badges (OBS/FNC/COB/BIB/PDM) e overrides auditados.
6. **Operações em lote como cidadão de primeira classe** — framework genérico de bulk-operations com etapa de **simulação**, fila assíncrona visível, log por item e retry.
7. **Auditoria e trilha onipresentes** — auditoria por entidade em toda tela de configuração/operação financeira; caixa de saída loga toda comunicação; overrides sempre com usuário + justificativa + data.
8. **Configurabilidade em camadas** — parâmetros globais → por unidade → por série×período → por turma → por diário; a regra mais específica prevalece. Feature flags em 3 escopos (global/tenant/perfil).
9. **Soft delete e ciclo de vida** — nada se exclui: inativa-se com motivo, comentário e destino (ex.: escola de transferência).
10. **Encadeamento temporal** — `proximo_periodo` e `proxima_serie` como listas ligadas que habilitam rematrícula e promoção em lote.

---

## 5. Requisitos de qualidade (corrigir as dívidas do legado)

O gate de review do sistema de referência **reprovou 318/318 telas**. O novo sistema nasce com:

**Segurança/LGPD**
- Cookies `Secure` + `HttpOnly` + `SameSite`; sessão fora de cookie legível (legado: `auth_jwt` sem Secure em 318 telas)
- Zero PII em URLs, docs de API e exemplos de swagger (legado: dados reais de menores no swagger público)
- Analytics com consentimento e sem session-replay em telas com PII de menores (legado: GA + Clarity + Hotjar simultâneos)
- Sem rotas `/dev/` em produção; SQL ad-hoc apenas com whitelist e auditoria
- Política de senha moderna (legado: máximo de 10 caracteres)
- Endpoints de dados sensíveis com escopo OAuth próprio e auditoria por token

**Acessibilidade (gate de release: zero violações critical/serious)**
- Todo campo com label programático; todo controle icon-only com `aria-label`
- Componentes compostos seguindo WAI-ARIA APG (legado: 1 popover global defeituoso contaminou ~300 telas)
- `<html lang="pt-BR">`; axe-core em browser real no CI

**Arquitetura de informação**
- Jornadas de negócio como fluxos de primeira classe (wizards/checklists), não 318 telas-ilha (legado: 66 arestas de navegação em 318 telas)
- Governança de rotas/nomenclatura (legado: `admin`/`administrador`/`admnistracao`, telas com nome enganoso)
- Estados empty/erro/carregando distintos e explícitos

**Stack e CSS**
- Stack de UI única com tokens DTCG em 3 camadas (legado: 5 frameworks empilhados, 10.462 `!important`, 1.267 cores, z-index 2147483647)
- Sem dependências EOL (legado: React 16.14, CKEditor 4, moment.js)

---

## 6. Nível de confiança deste blueprint

| Categoria | Status |
|---|---|
| Existência e estrutura das 318 telas, campos, endpoints, parâmetros | 🟣 **Observado em captura** |
| Hubs de navegação (aluno 7 abas, colaborador 6 abas) | 🟣 Observado (confidence 0.90) |
| Valores de parametrização citados (multa 2%, régua às 14h, fases Fund II…) | 🟣 Observados — mas são **do tenant CEMP**, exemplos e não defaults |
| Sequências ponta-a-ponta das jornadas de negócio | 🟡 **Inferidas** de rotas, endpoints e release notes |
| Lado família dos portais (responsável/aluno/professor por dentro) | 🟡 Inferido — **nunca capturado** |
| Entrega efetiva de e-mail/SMS/push | 🟡 Inferida de release notes e schemas |
| Renderização condicional do financeiro per-aluno | 🔴 Hipótese declarada na spec |

**Pendências para fechar lacunas antes de detalhar o SDD dessas áreas:** captura autenticada
do portal do responsável; form-probe do POST de comunicados; verificação de webhooks externos.
