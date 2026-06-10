# 03 — Superfície de API e Integrações

> Extraído de `_janus_spec/api/` — fontes: `endpoints.md` (242 endpoints observados via HAR em 317 telas),
> `api-inferred.yaml` (OpenAPI inferido), `sigaweb-public-api-swagger.json` (40 endpoints públicos),
> `integracoes-externas.md`, `bundle-mining-summary.md`, `api-surface-analysis.md`
> e `sigaweb-changelog-embedded.txt` (931 entradas de release notes).

---

## 1. Mapa da API por domínio funcional

### 1.1 Arquitetura geral: duas APIs disjuntas

O SigaWeb opera **duas APIs HTTP completamente separadas** (sobreposição zero por matching método + path):

| API | Prefixo | Audiência | Auth | Documentação |
|---|---|---|---|---|
| **Interna (UI)** | `/api/v1/...` (206 endpoints), `/api/...` legacy (9) | Exclusivamente a SPA React | Sessão cookie/JWT do usuário logado (227 de 242 endpoints exigem auth) | Nenhuma — descoberta via HAR |
| **Pública (parceiros)** | `/api/v0/...` (40 endpoints) | Integradores externos (ISAAC, ClassApp, Edebê, Teams) | Bearer token por instituição | Swagger 2.0 público em `/docs/?format=openapi` |

Além disso existem **endpoints versionados por migration do banco**: `/api/v1064840/...`, `/api/v1065000/...`, `/api/v1065010/...` — o número é a versão do schema do banco (e/ou tenant), formando snapshots congelados de contrato. O bundle JS revela **280 endpoints únicos** no total (mais do que os 206 capturados por navegação).

### 1.2 Padrões e convenções observados

- **Estilo híbrido REST/RPC**: recursos REST (`/api/v1/alunos/`, `/api/v1/turmas/`) convivem com sub-rotas RPC (`/alunos/{id}/detalhe_simplificado/`, `/alunos/alertas_do_aluno/{id}/`, `/colaborador/situacao_acesso_portal/{id}/`) e um padrão `crud/` explícito (`/alunos/crud/{id}/`, `/disciplina/crud/{id}/`, `/responsavel/crud/{id}/`) que separa payload de leitura/edição do payload de listagem.
- **Paginação Django REST Framework**: `?limit=N&offset=N` com envelope `{count, next, previous, results}` — confirmado em dezenas de respostas (`alunos`, `colaborador`, `descontos`, `turmas`, `grade_curricular`...). Filtros como query params (`periodo=35`, `apenas_ativos`, `ordering=-periodo__nome`, `search=`).
- **Selects de formulário centralizados**: família `/api/v1/fields/select/{entidade}/` (desconto, periodo, serie, unidade, responsavel, profissao, tipo_responsavel, situacao_aluno_turma, texto_personalizado...) + `/api/v1/fields/arvore/series_v2/` para árvores. Padrão útil de replicar: um endpoint genérico de opções para todos os dropdowns.
- **Quase tudo GET na captura** (236 GET / 6 POST): a UI carrega massivamente por leitura; mutações não foram exercitadas na captura, mas existem no bundle.
- **Verificação de permissão por endpoint dedicado**: `/verificar_permissao/relatorio_web/{aluno|caixa|financeiro|professor|responsavel}/` — autorização checada por chamada antes de exibir relatórios.
- **Anti-pattern crítico**: tenant_id no path (`GET /api/v1064840/aluno_ficha_medica/103/`) — PII clínica de menor em URL (vai para logs, history, referers) e permite enumeração de outros colégios. Um novo sistema deve resolver tenant por subdomínio/header/claim do token, nunca por path.

### 1.3 Catálogo por domínio (API interna `/api/v1/`)

