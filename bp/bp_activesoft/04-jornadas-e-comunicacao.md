# 04 — Jornadas de Usuário e Comunicação

> **Fontes:** `_janus_spec/ui/jornadas.md`, `_janus_spec/ui/comunicacao-familia.md`,
> `_janus_spec/ui/review-senior.md`, `_janus_spec/confidence-report.md`
> (complementadas por `_janus_spec/ui/clusters/*` e `_janus_spec/produto-estrategico.md`).

**Aviso de método (importante para o SDD):** o `jornadas.md` do Janus mapeou **318 telas, mas apenas 66 arestas e 50 "jornadas"**, todas do tipo *navigation* (2–3 passos) e concentradas em dois hubs: o **prontuário do aluno** e o **prontuário do colaborador**. Ou seja: a captura confirma *as telas* de cada jornada de negócio, mas **a sequência ponta-a-ponta das jornadas de negócio é reconstruída por inferência** (nomes de rotas, endpoints, release notes e o mapa de fluxo do `comunicacao-familia.md`). Cada jornada abaixo está marcada com 🟣 (observado em captura) ou 🟡 (inferido).

---

## 1. Jornadas críticas

### J1. Captação → Matrícula → Contrato (funil comercial) — 🟡 inferida (telas 🟣 observadas)

- **Atores:** Lead (responsável interessado), Vendedor de captação (`USUARIO_VENDEDOR_CAPTACAO` é tipo de usuário declarado no bundle), Secretaria, Responsável financeiro.
- **Passos e telas:**
  1. **Captação do lead** — `/captacao/` e `/captacao_alunos/` (tela capturada; contém botão isolado "WhatsApp do consultor" para contato manual com o lead).
  2. **Ficha de inscrição pública** — `/ficha_inscricao_novatos/`: a escola distribui um link, o responsável preenche de fora do sistema.
  3. **Matrícula online** — `/matricula_online/` (tela de *parametrização* do fluxo) + `/matricular/`, `/matriculas/`, `/procedimento_matricula/`.
  4. **Confirmação** — `/confirmacao_matricula/`; enturmação do aluno em `/alunos/<id>/enturmacao/` 🟣.
  5. **Contrato** — `/contrato/`, `/contratos/`, `/contratos_financeiros/`, `/situacao_contratos/`; assinatura digital via `/assinatura_eletronica/`.
  6. **Termos LGPD** — `/termo_consentimento/`, `/termo_uso_de_imagem/`, `/termo_uso_privacidade/`.
  7. **Rematrícula (via ISAAC)** — `/status_rematricula_isaac/`, `/status_matricula_plano_pagamento_isaac/`.
- **Pontos de decisão:** aprovação da inscrição pela escola; escolha do plano de pagamento (`/plano_pagamento/`); descontos/bolsas (`/aluno_turma_bolsa/`, `/alunos/<id>/desconto_autorizado/` 🟣); aceite dos termos pelo responsável.
- **Estados:** lead → inscrito → matrícula confirmada → contrato assinado → aluno ativo (`/situacao_aluno/` é cadastro configurável de situações; lista de alunos filtra por `?status=ativo` 🟣).
- **Exposição como API:** `POST /api/v0/financeiro/gerar_cobranca/` é público — sugere que plataformas de matrícula de terceiros disparam a cobrança da matrícula.

### J2. Ciclo de cobrança (mensalidade → boleto → baixa → inadimplência) — 🟡 inferida (telas 🟣)