**Acadêmico — cadastros base**
| Endpoint | Propósito |
|---|---|
| `GET /api/v1/alunos/`, `/alunos/crud/{id}/`, `/alunos/{id}/detalhe_simplificado/`, `/alunos/fotos/`, `/alunos/turma_oficial/`, `/alunos/alertas_do_aluno/{id}/` | Ficha do aluno (lista, CRUD, resumo, fotos, alertas) |
| `GET /api/v1/aluno_observacao/`, `/aluno_turma_bolsa/`, `/situacao_aluno/`, `/motivo_inativacao/` | Observações, bolsas por turma, situações e motivos de inativação |
| `GET /api/v{tenant}/aluno_ficha_medica/{id}/` | Ficha médica (PII sensível) |
| `GET /api/v1/responsavel/crud/{id}/`, `/responsavel_aluno_vinculado/`, `/responsavel_obs/`, `/responsavel_verifica_situacao/{id}/`, `/tipo_responsavel/` | Responsáveis e vínculos aluno-responsável |
| `GET /api/v1/colaborador/`, `/colaborador/crud/{id}/`, `/colaborador/crud/{id}/disciplinas/`, `/colaborador_escolaridade/{id}/`, `/colaborador/disciplinas_alocadas/`, `/colaborador/quadro_horarios/`, `/colaborador/quadro_horarios_turmas/`, `/colaborador/situacao_acesso_portal/{id}/` | Colaboradores/professores: cadastro, escolaridade, alocação, horários, acesso ao portal |
| `GET /api/v1/curso/`, `/serie/`, `/turmas/`, `/periodo/`, `/disciplina/`, `/disciplina/crud/{id}/`, `/grade_curricular/`, `/alunoturma/`, `/professores_turmas/`, `/sequencias/`, `/tipo_horario/`, `/feriado/`, `/feriado/busca_datas_feriados/` | Estrutura acadêmica: cursos, séries, turmas, períodos letivos, disciplinas, grade, enturmação, alocação de professores, calendário |

**Acadêmico — avaliação, diário e boletim**
| Endpoint | Propósito |
|---|---|
| `GET /api/v1/sistema_avaliacao/`, `/sistema_avaliacao/crud/{id}/`, `/fase_nota/crud/{id}/`, `/fase_nota/sistema_avaliacao_fases_nota_menu/{id}/` | Sistemas de avaliação configuráveis e fases de nota (bimestres/trimestres, recuperações) |
| `GET /api/v1/definicoes_boletim_serie/`, `/configuracoes_serie_periodo/crud/{periodo}/{serie}/` | Modelos de boletim por série e configurações série × período |
| `GET /api/v1/diario/diario/lista/`, `/diario/diario/crud/{id}/`, `/diario/{id}/alunos/lista/` | Diário de classe (frequência + conteúdo) |
| `GET /api/v1/historico/`, `/unidade/parametros_historico_notas/` | Histórico escolar e parâmetros de notas |
| `GET /api/v1/itinerarios_formativos/consultar_vinculo_aluno/` | Itinerários formativos do Novo Ensino Médio |
| `GET /api/v1/exportacao_educacenso/campos/` | Mapeamento de campos para exportação ao Educacenso/INEP |

**Financeiro** (o domínio mais profundo do produto)
| Endpoint | Propósito |
|---|---|
| `GET /api/v1/titulos/`, `/financeiro/resumo_titulos/`, `/calculo_multa_juros/`, `/descontos/`, `/planopagamento/`, `/servico/` | Títulos/cobranças, regras de multa e juros, descontos, planos de pagamento, serviços faturáveis |
| `GET /api/v1/conta_financeira/`, `/financeiro/plano_contas/`, `/financeiro/centro_resultado/`, `/financeiro/historico_padrao/`, `/financeiro/favorecidos/`, `/contas_pagar/`, `/contas_pagar/total/` | Contabilidade: plano de contas, centros de resultado, contas a pagar |
| `GET /api/v1/caixa/abertura_fechamento/caixa/`, `/caixa/tipo_recebimento/` | Caixa físico (abertura/fechamento, tipos de recebimento) |
| `GET /api/v1/financeiro/cobranca_registrada/lotes/`, `/processamentos/titulos_impedidos/`, `/financeiro/cheque_custodia/` | Lotes de boleto registrado, títulos com impedimento, cheques em custódia |
| `GET /api/v1/financeiro/recebimento_pix/`, `/recebimento_por_cartao/operadoras/`, `/formas_de_recebimento/list/` | PIX, cartão (operadoras) e formas de recebimento |
| `GET /api/v1/financeiro/regua_cobranca_automatica/`, `/agente_cobranca/` | Régua de cobrança automática e agentes de cobrança |
| `GET /api/v1/financeiro/forma_recebimento/validacao_isaac/`, `/financeiro/titulos_isaac_pre_matricula_pendentes/`, `/alunos/{id}/status_rematricula_isaac/` | Superfície de integração ISAAC embutida na API interna |
| `GET /api/v1/parametro_global/financeiro/` | Parâmetros financeiros globais |

**Matrícula e captação**
| Endpoint | Propósito |
|---|---|
| `GET /api/v1/procedimentomatricula/`, `/matricula_online/listar_campos/` | Procedimentos de matrícula e formulário dinâmico de matrícula online |
| `GET /api/v1/public/captacao_alunos/` | Captação de leads (endpoint público, sem auth — funil de novatos) |
| `GET /api/v1/campo_dinamico/agrupado/` | Campos dinâmicos configuráveis por escola (extensão de schema sem migration) |

**Comunicação**
| Endpoint | Propósito |
|---|---|
| `GET /api/v1/comunicados/` | Comunicados institucionais |
| `GET /api/v1/mensagem/conversas/`, `/mensagem/caixa_saida/mensagens/`, `/mensagem/mensagem_configuracao/`, `/mensagem/mensagem_configuracao_smtp/`, `/mensagem/mobile_tipo_mensagem/` | Mensageria interna: conversas, caixa de saída, configuração SMTP por escola, tipos de mensagem mobile |
| `GET /api/v1/fields/select/texto_personalizado/` | Templates HTML personalizáveis com variáveis (`[NOMEALUNO]`, `[CPFALUNO]`, `[MATRICULA]`, `[PARENTESCO_FILIACAO_1]`...) |

**Portal (aluno / responsável / professor)**
| Endpoint | Propósito |
|---|---|
| `GET /api/v1/parametro_internet/portal_web_geral/`, `/parametro_internet/portal_web_envio_mensagens/` | Parametrização do portal web por escola |
| `GET /api/v{tenant}/usuario/meu_perfil/`, `/api/v{tenant}/avaliacao_institucional/minhas_avaliacoes/` | Perfil do usuário logado e avaliação institucional (NPS interno) |
| `GET /api/v1/termo_uso_privacidade/`, `/unidade_termo_consentimento/`, `/unidade_termo_uso_imagem/{id}/` | LGPD: termos de privacidade, consentimento e uso de imagem por unidade |

**Dashboards e indicadores**
| Endpoint | Propósito |
|---|---|
| `GET /api/v1/indicadores_dashboard/{alunos_ativos\|inadimplencia\|ocupacao_turmas\|oportunidades_captacao\|situacao_contratos\|titulos_pendencia_registro_boleto}/` | KPIs nativos da home |
| `GET /api/v1/dashboard/indicadores/ocorrencias_registradas/`, `/percentual_ocupacao_turmas/` | Indicadores adicionais |
| `GET /api/v1/dashboards_metabase/pagina_inicial/` + `GET /api/embed/dashboard/{JWT}/dashcard/{id}/card/{id}` | BI delegado ao Metabase via embed assinado por JWT (claims: dashboard id, `sigla_da_unidade`, `id_da_unidade`, `exp`) |