- **Atores:** Secretaria/Financeiro da escola, ISAAC Pagamentos (operador externo de garantia), Responsável financeiro, bancos (BB, CAIXA, Itaú, PJ Bank).
- **Passos e telas:**
  1. **Configuração** — `/parametros_financeiro/`, `/plano_pagamento/`, `/forma_recebimento/`, `/calculo_multa_juros/`, `/plano_contas/`, `/centro_resultado/`.
  2. **Geração da cobrança** — `/alunos/<id>/cobranca/` 🟣 e visão global `/contas_receber/` 🟣 (título real capturado: **"Títulos e operações"**, com filtros, "Mais filtros", "Consultar" e **"Operações em lote"**). Endpoint: `POST /api/v0/financeiro/gerar_cobranca/`.
  3. **Registro do boleto** — filas assíncronas: `/titulos_pendencia_registro_boleto/`, `/titulos_isaac_fila_registro/`, `/titulos_pendencia_registro_boleto_isaac_pagamentos/`, `/fila_processamento/`, `/processamentos/`.
  4. **Entrega à família** — boleto por **email do responsável** (template HTML personalizável com variáveis `[NOMEALUNO]`, `[PARENTESCO_FILIACAO_1]` etc.) + portal do responsável + `GET /api/v0/informacoes_boleto/`; segunda via em `/aluno_segunda_via/`.
  5. **Cobrança automática da inadimplência** — `/regua_cobranca_automatica/` (régua com parâmetro mobile via ClassApp) + `/agente_cobranca/` + `/cobranca_ativa/`, `/cobranca_registrada/`, `/lancamentos_de_cobranca_em_aberto/`.
  6. **Recebimento e baixa** — `/baixa_pagamento/`, `/recebimento_por_cartao/` (+ `--operadoras`), `/recebimentos_por_pix/` (`ACESSO_PIX_IMEDIATO`, `ACESSO_BOLECODE_ITAU`), `/caixa/`, `/cheques/`, `/cheque_custodia/`.
  7. **Pós-recebimento** — `/recibos/`, `/nota_fiscal_servico/`/`/nfse/`, `/conciliacao_bancaria/`, `/exportacao_contabil/`, relatórios (faturamento, recebimento, descontos, ticket médio, cancelamentos).
- **Pontos de decisão críticos:**
  - **Cobrança garantida por ISAAC não é editável na escola** — release note literal: *"A cobrança que você está tentando editar é garantida pelo isaac e deve ser cancelada pela operação de inativar o aluno na turma"*. Cancelamento em lote via `/cancelar_titulos_isaac_lote/` (única aresta capturada fora dos hubs: chega lá a partir de `disciplina/<id>/atualizar` via botão "Cancelar" — sinal de navegação confusa, ver §3).
  - Aplicação de desconto: exige passar por `/alunos/<id>/desconto_autorizado/` 🟣 e `/descontos/` 🟣.
- **Estados do título:** gerado → pendente de registro (fila) → registrado → enviado → em aberto/vencido (régua) → pago/baixado → conciliado/cancelado.

### J3. Lançamento de notas → fechamento → boletim — 🟡 inferida (telas 🟣)

- **Atores:** Professor (portal próprio `/portal/professor/`), Coordenação, Secretaria, Responsável/Aluno (consumo).
- **Passos e telas:**
  1. **Configuração pedagógica (pré-ano)** — `/sistema_avaliacao/` 🟣, `/formula_composicao/`, `/fase_nota/`, `/nota_conceito/`, `/conceito/`, `/bimestre/`, `/periodos/`, `/configuracoes_serie_periodo/`, `/definicoes_boletim_serie/` 🟣, `/configuracoes_avaliacoes_competencia/`.
  2. **Lançamento** — `/planilha_notas_faltas/`, `/planilha_avaliacao_competencia/`, `/notas/`, `/avaliacoes/`; correções em massa via `/alterar_notas_lote/` e `/reprocessamento_notas/`.
  3. **Confirmação professor → coordenação via "Messenger interno"** — configurado em `/portal_web_parametros/envio_mensagem/` 🟣 (nome enganoso; é o fluxo de notas): *"O sistema deve ativar o uso do Messenger para o processamento de notas"*, *"O professor poderá efetuar a confirmação de notas da turma pelo portal"*, **"Bloquear boletim com impedimento"** — ponto de decisão de gate de publicação.
  4. **Decisões de aprovação** — `/reprovacao_por_faltas/`, `/reprovar_aluno_recuperacao/`, `/ranking_de_alunos/`.
  5. **Publicação do boletim** — `/boletim/`, `/boletim/novo/`, `/boletim/regras-publicacao/` 🟣 (existe tela dedicada de *regras de publicação* — o novo sistema deve manter esse gate explícito); consumo pela família via `GET /api/v0/detalhe_boletim/` + histórico em `/alunos/<id>/historico/` 🟣.
  6. **Encerramento do ano** — `/finalizar_turmas/`, `/promocao_aluno_turma/`, `/inativar_enturmacoes_lote/`, `/exportacao_educacenso/`, `/historico_padrao/` e `/configuracao/historico_escolar/` (com códigos BNCC no histórico do ensino médio).