**Configuração, plataforma e suporte**
| Endpoint | Propósito |
|---|---|
| `GET /api/v1/instituicao/`, `/empresa/`, `/unidades/`, `/unidade/retrieve/{id}/`, `/global/` | Instituição, empresas (CNPJs para NFS-e), unidades/filiais, config global |
| `GET /api/v1/parametro_controle_acesso/`, `/links_navegacao/` | Controle de acesso e menu dinâmico por perfil |
| `GET /api/v1/fila_processamento/`, `/fila_processamento/resumo_unidade/` | Fila de jobs assíncronos visível ao usuário (status por unidade) |
| `GET /api/v1/gerador_consultas/` | Gerador de consultas ad-hoc (Monaco editor / SQL builder no admin) |
| `GET /api/v1/relatorio_configuracoes/crud/`, `/relatorios_exportados/favoritos/lista/` | Configuração e favoritos de relatórios; relatórios renderizados em `/relatorio_web/...` |
| `GET /api/v1/zendesk_auth/` | SSO para widget Zendesk (chamado em 57+ telas) |
| `GET /api/v1/assistente_virtual/artigo/` + `POST /api/v1/assistente_virtual/artigo_visualizar/` | Help center embutido |
| `GET /api/v1/aviso_tela_inicial/`, `/central_novidades_visualizacao/`, `/activeblog/ultimo_artigo_visualizado/` | Avisos, central de novidades, blog do produto |
| `GET /api/v1/roteiro_treinamento/{minhas_tarefas_rva_ria\|totais_treinamentos_rva_ria\|verificar_periodo_rva}/` | Onboarding/treinamento gamificado de usuários da escola |
| `POST /api/v1/log_frontend/` | Telemetria de erros do frontend |
| Tabelas auxiliares: `/profissao/`, `/religiao/`, `/agendamento/` | Domínios de apoio |

---

## 2. API pública para parceiros (`/api/v0/`)

- **Host:** `siga.activesoft.com.br`; Swagger 2.0 servido **sem autenticação** em `/docs/?format=openapi` (gerado por drf-yasg/Django REST Framework).
- **Autenticação:** `securityDefinitions` = apiKey no header `Authorization` (`Bearer XXXXXXXX`). **Um token por instituição** — o token É o tenant; não há OAuth, escopos ou granularidade por recurso. Parâmetro `version` obrigatório (=0).
- **40 operações: 37 GET + 3 POST.** Os POSTs são os únicos pontos de escrita: `correcao_prova/` (notas em lote), `financeiro/gerar_cobranca/`, `marcar_frequencia_aluno/`.
- **Schemas declarados (4):**
  - `CorrecaoProva`: `turma_id`, `fase_nota_id`, `disciplina_id`, flags `sobrescrever_nota`/`sobrescrever_nota_confirmada`, array de `{matricula, nota01..nota10}` — lançamento de notas em lote por matrícula, com proteção contra sobrescrita de nota já confirmada.
  - `GerarCobranca`: `id_aluno`, `id_servico`, `valor`, `vencimento`, `id_forma_recebimento`, `id_calculo_multa_juros` (+ `parcela`, `repetir_cobranca`, `repeticoes`, `aglutinar`, `duplicar`, `comentario`) — parceiro cria cobrança recorrente/parcelada.
  - `MarcacaoFrequencia`: `data_hora`, `tipo_entrada_saida` (E|S), `matricula` (+ `id_responsavel_acompanhante`, `comentario`) — catraca/controle de acesso físico; complementada por `GET /identificar_marcacao/{chave_pessoa}/`.

**Grupos funcionais:**

| Grupo | Endpoints | Caso de uso |
|---|---|---|
| Controle de acesso físico | `acesso/alunos/`, `acesso/enturmacao/`, `acesso/vinculo_aluno_responsavel_liberado/`, `identificar_marcacao/{chave_pessoa}/`, `marcar_frequencia_aluno/` (POST), `listar_frequencia_aluno/` | Catracas, carteirinhas, apps de portaria (quem pode retirar o aluno) |
| Pedagógico / editoras e plataformas de conteúdo | `alocacao/`, `lista_alocacao_professor_disciplina/`, `diarios/`, `diario_frequencia/`, `enturmacao/`, `enturmacao_com_detalhes/`, `lista_disciplina_turma/`, `detalhe_boletim/`, `correcao_prova/` (POST) | Edebê, plataformas de avaliação: sincronizar turmas/professores/alunos e devolver notas; Microsoft Teams (alocação aula × sala) |
| Financeiro / cobrança | `financeiro/{calculo_multa_juros, empresas, forma_recebimento, servicos}/`, `financeiro/gerar_cobranca/` (POST), `informacoes_boleto/`, `lista_planos_pagamento/`, `lista_alunos_com_descontos/` | ISAAC e gateways: garantir cobranças, emitir boletos, consultar negociações |
| Cadastros (sync) | `lista_alunos/`, `lista_responsaveis/`, `lista_turmas/`, `lista_cursos/`, `lista_unidades/`, `lista_colaboradores/`, `lista_coordenadores/`, `lista_usuarios/`, `lista_aluno_ocorrencia/`, `lista_aluno_periodo/`, `lista_aluno_curso_ensino_superior/` | ClassApp (sync de alunos/turmas/responsáveis), plataformas de matrícula, Ensino Superior |
| Dados sensíveis (LGPD auto-declarado) | `lista_alunos_dados_sensiveis/`, `lista_responsaveis_dados_sensiveis/`, `lista_alunos_endereco/`, `lista_responsaveis_endereco/` | Criados "para clientes Isaac" (release note) — PII completa via GET, ponto de atenção de compliance |
| Sistema / arquivos | `parametro/`, `porta_arquivos/importar_documento_disciplina/{id}/signed-url/` | Parâmetros da escola; upload via S3 presigned URL |

**Outros mecanismos para parceiros (changelog):** "Criado sistema de geração de token (via API) para parceiros autenticarem no Portal web via Login direto" — ou seja, além de dados, existe **SSO de entrada no portal** gerado por parceiro; e melhorias contínuas dirigidas por parceiro ("Adicionado campo `mostrar_todas_as_turmas` na API de listagem de turmas dos parceiros", "API de boleto dos parceiros", "API de negociações para Isaac", "API de boletim para parceiros").

**Lições para o novo sistema:** uma API pública de parceiros precisa cobrir, no mínimo: sync de cadastros (alunos/responsáveis/turmas/colaboradores), leitura de boletim/frequência, escrita de notas e frequência em lote, geração/consulta de cobrança e boleto, e SSO de portal. Recomenda-se substituir o Bearer fixo por instituição por OAuth2 client-credentials com escopos (ex.: `cadastros:read`, `financeiro:write`, `dados_sensiveis:read`) e auditoria por token.

---

## 3. Catálogo das 15 integrações externas

| # | Integração | Propósito | Mecanismo | Equivalente que o novo sistema precisa cobrir |
|---|---|---|---|---|
| 1 | **ISAAC Pagamentos** | Terceirização/garantia da cobrança escolar (a mais extensa do produto). Rotas: `/operadores_isaac/`, `/cancelar_titulos_isaac_lote/`, `/titulos_isaac_fila_registro/`, `/titulos_pendencia_registro_boleto_isaac_pagamentos/`, `/vincular_servicos_produtos_plataforma_isaac/`, `/status_rematricula_isaac`. Regra dura: "cobrança garantida pelo isaac deve ser cancelada pela operação de inativar o aluno na turma" — o garantidor passa a ser dono do ciclo de vida do título | API bidirecional (APIs de parceiro dedicadas: dados cadastrais, negociações, atualização de títulos com linha digitável) + fila de registro + validação de forma de recebimento | Conector de "garantidor de recebíveis": sync de cadastro, espelhamento de títulos, bloqueio de edição local de títulos garantidos, fila de conciliação, status de rematrícula vinculado ao financeiro |
| 2 | **PJ Bank** | Gateway de boleto/conta digital; importação de extrato para conciliação ("Alterar Data de Última Importação PJ Bank"); sub-integração de e-mails para a conta da ISAAC | API + importação batch de extrato | Conciliação bancária automática com cursor de importação configurável |
| 3 | **PIX** | Pagamento instantâneo em cobranças, "boleto PIX" (bolecode), pagamento PIX no portal do responsável; flag `ACESSO_PIX_IMEDIATO`; rota `/recebimentos_por_pix/` | APIs PIX dos PSPs por forma de recebimento | PIX nativo (QR dinâmico, conciliação automática de recebimento, boleto híbrido com copia-e-cola) |
| 4 | **Banco do Brasil (BB API)** | Credenciamento direto em Formas de recebimento (boleto registrado) | API REST do BB (OAuth do banco) | Emissão e registro direto de boletos multi-banco |
| 5 | **CAIXA API** | Idem BB: "Adicionado credenciamento com a CAIXA API nas Formas de recebimento"; há também credenciamento **Itaú** ("processo de credenciamento com Itau", `ACESSO_BOLECODE_ITAU`) | API REST do banco | Arquitetura de adaptadores bancários plugáveis (BB, CAIXA, Itaú, + CNAB remessa/retorno) |
| 6 | **ReCB** (Recebimento Eletrônico de Cobrança Bancária) | Baixa de boletos via API; **feature-flagged por cliente** ("apenas para o Colégio CPI") | API + tela de processamento; rotas `/financeiro_rececb/`, `/integracao_recb/` | Webhook/polling de baixa eletrônica com toggles por instituição |
| 7 | **ClassApp** | Comunicação mobile escola-família: envio de mensagens, sync de alunos/responsáveis, tags, link de pagamento por app, régua de cobrança via mobile; telas `/log_classapp/`, `/integracoes/classapp/`, botão "forçar sincronização" | API com token de notificação; sync agendado + forçado; log de integração dedicado | App/canal de comunicação com responsáveis (ou conector equivalente): mensagens segmentadas, recibo de leitura, cobrança in-app, log auditável de sincronização |
| 8 | **Microsoft Teams** | Sync de turmas/disciplinas/alocações com salas do Teams (Microsoft Education) | API de parceiro (Teams consome alocação/turmas) | Provisionamento de turmas em Google Classroom/Teams via Graph API |
| 9 | **Edebê** | Editora pedagógica: SSO/sync de usuários para plataforma de conteúdo | API pública de parceiros (lista_usuarios, lista_colaboradores) | API de provisionamento de usuários para editoras/plataformas de conteúdo (padrão OneRoster seria o equivalente moderno) |
| 10 | **Educacenso / Censo Escolar (INEP)** | Exportação censitária obrigatória: `GET /api/v1/exportacao_educacenso/campos/` + tela `/exportacao_educacenso/`; código INEP do aluno em relatórios | Exportação batch no layout do INEP | Geração do arquivo de migração Educacenso (obrigação legal de toda escola brasileira) — requisito não negociável |
| 11 | **NFS-e / NF-e** | Módulo de nota fiscal de serviço: envio de RPS em lote, reprocessamento de rejeitados, cancelamento de NF, variáveis na discriminação do serviço (`CPFALUNO`, `MATRICULA`) | API das prefeituras (RPS → NFS-e) com fila de lotes e estados de erro | Emissor fiscal multi-município com fila, retry e cancelamento — idealmente via hub fiscal (eNotas/Focus/PlugNotas) |
| 12 | **Metabase** | BI embarcado: dashboards da home e relatórios gerenciais | Embed assinado por **JWT HS256** com claims `{resource: {dashboard}, params: {sigla_da_unidade, id_da_unidade}, exp}` — o tenant vai dentro do token | BI embedded com row-level security por unidade via token assinado |
| 13 | **Zendesk** | Suporte ao cliente do produto: widget embedded autenticado | `GET /api/v1/zendesk_auth/` (JWT de SSO) | Widget de suporte com SSO (Zendesk/Intercom) |
| 14 | **ViaCEP** | Autocomplete de endereço por CEP (`https://viacep.com.br/ws/`) | Chamada client-side direta | Lookup de CEP (ViaCEP/BrasilAPI) |
| 15 | **Analytics e utilitários embed**: Google Analytics + Microsoft Clarity + Hotjar (3 trackers simultâneos — risco LGPD: gravação de sessão com PII de menores), Sentry (DSN no bundle), Google Drive viewer, YouTube embed, CKEditor 4 (CDN, EOL), Monaco Editor | Telemetria de produto e visualização de conteúdo | Scripts client-side / embeds | Telemetria com anonimização e base legal LGPD explícita; visualizador de documentos; editor rich-text moderno (substituir CKEditor 4) |