- **Estados da nota:** lançada → confirmada pelo professor (Messenger) → processada/reprocessada → com/sem impedimento → boletim publicado → histórico consolidado.

### J4. Comunicado → família (broadcast) — 🟣 telas e campos observados; entrega 🟡 inferida

- **Atores:** Secretaria/Coordenação (autor), Responsáveis e alunos das turmas-alvo (leitores).
- **Passos e telas:**
  1. **Criação** — `/comunicados/cadastrar/` 🟣 (h1 capturado: "Cadastrar comunicado"). Campos observados no DOM:
     | Campo | Tipo |
     |---|---|
     | Título | text |
     | **Visível na área de Comunicados?** | toggle (controla visibilidade para a família) |
     | Data de publicação | date |
     | Texto | rich text/textarea (CKEditor 4) |
     | **Turmas para exibição** | multi-select (público-alvo) |
     | Situação do aluno na turma | filtro adicional de audiência |
  2. **Publicação** — listagem `/comunicados/` 🟣, detalhe `/comunicados/<id>/` 🟣, visualização `/comunicados/visualizar/`. API: `GET /api/v1/comunicados/`.
  3. **Entrega multi-canal (inferida):** visível no portal do responsável/app + disparo de email automático + push via ClassApp/App SIGA.
  4. **Auditoria** — tudo enviado fica logado na `/caixa_saida/` 🟣 (entidade `Mensagem`, com busca e paginação).
- **Pontos de decisão:** publicar agora vs. agendar (data de publicação); audiência por turma + situação do aluno; toggle de visibilidade.
- **Estados:** rascunho → publicado → visível/oculto para a família. **Empty state observado:** "Nenhum comunicado encontrado que atende aos filtros utilizados" (renderizado como `alert-warning` — anti-padrão de UI, ver §3).

### J5. Diário de classe (frequência, conteúdo, ocorrências) — 🟣 parcialmente observada

- **Atores:** Professor, Coordenação, Responsável (leitura via portal).
- **Passos e telas:**
  1. **Lista de diários** — `/diarios/` 🟣 + `/diarios/configuracoes/` 🟣.
  2. **Diário da turma** — `/diario/<id>/` 🟣 (capturado em 2 instâncias, ids 1982/1983): roster completo da turma com link de cada aluno para `/alunos/<id>/detalhe/` — a captura registrou ~20 nomes reais de alunos navegáveis (PII exposta até na spec, ver §3).
  3. **Frequência** — por aula no diário + lançamento em massa `/diarios/frequencia_em_lote/` 🟣; API pública `GET /api/v0/diario_frequencia/`.
  4. **Conteúdo lecionado** — `/conteudos/`, `/atividades/`, `/tarefas/`.
  5. **Ocorrências/observações** — `/alunos/<id>/observacoes/` 🟣 (API `GET /api/v1/aluno_observacao/`; pública `GET /api/v0/lista_aluno_ocorrencia/`); dashboard de ocorrências `/ocorrencias_alunos/?fromdashboard=1` 🟣.
- **Visibilidade para a família:** ocorrências e diário expostos no portal do responsável (inferido, dependente de configuração) — *"escola registra incidente uma vez, pai vê no app, conversa fica documentada"*.
- **Estados:** aula registrada → frequência lançada (individual ou lote) → ocorrência registrada → visível no portal; reprovação por faltas como desfecho extremo (`/reprovacao_por_faltas/`).

### J6. Hub 360° do aluno — 🟣 a única jornada de navegação efetivamente capturada (j-001 a j-042, confidence 0.90)