---

## 4. Aprendizados do changelog (931 entradas mineradas do bundle)

**Financeiro (o coração do produto — maior densidade de entradas):**
- **Negociação de dívida é entidade de primeira classe**: "opção 'cobrar apenas multa, sem juros' na tela de negociação", "manter descontos condicionais na negociação", "apagar negociação", "serviços do tipo negociação" na consolidação contábil, "impedimento de alunos com negociações com todos os títulos em aberto", API de negociações para a ISAAC. O novo sistema precisa modelar negociação como título derivado com regras próprias de multa/juros/desconto.
- **Descontos condicionados vs não-condicionados** são conceitos distintos (desconto por pontualidade vs abatimento permanente), com variáveis próprias em contrato (`[DESCONTO_NAOCONDICIONADO_TIPO_ABATIMENTO]`, `[DESCONTO_NAOCONDICIONADO_PERCENTUAL]`) e tratamento de "pagamento a maior quando há desconto".
- **Régua de cobrança automática** com condições finas: "só enviar mensagem se o título estiver registrado", envio imediato em lote, parâmetro mobile para usuários ClassApp, aba de envios no detalhe do título.
- **Filantropia é módulo completo** (Lei 12.101/CEBAS): ficha socioeconômica "Filantropia-IVM", solicitação de desconto pelo portal com termo de consentimento, relatório de "renda per capita", "intervalo de liberação de descontos", parâmetros e workflow de aprovação.
- **Caixa físico operacional**: abertura/fechamento com observação, retirada automática no fechamento, conta vinculada para retirada, pagamento rápido, tarifa de cartão por tipo de recebimento, comprovantes e segunda via.
- **Impedimentos** como mecanismo transversal: ocorrências podem bloquear renovação de matrícula pelo portal; títulos podem ficar "impedidos" (`/processamentos/titulos_impedidos/`); "modais operativos permitindo corrigir os impedimentos diretamente no alert de validação".

**Contratos e assinatura eletrônica:**
- Geração automática de contratos com fila própria; cancelamento automático por prazo-limite de assinatura e **reenvio em lote dos cancelados**; variáveis de cobrança no texto (valor por extenso das parcelas, dia do vencimento); filtro por usuário que interagiu com o contrato (criou/cancelou/enviou) — trilha de auditoria de contrato.

**Acadêmico:**
- **Boletim hipercustomizável por escola**: modelos por série/curso, agrupamento de disciplinas por área BNCC, ranking no boletim, "customização do boletim da instituição CNSA com notas sem vírgula", "boletim de turmas de progressão parcial", legenda por colégio — personalização por cliente é requisito de venda, não exceção.
- **Novo Ensino Médio**: controle de acesso dedicado (`ACESSO_NOVO_ENSINO_MEDIO`), matrícula em itinerários formativos pelo portal do aluno, divisão de carga horária no histórico do EM, modelos regionais de histórico ("modelo Curitiba/PR", "portaria e título no cabeçalho", "brasão da República").
- **Avaliação por competência** como sistema paralelo a notas: telas de configuração, planilha, auditoria e edição na ficha do aluno.
- **Reprocessamento de notas com fila**: "ação para limpar fila de processamento de notas de fases de avaliação" — cálculo de médias é assíncrono.
- Portal do aluno com **entrega de avaliações/atividades** (envio, comprovante de envio, visualização) — funcionalidade de AVA leve.

**Matrícula:**
- Procedimentos de matrícula seletivos por canal: "seletor para definir quais procedimentos estarão disponíveis para inscrição de novatos ou reserva de matrícula online"; renovação online com validação de configuração; ficha de inscrição de novatos com foto processada assincronamente; dashboards de funil ("Quantidade de matrículas e renovações de matrícula online").

**Operações em lote como padrão de UX**: gerar ocorrências em lote, frequência em lote, alterar notas em lote, liquidar títulos em lote (com botão "Simular"), inativar enturmações em lote, dispensa de tarefas em lote, RPS em lote, reordenar alunos em lote — qualquer sucessor precisa de framework genérico de bulk-operations com simulação e fila.

**Segurança/identidade (curioso)**: "A nova senha deve ter pelo menos 8 caracteres" convive com "A nova senha deve ter no máximo 10 caracteres" — política de senha fraca herdada de legado; recuperação de senha por e-mail com usuário de envio configurável; "envio de senhas" em massa para responsáveis (tela `/envio_de_senhas/`).