- **Padrão:** prontuário com abas persistentes. De **qualquer** aba — `detalhe`, `cobranca`, `desconto_autorizado`, `enturmacao`, `ficha_medica`, `historico`, `observacoes` — o usuário navega para `Descontos` e `Turmas`.
- **Significado para o blueprint:** a secretaria opera *centrada no aluno*: um registro, sete+ visões (cadastro, financeiro, descontos, enturmação, saúde, histórico acadêmico, ocorrências). É o componente de maior valor comprovado do sistema atual — **preservar**.
- **Vínculo com responsáveis:** `/responsaveis/<id>/detalhe|alunos_vinculados|observacoes/` 🟣 + `/responsavel_financeiro/`, `/responsavel_aluno_vinculado/`, `/tipo_responsavel/` — modelo N:N aluno↔responsável com papel financeiro destacado.

### J7. Hub do colaborador/professor — 🟣 capturada (j-043 a j-050)

- Abas: `detalhe`, `disciplinas`, `disciplinas_alocadas`, `escolaridade`, `obs`, `quadro_horarios` (ids 102 e 109 capturados). Navegação entre abas via `a.nav-link`.
- Alocação docente: `/professores_turmas/` (filtrável por período letivo, `?periodoid=35` 🟣).
- **Bug de navegação observado:** das abas do colaborador 109, o link "Disciplinas" leva para o colaborador **102** — aresta capturada aponta cruzamento de IDs; candidato a defeito real do sistema atual.

### J8. Acesso da família ao portal (provisionamento) — 🟡 inferida (telas 🟣)

- **Passos:** controle de quem acessa em `/controle_de_acesso/portal/resps/` 🟣 (+ `/situacao_acesso_portal/`, APIs `GET /api/v1/colaborador/situacao_acesso_portal/<id>/` e `GET /api/v0/acesso/vinculo_aluno_responsavel_liberado/`) → envio de credenciais em massa `/envio_de_senhas/` (email transacional) → login em `/login/?next=/portal/<tipo>/` → recuperação em `/recuperar_senha/`.
- **Arquitetura confirmada por sondagem:** o portal do responsável/aluno/professor **roda no mesmo domínio do admin** (`siga04.activesoft.com.br`), mesma SPA, roteado client-side por claim do JWT. Subdomínios (`pais.*`, `responsavel.*` etc.) não existem. Não há canal escola↔família fisicamente isolado.

---

## 2. Matriz de comunicação escola ↔ família (18 canais)

Fonte: `ui/comunicacao-familia.md` (consolidado de 75 telas canônicas + 144 endpoints UI + 40 endpoints da API pública SigaWeb). **Limitação assumida na própria fonte:** toda evidência vem do perfil admin; os portais não foram capturados por dentro.

| # | Canal | Direção | Gatilho | Canal técnico | Audit trail / evidência |
|---|---|---|---|---|---|
| 1 | **Comunicados** (broadcast por turma) | Escola → Família | Publicação manual com data; audiência = turmas + situação do aluno | In-app/portal + email automático (inferido) | `GET /api/v1/comunicados/`; histórico na listagem; envio logado na Caixa de saída |
| 2 | **Conversas do App** (chat 1:1/grupo) | **Bidirecional** | Mensagem de qualquer lado | Chat in-app (`/mensagens/`) | `GET /api/v1/mensagem/conversas/` — atenção: filtro `min_mensagens_nao_lidas=1` hardcoded no frontend |
| 3 | **ClassApp** (integração) | Escola → Família | Eventos do SIGA (comunicado, cobrança) | Push + chat no app ClassApp | `/integracoes/classapp/` + `/log_classapp/` (log dedicado); token de push em release notes |
| 4 | **App SIGA próprio (push)** | Escola → Família/Aluno | Disparo via API | Push notification | Release notes: "API de Envio de Push para o APP SIGA" |
| 5 | **Caixa de saída de mensagens** | — (meta-canal) | Qualquer envio | Log centralizado | **É o próprio audit trail**: tela `/caixa_saida/`, entidade `Mensagem` — "sempre se sabe quem recebeu o quê" |
| 6 | **Aviso na tela inicial** (banner global) | Sistema → Todos | Mensagem sistêmica do produto | Banner no app shell | `GET /api/v1/aviso_tela_inicial/` chamado em todas as 75 telas |
| 7 | **Portal do responsável** | Escola → Família | Login do responsável | Portal web (`/portal/responsavel/`, mesma SPA, claim JWT) | Acesso controlado por `/controle_de_acesso/portal/resps/` |
| 8 | **Portal do aluno** | Escola → Aluno | Login do aluno | Portal web (`/portal/aluno/`) | idem |
| 9 | **Avaliação institucional** (pesquisa/NPS) | **Família → Escola** | Pesquisa publicada pela escola | Form no portal/app | `/avaliacao_institucional/` + `--minhas_avaliacoes` + `/avaliacao_relatorio/` |
| 10 | **Email com templates personalizáveis** | Escola → Família | Eventos (boleto, comunicado, boletim) | Email HTML com variáveis e cabeçalho/rodapé configuráveis | `/texto_personalizado/`, `/mensagem_config_smtp/` |
| 11 | **SMS** | Escola → Família | Inferido: disparo automático por evento | SMS via gateway (provedor em nível de produto) | Campo `celular_para_envio_sms` no schema público do Aluno; **sem UI de envio detectada** |
| 12 | **WhatsApp do consultor** | Escola (captação) → Lead | Contato manual no funil de captação | WhatsApp manual (botão isolado em `/captacao_alunos/`) | String "WhatsApp do consultor" no bundle; sem trilha no sistema |
| 13 | **Boletim digital** | Escola → Família | Publicação do boletim (após regras/gates) | Portal + PDF/digital + email | `GET /api/v0/detalhe_boletim/` |
| 14 | **Diário de turma + ocorrências** | Escola interna → visível pela família | Registro de ocorrência/observação | View no portal do responsável | `GET /api/v1/aluno_observacao/`, `GET /api/v0/lista_aluno_ocorrencia/` |
| 15 | **Cobranças per-aluno (boleto)** | Escola → Família | Geração manual ou agendada de cobrança | Email do responsável + portal (+ SMS?) | `POST /api/v0/financeiro/gerar_cobranca/`, `GET /api/v0/informacoes_boleto/` |
| 16 | **Régua de cobrança automática** | Escola → Família | Vencimento/atraso de título | Email + push via ClassApp (parâmetro mobile confirmado) | `/regua_cobranca_automatica/`; release notes |
| 17 | **Envio de senhas** | Escola → Família | Provisionamento de acesso ao portal | Email transacional | `/envio_de_senhas/` |
| 18 | **Termos LGPD** (consentimento + uso de imagem) | Escola → Família (com aceite de volta) | Matrícula/atualização de termos | Aceite digital no portal | `GET /api/v1/termo_uso_privacidade/`, `/api/v1/unidade_termo_uso_imagem` |

**Achado estrutural relevante para o novo sistema:** foram sondadas 35+ rotas de configuração de canais (`/template_email/`, `/canal_envio/`, `/smtp/`, `/provedor_sms/`, `/push/`, `/whatsapp/`...) e **nenhuma existe no nível do cliente** — templates e provedores são gerenciados em nível de produto Activesoft, não da escola. O novo sistema deve decidir explicitamente o que é configurável por tenant.

---

## 3. Achados do review sênior — anti-padrões a corrigir e forças a preservar

O `review-senior.md` é um gate automatizado: **decisão 🔴 FAIL** — 318 telas revisadas, **1.394 findings** (318 errors, 765 warns, 311 infos), **100% das telas com ao menos 1 error**.

### Anti-padrões que o novo sistema DEVE corrigir