**Gestão de release acoplada ao banco**: "A nova rotina estará disponível após versão do banco de dados 1065170 ou superior" — features gated por versão de schema, coerente com o versionamento `/api/v{migration}/`.

---

## 5. Requisitos não-funcionais inferíveis

1. **Multi-tenancy por instância + unidade**: cada colégio tem instância/tenant identificado por versão de banco própria (`v1064840`, `v1065000`...) e hosts dedicados (`siga04.activesoft.com.br`); dentro do tenant há multi-unidade (`/unidades/`, filtros e relatórios por unidade, JWT do Metabase com `id_da_unidade`). O novo sistema deve isolar tenant fora da URL (subdomínio + claim) e modelar unidade como dimensão de todos os dados.

2. **Autenticação multi-perfil e SSO**: perfis aluno / responsável / professor / colaborador / admin com portais distintos (`/portal_aluno`, `/portal_responsavel`, `/portal_professor`) e checagem de permissão por chamada (`/verificar_permissao/...`, `/parametro_controle_acesso/`, `/colaborador/situacao_acesso_portal/`). Constantes `USUARIO_OPERADOR`, `USUARIO_SUPORTE`, `USUARIO_VENDEDOR_CAPTACAO` revelam **usuários internos do fornecedor com acesso ao ambiente do cliente** (suporte assistido) — exigir impersonação auditada. SSO de saída para Zendesk e Metabase (JWT) e SSO de entrada para parceiros (token de login direto no portal).

3. **Feature flags em três camadas**: (a) permissões/flags nomeadas no bundle (`ACESSO_PIX_IMEDIATO`, `ACESSO_NOVO_ENSINO_MEDIO`, `ACESSO_REDESIGN_LANCAMENTO_FINANCEIRO`, `ACESSO_API`...); (b) **toggle por cliente** confirmado ("Integração ReCB apenas para o Colégio CPI", tela `/whitelist/`, `/feature/`); (c) gating por versão de banco. Sucessor precisa de feature-flag service com escopo global/tenant/perfil.

4. **Filas de processamento assíncrono como cidadão de primeira classe**: `/api/v1/fila_processamento/` (+ resumo por unidade) exposto na UI; filas específicas para registro de boletos ISAAC, pendências de registro, geração automática de contratos, lotes de RPS/NFS-e, reprocessamento de notas, importação de fotos, processamento de ficha médica. Requisito: jobs com status visível ao usuário, retry e reprocessamento manual em lote.

5. **Paginação, filtros e exportação padronizados**: limit/offset em toda listagem; relatórios como subsistema próprio (`/relatorio_web/...` com permissão por área, favoritos, exportados) — motor de relatórios configurável é parte do produto, não acessório.

6. **Auditoria e LGPD**: botão/modal de auditoria (parâmetros, alunos, avaliação por competência), telas `/auditoria/`, termos de consentimento/uso de imagem/privacidade por unidade, endpoints "dados_sensiveis" auto-declarados. Anti-requisitos a corrigir: PII em path de URL, PII real em exemplo de swagger público, 3 trackers de sessão sem base legal clara, swagger público sem auth.

7. **Observabilidade**: `POST /api/v1/log_frontend/` (telemetria própria), Sentry, logs de integração dedicados por parceiro (LOG ClassApp) — cada conector deve nascer com log auditável e tela de pendências (`/integracoes/pendencias/`).

8. **Compatibilidade e ciclo de vida de API**: versionamento por snapshot de migration garante backward-compat por contrato congelado, ao custo de proliferação de versões; changelog embarcado no bundle (central de novidades in-app) e roteiro de treinamento gamificado indicam que **release notes e onboarding in-product** são features esperadas pelo mercado.

9. **Tech debt a não repetir**: React 16 + moment.js + CKEditor 4 (EOL) simultâneos; rotas `/dev/`, `/dev_antigo/`, `/dev-pagamento/` declaradas em produção; SQL ad-hoc via Monaco em `/gerador_consultas/` (superfície de ataque se não houver whitelist); API quase só GET com mutações pouco padronizadas (RPC ad-hoc).