**Segurança / compliance (errors do gate):**
1. **`insecure-cookie` em 318/318 telas** — 8 a 11 cookies sem flag `Secure` em HTTPS, incluindo `auth_jwt`, `sessionid` e `csrftoken`. No novo sistema: cookies `Secure` + `HttpOnly` + `SameSite` por padrão, sessão fora de cookie legível.
2. **PII exposta em escala:** swagger público com exemplo real de menor (nome, nascimento, telefone, matrícula); roster do diário com ~20 nomes reais de alunos copiados até para a spec; 3 analytics simultâneos (GA + Clarity com session replay + Hotjar-like) rastreando usuários incluindo menores. Novo sistema: redaction por padrão, docs de API sem dados reais, analytics com consentimento e sem replay em telas com PII.
3. **Rotas `/dev/`, `/dev_antigo/`, `/dev-pagamento/` no router de produção** e SQL ad-hoc via Monaco em `/gerador_consultas/` sem whitelist evidente.

**Arquitetura de informação / navegação (warns dominantes):**
4. **`low-confidence-purpose` em 318/318 telas** — nenhuma tela tem propósito identificável pela estrutura (sem h1 consistente, sem landmarks).
5. **`many-unknown-actions` em 309/318 telas** — ícones Material renderizados como texto cru nos elementos interativos (`insert_chart`, `schoolAcadêmicokeyboard_arrow_down`) — botões sem rótulo acessível. Novo sistema: todo controle com rótulo textual/aria.
6. **Grafo de navegação desconectado:** 318 telas e só 66 arestas — quase toda tela é ilha alcançada por menu global ou deep-link; o funil de matrícula e o ciclo de cobrança não têm fluxo encadeado na UI. Novo sistema: jornadas como fluxos de primeira classe (wizard/checklist), não coleção de telas soltas.
7. **Proliferação e duplicação de rotas:** `admin`/`administrador`/`admnistracao` (com typo), `dash`/`dashboard`/`dashboards`, `calendario`/`calendario_escolar`/`calendario_eventos`, `contas`/`contas_receber`/`cobranca`/`cobrancas`/`mensalidade(s)`/`boleto(s)`. Crescimento sem governança de IA.
8. **Navegação incoerente capturada:** botão "Cancelar" em `disciplina/<id>/atualizar` leva a `/cancelar_titulos_isaac_lote/`; aba "Disciplinas" do colaborador 109 abre dados do colaborador 102.
9. **Nomenclatura enganosa:** `/portal_web_parametros/envio_mensagem/` não configura envio de mensagens — configura o fluxo de confirmação de notas professor↔coordenação.
10. **Semântica de estados errada:** empty state de comunicados renderizado como `alert-warning`. Novo sistema: empty/erro/carregando como estados distintos e explícitos.
11. **Parâmetros hardcoded com efeito funcional:** chat lista só conversas com ≥1 não lida (`min_mensagens_nao_lidas=1` fixo no frontend) — admin não enxerga o histórico completo sem manipular URL.
12. **Acessibilidade:** 48 violações WCAG em 163 telas (ex.: `aria-allowed-attr` no popover global do header afetando ~46 telas).
13. **Stack EOL como causa raiz de UX:** React 16.14, CKEditor 4 (EOL abr/2023), moment.js deprecated.

### Pontos fortes a preservar no novo sistema

1. **Hub 360° do aluno com abas** (detalhe/cobrança/descontos/enturmação/ficha médica/histórico/observações) — único padrão de navegação consistente e comprovado em captura.
2. **Caixa de saída como audit trail universal** de tudo que foi enviado à família — diferencial de confiança; elevar a requisito de plataforma (todo canal loga entrega/leitura).
3. **Régua de cobrança automática multi-canal** + integração ISAAC/PIX/boleto registrado — "cobrar sem ser cobrador"; manter o conceito de cobrança garantida com regras claras de quem pode editar o quê.
4. **Comunicado com audiência estruturada** (turmas + situação do aluno + toggle de visibilidade + data de publicação) — modelo de targeting simples e eficaz.
5. **Gates de publicação do boletim** ("Bloquear boletim com impedimento", regras de publicação por série, confirmação do professor via Messenger) — workflow de qualidade pedagógica a manter.
6. **Configurabilidade profunda da escola** (sistema de avaliação, fórmulas, conceitos, históricos, vocabulários como `situacao_aluno`, `motivo_inativacao`) — cada colégio tem sua fórmula; preservar sem virar labirinto de 12+ telas de config.
7. **Portal multi-perfil com SSO contextual** (aluno/responsável/professor sob o mesmo domínio) — manter o conceito, com isolamento de claims mais explícito.
8. **LGPD digital nativa** (termos de consentimento e uso de imagem com aceite) e **avaliação institucional** (canal família→escola).
9. **Operações em lote** por toda parte (frequência, notas, enturmações, títulos) — necessidade real de secretaria; manter com confirmação e trilha de auditoria.
10. **Compliance embarcado** (Educacenso, exportação contábil, NFS-e, conciliação bancária) e **API pública para parceiros** (40 endpoints SigaWeb).

---

## 4. Nível de confiança da spec (resumo do confidence-report)

**Distribuição global das 126 afirmações marcadas:**

| Marcador | Significado | Qtde | % |
|---|---|---:|---:|
| 🟢 Confirmado | validado com fonte secundária | **0** | 0% |
| 🟣 Observado | visto diretamente em captura | **122** | **97%** |
| 🟡 Inferido | deduzido sem captura | 0 | 0% |
| 🔴 Lacuna (`[hipótese]`) | suposição declarada | **4** | 3% |

- **Gate de rastreabilidade: ✅ PASS** (zero pointers quebrados, 3% de 🔴 dentro do threshold). **Gate de review: 🔴 FAIL** (318 errors ≥ 5).
- **As 4 lacunas 🔴 estão todas no financeiro do aluno:** `clusters/alunos--id--cobranca.md` e `clusters/alunos--id--desconto-autorizado.md` — tags `has-list-items` e `has-pagination` presentes em só 1 de 3 instâncias (hipótese: a lista só renderiza quando o aluno tem títulos/descontos). O blueprint do módulo financeiro per-aluno deve tratar esses estados como **não confirmados**.
- **334 de 355 specs não têm nenhum marcador de confiança** (todas as `screens/*`, `jornadas.md`, `comunicacao-familia.md`, `SDD.md`, API docs, design system) — o sistema de marcação cobre praticamente só os 21 clusters; o restante é evidência de captura sem grading formal.

**O que está efetivamente CONFIRMADO POR CAPTURA (🟣) e pode ancorar o SDD:**
- Existência e estrutura das 318 telas (DOM, títulos, tags, cookies, entidades).
- Navegação por abas dos hubs aluno (7 visões × 3 instâncias) e colaborador (6 visões × 2 instâncias) — as 50 jornadas e 66 arestas, todas com confidence 0.90.
- Campos do formulário de cadastro de comunicado; filtros do chat; endpoints chamados (144 UI + 40 públicos); a tela "Títulos e operações" com operações em lote.

**O que é INFERIDO e o SDD deve sinalizar como tal:**
- **Todas as sequências ponta-a-ponta das jornadas de negócio** (J1–J5, J8) — reconstruídas de nomes de rota, endpoints e release notes; nenhum funil foi percorrido de fato.
- **Todo o lado família dos 18 canais:** os portais responsável/aluno/professor **nunca foram capturados por dentro**. Entrega de email/SMS/push é inferência de release notes e campos de schema.
- Configuração de canais (SMTP, SMS, templates) em nível de produto vs. tenant — inferência por ausência (35+ rotas sondadas sem sucesso).
- **Pendências listadas pela própria spec para fechar as lacunas:** credencial real de responsável do CEMP; inspeção da tabela `acesso_responsavel_portal`; form-probe controlado do POST de `/comunicados/cadastrar/`; verificação de webhooks externos.

---

**Resumo executivo:** o sistema atual é um ERP escolar de cobertura funcional excepcional (funil completo captação→contrato→cobrança→pedagógico→comunicação, 18 canais, audit trail central), porém com navegação fragmentada (318 ilhas), 100% das telas reprovadas no gate de segurança (cookies inseguros + PII), acessibilidade ruim e nomenclatura caótica. O novo sistema deve preservar o modelo de hub do aluno, a caixa de saída, a régua de cobrança e os gates de publicação de boletim — e tratar como **inferido** (a validar com captura do portal da família) tudo que diz respeito ao lado responsável/aluno.
