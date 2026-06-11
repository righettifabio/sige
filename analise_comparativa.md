# Análise Comparativa — SIGE (base) × Activesoft SIGA (referência)

> **Etapa 1 de 2.** Este documento compara o sistema atual (`bp/bp_sige_atual`, base fixa que não pode ser descontinuada) com o sistema de referência (`bp/bp_activesoft`, fonte de capacidades de evolução). Ele NÃO é o blueprint final — produz os insumos para a etapa seguinte.
>
> **Convenções:** MANTER · ACRESCENTAR · APRIMORAR · CONFLITO · FORA DE ESCOPO. Marcadores: `[LACUNA]`, `[SUPOSIÇÃO]`, `[SUGESTÃO ALÉM DAS FONTES]`. Como os segmentos-alvo do SIGE não foram declarados, capacidades que só fazem sentido além de infantil/fundamental estão sinalizadas com **(por segmento)** na matriz.

---

## 1. Sumário da análise

A base (SIGE atual) é um sistema **verticalizado em educação infantil/creche multiunidade**: seu núcleo diferenciado é a rotina diária do aluno (refeições com escala de consumo, sono, medicação ministrada por horário, troca de fraldas, presença parcial), o fluxo de **requerimentos formais com assinatura digital e implementação automática**, fotos com **dupla aprovação**, e um ciclo financeiro de boletos CNAB (Santander/Sicoob) com NFS-e municipal. O Activesoft é um **ERP escolar horizontal** (infantil ao médio/técnico) de um grupo multitenant: seu núcleo diferenciado é o **motor de avaliação fórmula-dirigido** (fases, aprovação por nota E frequência, boletim hipercustomizável, histórico escolar), o **financeiro profundo** (PIX, registro online de boleto, régua de cobrança, negociação, caixa físico, contábil) e padrões transversais valiosos (situação do aluno como meta-modelo configurável, impedimentos por canal, caixa de saída como trilha de auditoria da comunicação, operações em lote com simulação).

Principais achados:

1. **Sobreposição real é menor do que parece.** Os dois sistemas cobrem o mesmo ciclo (captação → matrícula → financeiro → comunicação), mas com profundidades invertidas: a base é mais rica em rotina infantil, conteúdo com aprovação (fotos, comunicados) e confirmação de leitura; o Activesoft é mais rico em pedagógico formal (notas/boletim/histórico) e financeiro. A maior parte do Activesoft entra como **ACRESCENTAR** (62 capacidades) e **APRIMORAR** (31), não como conflito.
2. **Conflitos são estruturais, não funcionais.** Os 10 conflitos identificados (§7) estão nos modelos: estrutura acadêmica (Atendimento por agrupamento etário × Série/Disciplina/Grade), modelo de cobrança (cotas CMAE/CC/CRV × Título/Plano de Pagamento), fluxo contratual (Requerimento+Autentique × pré-matrícula por pagamento+Clicksign), estados do aluno (fixos × meta-modelo configurável) e permissões da família (atribuições por parente × matriz situação×recurso). Todos têm estratégia de coexistência viável — em geral, tratar o modelo da base como **caso particular semeado** do modelo mais geral do Activesoft.
3. **O Activesoft traz dois requisitos legais que a base não declara**: exportação **Educacenso/INEP** (obrigação de toda escola brasileira) e **termos LGPD digitais** (consentimento e uso de imagem com aceite datado). São acréscimos prioritários. `[LACUNA]` — o BP da base não diz como a escola cumpre o Educacenso hoje.
4. **Um conflito exige cautela jurídica:** o mecanismo de **impedimento por inadimplência** do Activesoft (bloquear boletim/portal/matrícula) colide com o princípio da base de acesso pleno do responsável legal e com limites da legislação brasileira (Lei 9.870/1999 veda retenção de documentos escolares e penalidades pedagógicas por inadimplência; a recusa de **re**matrícula é permitida). O mecanismo deve entrar configurável e com guard-rails legais (§7.7).
5. **Parte do Activesoft é recurso de plataforma multitenant do fornecedor**, não recurso de uso da escola (tiers comerciais, feature flags por cliente, usuários de suporte do fornecedor, central de novidades do produto, treinamento gamificado, Zendesk). Foram separados no inventário (§3.2) e, em regra, classificados FORA DE ESCOPO — com duas adaptações que convergem com o §12 do BP da base: **impersonação auditada** e **feature flags internos**.
6. **Os anti-padrões do Activesoft são insumo tão valioso quanto as capacidades**: o gate de qualidade reprovou 318/318 telas (cookies inseguros, PII em URL/swagger, 3 trackers sobre menores, senha máx. 10 caracteres, navegação fragmentada, 10.462 `!important`). Esses achados viram requisitos negativos do blueprint, somando-se ao §12 da base.

Decisões em aberto para a etapa de blueprint: segmentos-alvo do SIGE (define ~14 capacidades condicionais), adoção ou não de caixa físico/recebimento presencial, multi-empresa (CNPJs múltiplos), profundidade do módulo contábil (contas a pagar/DFC/exportação) e postura sobre o gerador de consultas SQL ad-hoc.

---

## 2. Inventário do sistema atual (base — MANTER integral)

Tudo nesta seção recebe classificação **MANTER** por definição. Fonte: `bp_sige_atual/bp_sige.md`.

### 2.1 Módulos e funcionalidades

| Módulo | Capacidades |
|---|---|
| **Pessoas e acesso** | Cadastro único de Pessoa (PF/PJ, validação CPF/CNPJ); Parente com atribuições granulares por aluno (9 códigos); responsável legal com acesso pleno; Colaborador com permissões por unidade e módulo + flags (assinatura, vencimento de boleto, visibilidade); login e-mail/CPF + senha; login social Google/Apple/Facebook; recuperação por token (web) e código 6 dígitos por e-mail/SMS (app); troca de perfil colaborador↔parente |
| **Acadêmico (infantil)** | Aluno por ano letivo com Atendimento (unidade, curso, agrupamento etário em meses, nível, turno, permanência, horário); turma por tipo de serviço com vagas regulares e **inclusivas** separadas; cancelamento reativável com 3 motivos; alunos soltos; importação em lote; presença diária entrada/saída com **presença parcial** |
| **Rotina diária (creche)** | Diário do aluno: refeições com escala de consumo (0–4), sono, medicação prescrita e ministrada por horário com anexo de receita, troca de fraldas, saída com pessoa autorizada; fechamento noturno de presenças; acompanhamento quase em tempo real pelo app |
| **Financeiro** | Boleto com 3 tipos de cota (CMAE/CC/CRV); valores em centavos; serviço = combinação de atributos com preço por vigência; remessa/retorno CNAB Santander e Sicoob; baixa automática e manual; juros/multa/mora (2% multa, juro diário mín. R$ 0,01, desconto até o vencimento); vencimento nunca em fim de semana/feriado; dia de vencimento 1–28 por aluno; boleto liquidado imutável; painel de cobrança com e-mail consolidado; NFS-e (RPS, lotes, certificado digital, controle mensal por unidade); plano de contas com movimentos e conciliação boleto×movimento; fornecedores; bolsas com vigência monitorada |
| **Requerimentos** | 8 tipos; workflow 0→1→3→4 (+2 negado, +5 cancelado); contrato PDF; assinatura digital multi-signatário (Autentique); **implementação automática na data prevista** com recálculo financeiro |
| **Comercial** | Prospect (incl. "por nascer"); formulário público de agendamento de visita com e-mails automáticos e alerta de duplicidade; interações com status derivado; inativação com motivo; indicações; acompanhamento sistemático |
| **Comunicação** | Comunicados com menções/aprovação/lixeira/leitura confirmada; circulares individualizadas com rastreio e **reenvio aos não confirmados**; ocorrências com comentários encadeados; fotos com **dupla aprovação** e compartilhamento seletivo; notificações internas + push OneSignal (campanhas com filtro por atribuição); e-mails transacionais; documentos institucionais em capítulos |
| **Administrativo** | Multiunidade com **extensão de unidade**; ano letivo único corrente com janela de renovação; dias letivos/feriados parametrizados; renovação em lote com projeção de atendimento; log de auditoria; exportação publicitária CSV |
| **App/API** | Capacidades de responsáveis (diário, financeiro c/ PDF de boleto, circulares, fotos, renovação, dados) e colaboradores (rotina de sala completa); resposta padrão `{status, message, data}`; verificação de versão mínima por plataforma; deep links Android/iOS |
| **Rotinas agendadas** | ~20 rotinas (baixa, notificações de vencimento com reaviso 3 dias, conciliação, prova real, bolsas, RPS, renovações, implementação de requerimentos, verificação de assinaturas, fechamento de presenças, exportações) |

### 2.2 Entidades (~67, em 8 subdomínios)

Pessoas (Pessoa, Parente, Parentesco, Colaborador, Login…); Acadêmico (Aluno, Turma, Curso, Agrupamento, Nível, Turno, Permanência, Horário, Presença, Diário, Cancelamento); Rotina (Refeição, Medicação, Troca de Fralda, Repouso, Agendamento); Financeiro (Boleto, Serviço, Valor, Movimento, RPS, Nota Fiscal, Registro de Pagamento…); Comunicação (Comunicado, Circular, Ocorrência, Notificação, Foto, Documento); Comercial (Prospect, Interação, Inativação, Indicação); Administrativo (Unidade, Ano Letivo, Dia Letivo, Feriado, Requerimento, Log).

### 2.3 Integrações da base

Autentique (assinatura), Santander/Sicoob (CNAB), Prefeitura (NFS-e por webservice + certificado digital), Google (login + e-mail), Apple/Facebook (login), OneSignal (push), provedor de SMS, exportação publicitária.

### 2.4 Restrições herdadas a corrigir (§12 da base — vinculantes para o blueprint)

Eliminar senha-mestre global (→ impersonação auditada); segredos fora do código; convite em vez de senha derivada de CPF; relações normalizadas em vez de listas JSON; invariantes na escrita (fim das rotinas de correção); API unificada; feriados 100% parametrizados; tokens com revogação; reavaliar limites de lote/paginação.

---

## 3. Inventário do Activesoft

Fonte: `bp_activesoft/00–06`. Confiabilidade: 97% das afirmações observadas em captura; **o lado família dos portais é inferido** (nunca capturado por dentro); valores concretos (multa 2%, régua às 14h) são parametrização do tenant CEMP, não defaults.

### 3.1 Recursos de uso da escola (14 módulos)

| Módulo | Capacidades-chave |
|---|---|
| **Alunos (hub 360°)** | Detalhe em 7 abas (cadastro, ficha médica, turmas, ocorrências, histórico, descontos, cobranças); nome social × civil; dados censitários; campos dinâmicos; semáforos de pendência OBS/FNC/COB/BIB/PDM; ficha médica com 44 atributos; ranking de alunos |
| **Responsáveis** | PF/PJ; papéis **financeiro × pedagógico**; dia preferencial de vencimento; instrução de boleto individual; acesso ao portal **derivado** do vínculo com aluno ativo elegível; catraca/digital |
| **Colaboradores** | Formação acadêmica; disciplinas habilitadas; **alocação didática turma×disciplina ativa o acesso do professor**; quadro de disponibilidade (slots M/T/N 1–8); dados INEP/gestor Educacenso |
| **Estrutura acadêmica** | Período → Curso → Série → Turma → Disciplina via Grade Curricular; série com serviço financeiro padrão e `proxima_serie`; turma com vagas, restrições (idade, sexo, pré-requisito), código INEP, modelo de contrato; disciplinas compostas; itinerários formativos (NEM) |
| **Diários de classe** | Diário = turma×disciplina×fase; aulas/conteúdo/tarefas/chamada; ~25 parâmetros por escola; criação automática de aulas pelo quadro de horário; falta justificada/dispensada; frequência por catraca via API |
| **Avaliação/Notas/Boletim** | Sistemas de avaliação por série; fases encadeadas por **fórmulas (DSL)**; aprovação = **nota E frequência**; recuperação roteada por fórmula; fila assíncrona com confirmação professor→coordenação; planilha de notas; alteração em lote **com simulação**; conceitos (nota OU percentual); avaliação por relatório (descritiva) e por competência; boletim com ~60 flags e regras de publicação; histórico escolar configurável; promoção/finalização em lote |
| **Comunicação** | Comunicados com audiência turma+situação; chat 1:1 (Conversas do App); **caixa de saída** = trilha de auditoria universal de envios; templates HTML com variáveis; envio de senhas em massa; banner global |
| **Financeiro** | Título/parcela; planos de pagamento; multa/juros como cadastro nomeado; descontos condicionais×permanentes com ordem de cálculo; régua de cobrança automática (offsets -15/-5/0 × canal × serviço); negociação de dívida; telecobrança; caixa físico (cheques, cartão com repasse D+n, PIX); contas a pagar; plano de contas hierárquico; centro de resultado; conciliação bancária; DFC; exportação contábil; NFS-e em lote; declaração IR; recibos; pagamento a menor/a maior com travas; data de bloqueio contábil; filantropia (CEBAS) |
| **Captação & Matrícula** | Funil com vendedor + WhatsApp do consultor; ficha de inscrição pública; matrícula online (form-builder); **pré-matrícula confirmada por pagamento**; procedimentos de matrícula (checklist documental com prazo); sequenciais nomeados; contrato com assinatura Clicksign; SLA de 14 dias; RM online com etapas configuráveis |
| **LGPD/Termos** | Termo de consentimento e de uso de imagem por unidade, com texto rico, flag ativo e **aceite datado**; endpoints de dados sensíveis segregados; CPF mascarado em tela |
| **Calendário & Agendamentos** | Feriados com **efeito financeiro/biblioteca** e cópia entre períodos; responsável **marca reunião pelo portal** sobre disponibilidades cadastradas |
| **Portais** | Aluno/responsável/professor no mesmo domínio (claim JWT); matriz de permissões **persona × situação do aluno × recurso** (~100 chaves); carteirinha digital; porta-arquivos; boletim detalhado; ocorrências por categoria; edição de cadastro/ficha médica pelo responsável; contato com a instituição por setor |
| **Configurações & Admin** | Multi-unidade + multi-empresa (CNPJ); auditoria por entidade em quase toda tela; gerador de consultas SQL; central de relatórios com favoritos e fila de exportação; **Educacenso** (blocos 00–60 com validação); fila de processamento visível; campos dinâmicos; cadastros auxiliares configuráveis |
| **Integrações (15)** | ISAAC (garantidora), PJ Bank, PIX, BB/CAIXA/Itaú API, ReCB, ClassApp, Teams, Edebê, Clicksign, ViaCEP, Zendesk, Metabase (BI embarcado), Google for Education, catraca GPA, API pública de parceiros (40 endpoints, 3 de escrita) |

### 3.2 Recursos de plataforma multitenant (separados — fornecedor, não escola)

| Recurso | Natureza |
|---|---|
| Instâncias por cliente (`sigaNN`, banco por cliente), versionamento de API por migration do banco | Operação do fornecedor |
| **Tiers comerciais** (Light/Basic) e **feature flags por instituição** (`whitelist`, "ReCB apenas para o Colégio CPI") | Comercialização SaaS |
| **Usuários de suporte do fornecedor** dentro do ambiente do cliente (`USUARIO_SUPORTE`); parâmetros "alterados apenas pela equipe Activesoft" | Operação assistida do fornecedor |
| Central de novidades (changelog in-app), roteiro de treinamento gamificado, assistente virtual + Zendesk | Sucesso do cliente do fornecedor |
| Provedores de e-mail/SMS/push geridos no nível do produto (35+ rotas de config sondadas e inexistentes no tenant) | Infraestrutura do fornecedor |
| Hierarquia Instituição → Unidade → Empresa | **Híbrido**: Instituição = plataforma; Unidade/Empresa = uso da escola |

### 3.3 Padrões transversais do Activesoft (capacidades arquiteturais reutilizáveis)

Situação do aluno como meta-modelo configurável; impedimento como mecanismo transversal por canal; operações em lote com simulação + fila assíncrona visível; auditoria onipresente; soft delete com motivo e destino; encadeamento temporal (`proximo_periodo`/`proxima_serie`); parametrização em camadas (global → unidade → série×período → turma → diário); snapshot/versionamento de fórmulas de cálculo.

### 3.4 Anti-padrões do Activesoft (NÃO importar — viram requisitos negativos)

Cookies sem `Secure` (318/318 telas); PII de menores em URL/swagger público/spec; 3 trackers com session replay sobre menores; senha máx. 10 caracteres; tenant no path da URL; rotas `/dev/` em produção; SQL ad-hoc sem whitelist; 318 telas-ilha (66 arestas de navegação); nomenclatura caótica e enganosa; 5 frameworks de UI empilhados (10.462 `!important`, z-index 2147483647); dados sujos em cadastros auxiliares ("maãe", "mae"); stack EOL (React 16, CKEditor 4); Bearer único por instituição sem escopos na API pública.

---

## 4. Matriz comparativa

Colunas: **Capacidade · Na base (atual) · No Activesoft · Classificação · Impacto sobre a base · Observações**. Agrupada por domínio. "(por segmento)" = só faz sentido além de infantil/fundamental ou depende dos segmentos-alvo não declarados.

### 4.1 Pessoas, identidade e acesso

| Capacidade | Na base | No Activesoft | Classificação | Impacto sobre a base | Observações |
|---|---|---|---|---|---|
| Cadastro único de Pessoa (PF/PJ) | Sim — entidade Pessoa única para aluno/parente/colaborador/fornecedor | Não — Aluno, Responsável e Colaborador são cadastros separados (endereço compartilhado com propagação) | **MANTER** | Nenhum — base é superior | Não importar a separação do Activesoft; a Pessoa única já resolve o problema que o "endereço propagado" remenda |
| Atribuições granulares por parente×aluno (9 códigos) | Sim | Parcial — papéis fin/ped + matriz por situação | **MANTER** | Nenhum | Núcleo do modelo de privacidade da base; ver conflito C4 (§7.4) para a composição |
| Papéis responsável financeiro × pedagógico | Parcial — responsável legal + responsável financeiro | Sim — designação explícita por vínculo, com efeitos distintos (pagador de títulos × destinatário pedagógico) | **APRIMORAR** | Aditivo: formalizar o papel pedagógico no vínculo Parente↔Aluno | Coexiste com atribuições: papel define defaults, atribuição refina |
| Parentesco configurável | Sim (tabela) | Sim (tabela, com dados sujos no tenant) | **MANTER** | Nenhum | Lição negativa: validar/deduplicar valores ("maãe"/"mae") |
| Ficha médica estruturada | Parcial — medicações no diário, necessidades especiais no prospect, alergias citadas só como NFR | Sim — 44 atributos (alergias, doenças, deficiências c/ laudo, SUS, plano, hospital de remoção, contato de emergência, restrições alimentares, remédio para febre) | **APRIMORAR** | Aditivo: nova entidade 1:1 com aluno; a medicação diária da base passa a referenciar a ficha | Alto valor para creche (alergia/restrição alimentar alimenta a rotina de refeições da base). `[LACUNA]` na base: não há entidade de saúde consolidada |
| Nome social × nome civil | Não | Sim (toggle) | **ACRESCENTAR** | Aditivo no cadastro de Pessoa | Boa prática e aderência a normas de registro escolar |
| Dados censitários (cor/raça, deficiências p/ INEP) | Não | Sim (aluno e colaborador) | **ACRESCENTAR** | Aditivo | Pré-requisito do Educacenso (ver 4.8) |
| Campos dinâmicos por escola | Não | Sim (`campo_dinamico` no cadastro do aluno e da unidade) | **ACRESCENTAR** | Aditivo | Extensão de schema sem migração; útil para multiescola futuro |
| Formação acadêmica do colaborador | Não | Sim (escolaridade, disciplinas habilitadas) | **ACRESCENTAR** (por segmento) | Aditivo | Relevante quando houver docência por disciplina; também alimenta Educacenso (bloco 50) |
| Quadro de disponibilidade do professor (slots M/T/N) | Não — colaborador tem jornada simples | Sim — matriz slot×dia por período/turno | **ACRESCENTAR** (por segmento) | Aditivo | Pré-requisito da criação automática de aulas no diário de classe |
| Acesso do professor derivado da alocação didática | Não — permissões por módulo/unidade | Sim — alocar em turma×disciplina ativa o portal | **ACRESCENTAR** (por segmento) | Aditivo: novo vetor de permissão derivada, sem remover os vetores atuais | Padrão "acesso derivado de vínculo, não concedido à mão" também vale para responsáveis (já é assim na base) |
| Login social (Google/Apple/Facebook) | Sim | Não observado | **MANTER** | Nenhum | Vantagem da base |
| Recuperação de senha por e-mail/SMS | Sim (código 6 dígitos, 1h) | E-mail com link; envio de senhas em massa | **MANTER** + ver 4.6 (envio em massa) | Nenhum | Política de senha do Activesoft (máx. 10 chars) é anti-padrão — descartar |
| Impersonação auditada (suporte/admin) | Não — existe senha-mestre global (a eliminar, §12.1) | Usuários do fornecedor no ambiente do cliente (anti-padrão a corrigir, segundo o próprio BP) | **ACRESCENTAR** (adaptado) | Substitui a senha-mestre | Convergência: os dois BPs apontam a mesma solução — impersonação com permissão, trilha e prazo |

### 4.2 Estrutura acadêmica e matrícula

| Capacidade | Na base | No Activesoft | Classificação | Impacto sobre a base | Observações |
|---|---|---|---|---|---|
| Estrutura por agrupamento etário (meses) + nível + turno + permanência + horário | Sim — classificação automática pela data de nascimento | Não — restrição etária é atributo da turma (nascimento mín/máx) | **MANTER** | Nenhum | Essência do produto para infantil; ver conflito C1 |
| Hierarquia Período→Curso→Série→Turma→Disciplina + Grade Curricular | Não — Curso→Agrupamento→Nível; turma por tipo de serviço; sem disciplina | Sim — espinha dorsal, com série→serviço financeiro e `proxima_serie` | **CONFLITO** | Estrutural — ver §7.1 | Coexistência: modelo unificado em que Agrupamento/Nível da base é um tipo de "série etária" e Disciplina/Grade é extensão opcional por segmento |
| Encadeamento `proxima_serie`/`proximo_periodo` | Implícito — renovação projeta curso/agrupamento seguinte pela idade | Explícito — listas ligadas que habilitam promoção e rematrícula em lote | **APRIMORAR** | Aditivo: tornar explícita a progressão que hoje é calculada | Preserva a derivação por idade da base como regra de fallback |
| Vagas regulares × vagas inclusivas | Sim — contagem separada por turma | Não — apenas quantidade máxima | **MANTER** | Nenhum | Vantagem da base (relevante p/ legislação de inclusão); ocupação projetada considera trocas aprovadas — manter |
| Restrições de elegibilidade da turma (idade, sexo, pré-requisito de outro vínculo) | Parcial — agrupamento etário | Sim — nascimento mín/máx, sexo, "exigir vínculo em outra turma" (útil p/ extracurriculares) | **APRIMORAR** | Aditivo | Pré-requisito de vínculo habilita turmas de atividades extras com matrícula condicionada |
| Situação do aluno na turma como meta-modelo configurável | Não — estados fixos (ativo, cancelado reativável) | Sim — cadastro configurável mapeado a situação-sistema/acadêmica/Educacenso + flags (digita nota, assina contrato) | **APRIMORAR** (com cuidado de migração) | Estrutural-aditivo — ver §7.6 | Peça de estado central do Activesoft; semear com as situações equivalentes da base |
| Motivos de inativação configuráveis | Parcial — 3 motivos fixos de cancelamento | Sim — cadastro próprio + destino de transferência | **APRIMORAR** | Aditivo: motivos viram cadastro, semeado com os 3 atuais | Manter a semântica "reativável enquanto não finalizado" da base |
| Enturmação como entidade rica (datas, planos, autorização com justificativa, exceção auditada) | Parcial — aluno↔turma + registro de atendimento | Sim — ~60 campos, incl. matrícula autorizada apesar de impedimento com justificativa+autor | **APRIMORAR** | Aditivo | O "override auditado" é padrão a adotar em todo bloqueio |
| Sequenciais de matrícula nomeados (por unidade/tipo/ano) | Não — número de matrícula simples | Sim | **ACRESCENTAR** | Aditivo | Resolve numeração independente por unidade — a base é multiunidade |
| Progressão parcial / dependência | Não | Sim (séries com `quantidade_dependencias`, turma de progressão) | **FORA DE ESCOPO** (por segmento) | — | Só faz sentido em fundamental II/médio; sinalizado caso os segmentos-alvo se ampliem |
| Promoção entre séries / finalizar turmas em lote | Parcial — renovação em lote com projeção | Sim — com filtro de impedidos, situação de destino, irreversibilidade declarada | **APRIMORAR** | Aditivo: a renovação da base ganha o desenho de lote com simulação | Manter o requerimento de renovação como mecanismo formal; ver C3 |
| Matrícula online self-service (form-builder público) | Parcial — formulário público só de agendamento de visita | Sim — ficha de inscrição pública + RM online com etapas configuráveis (ficha médica, plano, serviços) | **ACRESCENTAR** | Aditivo: novo canal de entrada que desemboca no fluxo de requerimento/contrato da base | Termos LGPD embutidos no fluxo (ver 4.7) |
| Procedimentos de matrícula (checklist documental com prazo e upload) | Não | Sim — por série×período, com obrigatoriedade, prazo, envio no portal, flag PDM | **ACRESCENTAR** | Aditivo | Útil já no infantil (carteira de vacinação, laudos, fotos) `[SUGESTÃO ALÉM DAS FONTES: incluir vacinação como procedimento típico]` |
| Pré-matrícula confirmada por pagamento | Não — matrícula efetiva com assinatura do contrato | Sim — pagar qualquer parcela do título de pré-matrícula muda a situação automaticamente | **CONFLITO** (leve) | Ver §7.3 | Coexistência: gatilhos alternativos de efetivação (assinatura E/OU pagamento), parametrizável por escola |
| Requerimentos formais com workflow e implementação automática na data | Sim — 8 tipos, status 0–5, assinatura, rotina de implementação | Não há equivalente unificado (mudanças são operações diretas + solicitações dispersas) | **MANTER** | Nenhum | Diferencial da base; vira o "motor de mudança contratual" sobre o qual os acréscimos se apoiam |
| Calendário escolar (dias letivos, feriados) | Sim — feriados afetam vencimento; dias letivos com atividades | Sim — feriado com efeito financeiro/biblioteca por unidade + **cópia entre períodos**; interação com diário | **APRIMORAR** | Aditivo: efeitos tipados do feriado + cópia ano a ano | Resolve também o §12.7 da base (feriados parcialmente em código) |
| Agendamento de reunião self-service pelo responsável | Não — agendamento criado pela equipe | Sim — escola cadastra disponibilidades, responsável marca pelo portal, com motivo de cancelamento | **APRIMORAR** | Aditivo sobre a entidade Agendamento existente | Bloqueável por impedimento (configurável) — ver §7.7 |

### 4.3 Rotina diária e pedagógico

| Capacidade | Na base | No Activesoft | Classificação | Impacto sobre a base | Observações |
|---|---|---|---|---|---|
| Diário do aluno (rotina creche: refeições, sono, medicação, fralda, presença parcial) | Sim — núcleo do produto | Apenas flag `utiliza_rotina_educacao_infantil` na série, sem detalhe | **MANTER** | Nenhum | Maior diferencial da base; nenhuma capacidade do Activesoft o substitui |
| Presença diária com entrada/saída reais e fechamento noturno | Sim | Frequência por aula; entradas/saídas via catraca | **MANTER** | Nenhum | Ver C5 para coexistência com frequência por aula |
| Frequência por aula/disciplina com falta justificada/dispensada | Não | Sim — chamada por diário, lote, peso configurável da falta justificada na aprovação | **ACRESCENTAR** (por segmento) | Aditivo: segundo modo de frequência, selecionado pelo segmento da turma | C5: presença diária (infantil) e frequência por aula (fundamental+) coexistem por tipo de turma |
| Diário de classe (turma×disciplina×fase: aulas, conteúdo, tarefas) | Não | Sim — com ~25 parâmetros, prazos, criação automática de aulas pelo quadro de horários | **ACRESCENTAR** (por segmento) | Aditivo | Não substitui o diário do aluno da base — são entidades distintas com nomes parecidos (renomear no blueprint para evitar colisão semântica) |
| Motor de avaliação (sistemas por série, fases, fórmulas DSL, aprovação = nota E frequência, recuperação roteada) | Não | Sim — núcleo pedagógico, com fila assíncrona, confirmação professor→coordenação, snapshot de fórmulas | **ACRESCENTAR** (por segmento) | Aditivo de grande porte | Não hardcodar bimestres; valores do tenant são exemplos |
| Avaliação por relatório (descritiva) | Não — comunicados fazem papel informal | Sim — para educação infantil/Montessori, com confirmação do professor | **ACRESCENTAR** | Aditivo | **Encaixe direto no perfil da base (escola Montessori)** — provável primeiro passo pedagógico, antes de notas |
| Avaliação por competência | Não | Sim — índices, tipos, fases, planilha, relatório configurável | **ACRESCENTAR** (por segmento) | Aditivo | Alternativa moderna a notas para fundamental I |
| Conceitos (escala qualitativa) | Não | Sim — unidade única (nota OU percentual) travada pelo 1º conceito da série | **ACRESCENTAR** (por segmento) | Aditivo | Invariante nº 6 do Activesoft a preservar |
| Boletim configurável + regras de publicação | Não | Sim — ~60 flags, assinaturas, protocolo de recebimento, gate "bloquear com impedimento" | **ACRESCENTAR** (por segmento) | Aditivo | Gate de publicação explícito é padrão a adotar; bloqueio por impedimento sujeito a guard-rail legal (§7.7) |
| Histórico escolar | Não | Sim — geração automática ao concluir turma, cálculo de CH parametrizado, histórico externo | **ACRESCENTAR** (por segmento) | Aditivo | Obrigação documental de escola regular; depende do motor de avaliação |
| Ranking de alunos | Não | Sim — por fase/curso/série, exibível no boletim | **FORA DE ESCOPO** | — | `[SUPOSIÇÃO]` incompatível com a proposta pedagógica montessoriana da operadora atual da base; reavaliar se os segmentos-alvo mudarem |
| Reprovação por faltas / reprovar em recuperação (lote) | Não | Sim | **ACRESCENTAR** (por segmento) | Aditivo | Acompanha o motor de avaliação |
| Avaliação institucional (pesquisa família→escola) | Não | Sim | **ACRESCENTAR** | Aditivo | Canal família→escola de baixo custo e alto valor de retenção |
| Itinerários formativos / Novo Ensino Médio | Não | Sim | **FORA DE ESCOPO** (por segmento) | — | Só para segmento médio |
| Leitor de gabarito / simulado ENEM | Não | Sim (portal) | **FORA DE ESCOPO** (por segmento) | — | Idem |
| Ocorrências com categorias, data de liberação e visibilidade por categoria | Sim — ocorrências com comentários encadeados e menções | Sim — tipos (pedagógica/disciplinar/financeira/psicológica), liberação programada ao responsável, flag "gera impedimento" | **APRIMORAR** | Aditivo: tipologia + visibilidade por categoria + liberação programada sobre a entidade existente | Manter comentários encadeados da base (Activesoft não tem) |

### 4.4 Financeiro

| Capacidade | Na base | No Activesoft | Classificação | Impacto sobre a base | Observações |
|---|---|---|---|---|---|
| Boleto com cotas CMAE/CC/CRV | Sim | Não — Título genérico + tipo de serviço | **MANTER** | Nenhum | Ver C2: cota vira classificação do título no modelo unificado |
| Título/parcela + planos de pagamento (parcelas × serviços × descontos) | Parcial — boleto por serviço/vigência, dia de vencimento por aluno | Sim — plano como pacote ofertável na matrícula, anuidade, parcela "NN/NN" | **CONFLITO** (estrutural) | Ver §7.2 | Coexistência: plano de pagamento como camada de oferta sobre o modelo de serviços/valores da base |
| CNAB remessa/retorno (Santander, Sicoob) | Sim | Sim (CNAB 400, agentes de cobrança configuráveis) | **MANTER** + **APRIMORAR** | Aditivo: "agente de cobrança" como cadastro (banco, formato, flags) em vez de 2 bancos fixos | Arquitetura de adaptadores bancários plugáveis |
| Registro online de boleto via API bancária (BB/CAIXA/Itaú, bolecode) | Não | Sim | **ACRESCENTAR** | Aditivo: novas formas de recebimento | Reduz dependência do ciclo remessa/retorno |
| PIX (QR, conciliação automática, boleto híbrido) | Não | Sim | **ACRESCENTAR** | Aditivo | Prioridade alta — padrão de mercado brasileiro em 2026 |
| Multa/juros/mora | Sim — regra fixa (2% multa, juro diário mín. R$ 0,01, desconto até vencimento) | Cadastro nomeado reutilizável (% ou valor; ao mês ou ao dia) + default por unidade | **APRIMORAR** | Estrutural-leve: regra fixa vira o registro default do cadastro | C9: preservar o comportamento atual como seed imutável de migração |
| Vencimento apenas em dia útil; dia 1–28 por aluno | Sim | Feriado com efeito financeiro; **dia preferencial por responsável** | **APRIMORAR** | Aditivo: dia preferencial passa do aluno para o responsável financeiro (com migração 1:1) | Semântica próxima; o vínculo ao pagador é mais correto |
| Descontos: condicional (pontualidade) × permanente, ordem de cálculo, solicitação pelo portal | Parcial — desconto até o vencimento + bolsas com vigência | Sim — cadastro com regra de concessão, ordem de cálculo em cascata, workflow de solicitação com termo de consentimento, observação automática no aluno | **APRIMORAR** | Aditivo | Relatórios contábeis tratam condicionais separadamente (invariante 18) |
| Régua de cobrança automática configurável | Parcial — avisos fixos (vence hoje; vencido com reaviso a cada 3 dias) | Sim — regras por offset de dias × canal × serviço × forma, texto por regra, "só se registrado" | **APRIMORAR** | Estrutural-leve: os avisos atuais viram 2 regras default da régua | Alinha com §12.9 (limites herdados a reavaliar) |
| Negociação de dívida | Não | Sim — entidade de 1ª classe: título derivado, multa/juros próprios, descontos condicionais mantidos, exclusão dos relatórios contábeis | **ACRESCENTAR** | Aditivo | Alto valor; respeitar imutabilidade do boleto liquidado da base |
| Telecobrança (cobrança ativa com agenda de contato) | Não | Sim | **ACRESCENTAR** | Aditivo | Complementa o painel de cobrança da base |
| Caixa físico (abertura/fechamento nominal, cheques, custódia, cartão com repasse D+n, PIX presencial) | Não | Sim — módulo completo | **ACRESCENTAR** (condicional) | Aditivo | `[LACUNA]` a base não diz se a escola recebe presencialmente; decidir na etapa de blueprint |
| Contas a pagar / favorecidos | Parcial — Movimento de despesa + fornecedores | Sim — fornecedor, empresa, plano de contas multi-select, centro de resultado, situação | **APRIMORAR** | Aditivo sobre Movimento/Fornecedor | |
| Plano de contas hierárquico + centro de resultado | Parcial — classificação de custo plana | Sim — sintética/analítica, natureza, nº contábil para exportação | **APRIMORAR** | Estrutural-leve: classificação plana migra para hierarquia | |
| Conciliação bancária (extrato × lançamentos) | Parcial — conciliação boleto×movimento e prova real | Sim — por conta financeira, com data de início e importação de extrato (PJ Bank) | **APRIMORAR** | Aditivo | Mantém as rotinas de prova real da base |
| Fluxo de caixa previsto/realizado (DFC) | Não | Sim | **ACRESCENTAR** | Aditivo | |
| Exportação contábil | Não | Sim — com nº de conta contábil e data de corte | **ACRESCENTAR** | Aditivo | |
| NFS-e (RPS, lotes, certificado) | Sim — webservice municipal, simulação prévia, controle mensal | Sim — + variáveis na discriminação (`[NOMEALUNO]`…), recriação de lotes rejeitados | **MANTER** + **APRIMORAR** | Aditivo (variáveis, reprocessamento de rejeitados) | `[SUGESTÃO ALÉM DAS FONTES]`: avaliar hub fiscal multi-município na etapa de blueprint, citado no Activesoft como recomendação |
| Declaração de imposto de renda | Não | Sim — no portal do responsável, períodos configuráveis | **ACRESCENTAR** | Aditivo | Demanda recorrente de pais em escola privada |
| Recibos / 2ª via | Parcial — visualização/PDF do boleto | Sim — recibo formal, 2ª via, carnê | **APRIMORAR** | Aditivo | |
| Pagamento a menor/a maior com regras e travas | Não — boleto liquidado imutável | Sim — desconto automático limitado, diferença em novo título, travas de sanidade (40% a menor; >180 dias) | **ACRESCENTAR** | Aditivo: regras de liquidação antes da imutabilidade | Compatível: o título continua imutável após liquidado; as regras agem na liquidação |
| Data de bloqueio de operações financeiras (fechamento contábil) | Não | Sim — com operadores autorizados a ultrapassar | **ACRESCENTAR** | Aditivo | Override auditado, no padrão do Activesoft |
| Impedimentos transversais por canal (OBS/FNC/COB/BIB/PDM) | Não — cobrança não bloqueia acesso; parente mantém acesso até data efetiva do cancelamento | Sim — pendências bloqueiam matrícula/boletim/caixa/portal/agendamento, por canal, com badges e overrides | **CONFLITO** | Ver §7.7 | Adotar o mecanismo com guard-rails legais (Lei 9.870/1999) e preservando o acesso do responsável legal aos dados do filho |
| Garantidora de recebíveis (ISAAC) | Não | Sim — integração profunda, título garantido não editável localmente | **FORA DE ESCOPO** | — | Parceria comercial do grupo Arco, não disponível ao SIGE. O *conceito* de conector de garantidora fica registrado como opção futura (§8) |
| Multi-empresa (CNPJs de faturamento por unidade) | Não declarado — mas certificados fiscais "renovados anualmente **por CNPJ**" sugerem >1 | Sim — Empresa como entidade fiscal, caixa multi-empresa, aglutinação de títulos | **ACRESCENTAR** (condicional) | Aditivo | `[LACUNA]` confirmar quantos CNPJs a operação da base tem hoje |
| Bolsas com vigência monitorada e auditoria de descontos | Sim — rotinas de alerta e verificação | Concessão por aluno×turma×vigência com autorização e origem da solicitação | **MANTER** + **APRIMORAR** | Aditivo (workflow de autorização) | |
| Filantropia / CEBAS (ficha socioeconômica, renda per capita) | Não | Sim — módulo completo | **FORA DE ESCOPO** (condicional) | — | Só para escolas com certificação de filantropia; `[SUPOSIÇÃO]` não é o caso da operação atual |

### 4.5 Captação e comercial

| Capacidade | Na base | No Activesoft | Classificação | Impacto sobre a base | Observações |
|---|---|---|---|---|---|
| Prospect + interações + funil por status | Sim — incl. "por nascer", agrupamento pela idade, anexos | Sim — funil com oportunidades ganhas/perdidas em dashboard | **MANTER** + **APRIMORAR** | Aditivo: KPIs de funil (ganhas/perdidas) no dashboard | "Por nascer" é diferencial da base para creche |
| Formulário público de agendamento de visita | Sim — com e-mails automáticos e alerta de duplicidade | Captação pública por link com código da instituição | **MANTER** | Nenhum | |
| Papel "vendedor de captação" + WhatsApp do consultor | Não — colaborador responsável genérico | Sim — tipo de usuário próprio, botão de WhatsApp | **ACRESCENTAR** (leve) | Aditivo | WhatsApp manual, sem trilha — se adotado, registrar a interação no CRM da base |
| Indicações de famílias | Sim | Não observado | **MANTER** | Nenhum | |

### 4.6 Comunicação

| Capacidade | Na base | No Activesoft | Classificação | Impacto sobre a base | Observações |
|---|---|---|---|---|---|
| Comunicados com aprovação, menções, leitura confirmada, lixeira | Sim | Comunicado broadcast com rich-text, agendamento por data de publicação | **MANTER** + **APRIMORAR** | Aditivo: agendamento de publicação e audiência estruturada | |
| Audiência estruturada (turmas + situação do aluno + visibilidade) | Parcial — turmas/alunos destinatários | Sim — multi-select de turmas × filtro por situação × toggle de visibilidade | **APRIMORAR** | Aditivo | Depende do meta-modelo de situação (C6) |
| Circulares individualizadas com confirmação de leitura e reenvio | Sim | Não há equivalente direto (protocolo de recebimento existe só para boletim) | **MANTER** | Nenhum | Vantagem da base |
| Fotos com dupla aprovação e compartilhamento seletivo | Sim | Não observado | **MANTER** | Nenhum | Diferencial de privacidade da base; soma-se ao termo de uso de imagem (4.7) |
| Caixa de saída (trilha de auditoria universal de envios) | Parcial — notificações internas espelham pushes | Sim — toda mensagem (e-mail/SMS/mobile) logada com destinatário, canal, data e log de envio; reenvio em lote | **ACRESCENTAR** | Aditivo: camada de log universal sob os envios já existentes | "Prova de entrega da escola" — um dos padrões de maior valor do Activesoft |
| Chat 1:1/grupo escola↔família | Não — comunicação é por comunicado/circular/ocorrência | Sim — Conversas do App, com tipos de mensagem e contador de não lidas | **ACRESCENTAR** | Aditivo | Decisão de produto: chat muda a operação da escola (SLA de resposta); recomendar adoção com janelas e responsáveis definidos |
| Templates HTML de e-mail com variáveis | Não — e-mails fixos por gatilho | Sim — `texto_personalizado` com variáveis e cabeçalho/rodapé | **APRIMORAR** | Aditivo: gatilhos atuais passam a referenciar templates editáveis | |
| Campanhas push do administrador | Sim — com filtro por atribuição | Push via app próprio/ClassApp | **MANTER** | Nenhum | ClassApp FORA DE ESCOPO (base tem app próprio) |
| Banner/aviso de tela inicial | Não | Sim — global | **ACRESCENTAR** (leve) | Aditivo | |
| Envio de credenciais em massa (link de recuperação) | Não — senha derivada enviada na criação (anti-padrão §12.3) | Sim — e-mail com link de redefinição, por tipo de acesso, só com e-mail cadastrado | **APRIMORAR** | Substitui o mecanismo condenado pelo §12.3 | Convergência: implementar como "convite com definição de senha pelo usuário" |
| Contato com a instituição por setor (e-mails secretaria/tesouraria/direção) | Não | Sim — in-app, com tipo "sugestão ou crítica" | **ACRESCENTAR** (leve) | Aditivo | |
| Documentos institucionais em capítulos | Sim | Porta-arquivos (upload pela família/professores) | **MANTER** + **ACRESCENTAR** (porta-arquivos) | Aditivo | Porta-arquivos também serve aos procedimentos de matrícula (upload de documentos) |

### 4.7 LGPD, termos e privacidade

| Capacidade | Na base | No Activesoft | Classificação | Impacto sobre a base | Observações |
|---|---|---|---|---|---|
| Termo de consentimento LGPD com aceite datado | Não — LGPD tratada como NFR de proteção de acesso | Sim — texto por unidade, ativo, exibido em módulos definidos, aceite com data/hora, canal do DPO | **ACRESCENTAR** | Aditivo | Prioridade legal; dados da base incluem saúde de menores |
| Termo de uso de imagem | Não — dupla aprovação de fotos é controle operacional, sem base de consentimento registrada | Sim — texto rico por unidade, aceite no fluxo de matrícula | **ACRESCENTAR** | Aditivo: o fluxo de fotos da base passa a verificar o consentimento vigente | Composição forte: dupla aprovação (base) + consentimento formal (Activesoft) |
| Segregação de dados sensíveis na API | Não declarado | Sim — endpoints `*_dados_sensiveis` auto-declarados | **ACRESCENTAR** | Aditivo (contrato de API) | Com escopo de autorização próprio e auditoria por token (corrigindo o anti-padrão do Bearer único) |
| CPF mascarado em tela | Não declarado | Sim | **ACRESCENTAR** (leve) | Aditivo | |
| Solicitação de alteração cadastral pelo responsável com consentimento | Parcial — app permite editar cadastro próprio | Sim — flags por campo (CPF, RG, endereço, filiação) + termo | **APRIMORAR** | Aditivo: granularidade por campo | |

### 4.8 Compliance e obrigações legais

| Capacidade | Na base | No Activesoft | Classificação | Impacto sobre a base | Observações |
|---|---|---|---|---|---|
| Educacenso / Censo Escolar (INEP) | **Não consta** | Sim — exportação por blocos 00–60, validação de obrigatórios, códigos INEP em série/turma/aluno/colaborador | **ACRESCENTAR** | Aditivo, mas com pré-requisitos de cadastro (dados censitários, códigos INEP) | Obrigação legal de toda escola. `[LACUNA]` como a escola da base declara o censo hoje (provavelmente à mão no sistema do INEP) |
| NFS-e municipal | Sim | Sim | **MANTER** | — | Já coberto em 4.4 |
| Secretaria digital (documentos escolares eletrônicos assinados) | Não | Sim (módulo satélite) | **ACRESCENTAR** (condicional, por segmento) | Aditivo | Valor cresce com fundamental+ (declarações, transferências, históricos) |

### 4.9 Portais e aplicativo

| Capacidade | Na base | No Activesoft | Classificação | Impacto sobre a base | Observações |
|---|---|---|---|---|---|
| App móvel para responsáveis (diário, financeiro, circulares, fotos, renovação) | Sim | Portal/app do responsável (lado de dentro **inferido**, nunca capturado) | **MANTER** | Nenhum | A base é a fonte confiável aqui — o BP do Activesoft declara essa área como inferida |
| App para colaboradores (rotina de sala) | Sim | Portal do professor (notas/diário/chamada) | **MANTER** | Nenhum | |
| Portal web do responsável | Não — responsável só tem app | Sim — mesmo domínio, claim JWT | **ACRESCENTAR** | Aditivo: front web sobre a mesma API do app | Reduz exclusão de famílias sem smartphone adequado |
| Portal do aluno | Não — alunos são bebês/crianças pequenas | Sim — boletim, atividades, carteirinha, porta-arquivos | **ACRESCENTAR** (por segmento) | Aditivo | Sem sentido no infantil; necessário de fundamental II em diante |
| Portal do professor (notas, chamada online, confirmação) | Não | Sim | **ACRESCENTAR** (por segmento) | Aditivo | Acoplado ao motor de avaliação |
| Matriz de permissões persona × situação do aluno × recurso (~100 chaves) | Parcial — atribuições por parente×aluno + flags de colaborador | Sim — controle fino do que cada portal exibe, por situação | **APRIMORAR** | Estrutural-leve — ver C4 | Compor: atribuição (quem vê o quê de qual aluno) × situação (quando) × recurso (o quê) |
| Carteirinha estudantil digital | Não | Sim — com validade configurável | **ACRESCENTAR** (leve) | Aditivo | |
| Edição de ficha médica pelo responsável | Não (ficha médica não existe como entidade) | Sim — toggle visualizar/editar | **ACRESCENTAR** | Aditivo (depende da ficha médica, 4.1) | Para creche: responsável mantém alergias/medicações atualizadas — alto valor |
| Verificação de versão mínima do app / deep links | Sim | Não observado | **MANTER** | Nenhum | |

### 4.10 Administração, plataforma de uso e relatórios

| Capacidade | Na base | No Activesoft | Classificação | Impacto sobre a base | Observações |
|---|---|---|---|---|---|
| Multiunidade com extensão de unidade | Sim | Multi-unidade + multi-empresa (3 níveis com Instituição) | **MANTER** + ver multi-empresa (4.4) | Nenhum | "Instituição" como nível de plataforma só importa se o SIGE virar multi-escola — decisão da etapa de blueprint, fora desta análise |
| Auditoria (log de quem fez o quê) | Sim — entidade Log em entidades sensíveis | Onipresente — botão de auditoria por tela, campos de autorização embutidos no registro, overrides com justificativa | **APRIMORAR** | Aditivo: padronizar trilha por entidade + autoria embutida em todo override | |
| Soft delete / exclusão lógica | Sim — padrão declarado | Sim — com motivo, comentário e destino | **MANTER** + **APRIMORAR** (motivo/destino) | Aditivo | |
| Operações em lote com etapa de simulação | Parcial — renovação em lote, importações | Sim — framework genérico: filtrar → editar → **simular** → processar, com fila e log por item | **APRIMORAR** | Aditivo: o desenho de lote vira padrão de plataforma | Um dos 10 pilares do Activesoft |
| Fila de processamento assíncrono visível ao usuário (status, erro, retry) | Não — rotinas agendadas opacas | Sim — `fila_processamento` com log por item | **ACRESCENTAR** | Aditivo: as ~20 rotinas da base ganham visibilidade e reprocessamento | Reduz a dependência de "rotinas de correção" condenadas pelo §12.5 |
| Central de relatórios (favoritos, fila de exportação, modelos) | Parcial — relatórios dispersos por módulo + CSV | Sim — central com modelos, favoritos, relatórios exportados | **APRIMORAR** | Aditivo | |
| Gerador de consultas ad-hoc (SQL) | Não | Sim — Monaco/SQL, consultas salvas; **anti-padrão de segurança apontado pelo próprio BP** | **CONFLITO** (com a postura de segurança) | Ver §7.10 | Se adotado: somente leitura, whitelist de objetos, mascaramento de PII e auditoria por execução; alternativa: não adotar e investir na central de relatórios |
| Dashboard de KPIs (ocupação, inadimplência R$, funil, contratos >14d) | Parcial — totais de prospects/matrículas/cancelamentos | Sim — KPIs acionáveis com link para a lista-alvo | **APRIMORAR** | Aditivo | |
| BI embarcado (dashboards por unidade) | Não | Sim — Metabase via JWT com row-level security por unidade | **ACRESCENTAR** | Aditivo | Ferramenta livre; o requisito é "BI embarcado com segregação por unidade" |
| Feature flags (escopos global/tenant/perfil) | Não | Sim — 3 camadas | **ACRESCENTAR** (adaptado) | Aditivo | Para o SIGE: flags internas de lançamento gradual, não tiers comerciais |
| Catraca / controle de acesso físico + frequência | Parcial — registro manual de saída com pessoa autorizada | Sim — identificador, digital, API `marcar_frequencia`/`identificar_marcacao`, "quem pode retirar o aluno" | **ACRESCENTAR** (condicional) | Aditivo | Alto valor de segurança para creche (retirada do aluno); depende de hardware |
| ViaCEP (autocomplete de endereço) | Não | Sim | **ACRESCENTAR** (leve) | Aditivo | |
| Google for Education (provisionamento de contas) | Não | Sim — 3 regras de geração de e-mail | **ACRESCENTAR** (condicional) | Aditivo | Só se a escola usar GFE |
| Microsoft Teams / editoras (Edebê) | Não | Sim | **FORA DE ESCOPO** (condicional) | — | Dependem de parcerias que a base não tem; o padrão "provisionamento para plataformas de conteúdo" fica via API pública |
| API pública para parceiros | Não — API é exclusiva do app próprio | Sim — 40 endpoints (sync de cadastros, boletim, frequência, cobrança, SSO de portal) | **ACRESCENTAR** | Aditivo | Com OAuth2 client-credentials + escopos (lição do anti-padrão Bearer único) |
| Biblioteca / livraria | Não | Sim — módulos satélites (geram pendência BIB) | **FORA DE ESCOPO** (condicional) | — | Sem demanda na base; reavaliar por segmento |
| Exportação publicitária (públicos de mídia) | Sim | Não observado | **MANTER** | Nenhum | Condicionar ao consentimento LGPD (4.7) quando os termos existirem |
| Help center contextual / suporte embutido | Não | Sim — Zendesk + assistente por rota (recurso de plataforma) | **FORA DE ESCOPO** | — | Operação de suporte do fornecedor; um help simples pode entrar depois como `[SUGESTÃO ALÉM DAS FONTES]` |
| Central de novidades / treinamento gamificado | Não | Sim (recurso de plataforma) | **FORA DE ESCOPO** | — | Sucesso do cliente do fornecedor multitenant |

---

## 5. Acréscimos candidatos (ACRESCENTAR)

Capacidades novas, aditivas e sem colisão com a base, com o valor que trazem. Ordenadas por valor para o perfil atual da base (infantil multiunidade), depois condicionais.

### 5.1 Valor imediato para o perfil atual

1. **Termos LGPD digitais (consentimento + uso de imagem) com aceite datado** — fecha o risco legal mais exposto da base (fotos de menores e dados de saúde sem consentimento registrado); compõe com a dupla aprovação de fotos existente.
2. **Educacenso/INEP** — obrigação legal de toda escola; hoje presumivelmente cumprida fora do sistema `[SUPOSIÇÃO]`. Arrasta os pré-requisitos: dados censitários e códigos INEP nos cadastros.
3. **Caixa de saída (trilha de auditoria universal da comunicação)** — "prova de entrega" da escola perante as famílias; unifica o rastreio que a base já faz parcialmente (circulares, notificações) em um log único multicanal com reenvio.
4. **PIX + registro online de boleto (APIs bancárias)** — moderniza o recebimento; reduz dependência do ciclo remessa/retorno e acelera a baixa.
5. **Negociação de dívida** — entidade própria (título derivado, multa/juros próprios, fora dos relatórios contábeis); hoje a base não tem caminho estruturado para acordos.
6. **Ficha médica estruturada + edição pelo responsável** — para creche, alergias/restrições alimentares conectam diretamente com a rotina de refeições e medicações da base. (Classificada APRIMORAR na matriz por haver fragmentos na base; o grosso é acréscimo.)
7. **Avaliação por relatório (descritiva)** — primeiro degrau pedagógico aderente ao perfil Montessori da operação atual, sem exigir o motor de notas completo.
8. **Procedimentos de matrícula (checklist documental)** — organiza documentos de matrícula com prazo, obrigatoriedade e upload pelo responsável (via porta-arquivos).
9. **Matrícula online self-service (ficha pública + RM online)** — reduz trabalho de secretaria; desemboca no fluxo de requerimento/contrato existente.
10. **Avaliação institucional (pesquisa família→escola)** — canal de retenção barato.
11. **Fila de processamento assíncrono visível + framework de lote com simulação** — dá transparência e reprocessamento às ~20 rotinas da base; reduz a classe de problemas que gerou as "rotinas de correção" (§12.5).
12. **Chat 1:1 escola↔família** — eleva a comunicação a tempo real; exige decisão operacional (responsáveis e janelas de resposta).
13. **Impersonação auditada** — substitui a senha-mestre global (§12.1); convergência dos dois BPs.
14. **API pública de parceiros (OAuth2 + escopos)** — abre integrações (catraca, plataformas de conteúdo, contabilidade) sem acoplar o núcleo.
15. **Pagamento a menor/a maior com travas; data de bloqueio contábil; DFC; exportação contábil; declaração de IR; telecobrança; sequenciais de matrícula nomeados; banner global; ViaCEP; carteirinha digital; campos dinâmicos; nome social; contato por setor; porta-arquivos; BI embarcado; feature flags internas; central de KPIs** — acréscimos de menor porte, todos aditivos.

### 5.2 Condicionais (decidir na etapa de blueprint)

- **Caixa físico completo** (cheques, cartão com repasse, custódia) — só se houver recebimento presencial relevante `[LACUNA]`.
- **Multi-empresa (CNPJs)** — confirmar a estrutura societária da operação `[LACUNA]`.
- **Catraca/controle de acesso físico** — depende de hardware; alto valor de segurança na retirada de alunos de creche.
- **Google for Education** — só se a escola adotar o ecossistema.
- **Secretaria digital** — valor cresce com segmentos formais.

### 5.3 Por segmento (ativados se o SIGE mirar fundamental+)

Motor de avaliação completo (sistemas/fases/fórmulas/aprovação nota E frequência), diário de classe, frequência por aula, conceitos, avaliação por competência, boletim + regras de publicação, histórico escolar, promoção/finalização em lote, reprovação por faltas, portal do aluno, portal do professor, quadro de horários docente, formação/alocação didática, disciplinas e grade curricular.

---

## 6. Aprimoramentos candidatos (APRIMORAR)

Para cada item, a estratégia de coexistência com a base (que não pode ser descontinuada).

| # | Aprimoramento | O que a base já faz | O que o Activesoft acrescenta | Estratégia de coexistência |
|---|---|---|---|---|
| A1 | **Situação do aluno como meta-modelo** | Estados fixos: ativo / cancelado (reativável) / finalizado | Cadastro configurável mapeado a domínios de sistema (ativo/inativo), acadêmico, Educacenso, + flags de comportamento | Criar o cadastro de situações **semeado com os estados atuais da base**, com mapeamento 1:1; comportamento existente (cancelamento reativável, acesso até a data efetiva) vira propriedade das situações seed. Nada muda para a escola no dia 1 |
| A2 | **Régua de cobrança configurável** | Avisos fixos: vence hoje; vencido com reaviso a cada 3 dias | Regras por offset×canal×serviço×forma com texto próprio | As 2 regras atuais viram seeds da régua; o comportamento default é idêntico ao atual |
| A3 | **Multa/juros como cadastro nomeado** | Regra única hardcoded (2% multa; juro diário ≥ R$ 0,01; desconto até o vencimento) | Cadastros nomeados (% ou valor, mês ou dia) com default por unidade | Regra atual vira o registro default imutável na migração; novas regras são opcionais. Boleto liquidado permanece imutável |
| A4 | **Dia de vencimento por responsável** | Dia 1–28 por **aluno** | Dia preferencial por **responsável financeiro** | Migrar o dia do aluno para o seu responsável financeiro (1:1 hoje); manter override por aluno para casos divergentes |
| A5 | **Descontos com regra de concessão e ordem de cálculo** | Desconto por pontualidade embutido no boleto; bolsas com vigência | Condicional × permanente, cascata ordenada, workflow de solicitação | Desconto-pontualidade atual vira desconto condicional seed; bolsas mantêm vigência/auditoria e ganham workflow de autorização |
| A6 | **Matriz de permissões dos portais** | Atribuições por parente×aluno (9 códigos); flags de colaborador | Persona × situação do aluno × recurso (~100 chaves) | Composição em camadas: atribuição responde "quem vê o quê de qual aluno"; situação responde "quando"; recurso refina "o quê". Responsável legal mantém acesso pleno (regra da base prevalece) |
| A7 | **Auditoria onipresente** | Log em entidades sensíveis | Trilha por entidade em toda tela de config/financeiro; override sempre com usuário+justificativa+data | Estender o Log existente como padrão de plataforma; todo bloqueio/override novo nasce auditado |
| A8 | **Operações em lote com simulação** | Renovação em lote, importações | Filtrar→editar→simular→processar com fila e log por item | A renovação em lote da base é o primeiro caso migrado para o framework; o requerimento de renovação continua sendo o artefato formal gerado |
| A9 | **Feriados com efeitos tipados + cópia entre períodos** | Feriados deslocam vencimento e dias letivos | Efeito financeiro/biblioteca por flag, escopo por unidade, cópia ano a ano | Resolve §12.7; efeitos atuais viram flags ligadas por default |
| A10 | **Ocorrências tipadas com visibilidade por categoria** | Ocorrência com comentários encadeados, menções, controle de visualização | Tipos (pedagógica/disciplinar/financeira/psicológica), data de liberação ao responsável, flag de impedimento | Tipologia e liberação programada são atributos novos da entidade atual; comentários encadeados (só a base tem) permanecem |
| A11 | **Templates de e-mail editáveis com variáveis** | E-mails transacionais fixos por gatilho | Templates HTML com variáveis e identidade da escola | Gatilhos atuais passam a apontar para templates seed idênticos aos textos atuais |
| A12 | **Convite/redefinição de senha em massa** | Senha derivada de CPF enviada na criação (condenada por §12.3) | Link de redefinição em massa por tipo de acesso | Implementação direta do §12.3 usando o desenho do Activesoft (convite com definição de senha pelo usuário) |
| A13 | **Plano de contas hierárquico + centro de resultado** | Classificação de custo plana | Sintética/analítica, natureza, conta para exportação | Classificações atuais migram como contas analíticas sob uma hierarquia mínima |
| A14 | **Central de relatórios** | Relatórios por módulo + CSV | Modelos, favoritos, fila de exportação | Relatórios atuais são catalogados na central sem mudar seu conteúdo |
| A15 | **Agente de cobrança como cadastro** | Santander e Sicoob fixos | Banco/carteira/formato/flags configuráveis | Os 2 bancos viram registros seed; novos bancos não exigem código (adaptadores) |
| A16 | **Agendamento self-service** | Agendamento criado pela equipe | Disponibilidades + marcação pelo responsável | Agendamento atual continua; o self-service é um canal novo de criação |
| A17 | **Audiência estruturada de comunicados** | Destinatários por turmas/alunos | Turmas × situação × toggle de visibilidade | Aditivo; depende de A1 |
| A18 | **KPIs acionáveis no dashboard** | Totais simples | Ocupação, inadimplência R$, funil, contratos >14 dias, cada um linkando para a lista | Os gráficos atuais permanecem; KPIs viram cards adicionais |
| A19 | **Enturmação com autorização auditada** | Aluno↔turma + log | Matrícula autorizada apesar de impedimento com justificativa/autor; situação de exceção | Atributos novos; só ganham uso quando impedimentos (C7) existirem |
| A20 | **Restrições de elegibilidade de turma** | Agrupamento etário | Nascimento mín/máx, sexo, pré-requisito de vínculo | O agrupamento continua sendo a regra primária; restrições adicionais são opcionais por turma |

---

## 7. Conflitos (CONFLITO) — colisão e caminho proposto

### 7.1 C1 — Estrutura acadêmica: Atendimento etário × Série/Disciplina/Grade
**Colisão:** a base modela o aluno por **Atendimento** (curso → agrupamento etário em meses → nível → turno → permanência → horário, turma por tipo de serviço, sem disciplinas); o Activesoft modela Período → Curso → **Série** → Turma → **Disciplina** via Grade Curricular, com a série carregando o serviço financeiro e a próxima série. São duas espinhas dorsais diferentes para o mesmo lugar do domínio.
**Caminho — coexistência por generalização:** modelo unificado em que (a) Agrupamento/Nível da base são preservados como dimensões obrigatórias do segmento infantil, com a derivação automática por data de nascimento intocada; (b) Série/Disciplina/Grade entram como **extensão opcional por segmento** (o Activesoft mesmo tem `utiliza_rotina_educacao_infantil` e a pseudo-disciplina "FREQUÊNCIA DIÁRIA", mostrando que o modelo dele já degrada para o caso sem notas); (c) Permanência e Horário — que não existem no Activesoft — permanecem como dimensões próprias da base (são a âncora do modelo comercial de creche). **Nada da base é descontinuado**; escolas infantis nunca veem disciplina/grade.

### 7.2 C2 — Modelo de cobrança: cotas CMAE/CC/CRV + Serviço por atributos × Título/Plano de Pagamento
**Colisão:** a base emite **Boletos tipados por cota** a partir de Serviços definidos por combinação de atributos com Valor por vigência; o Activesoft trabalha com **Título/parcela** genérico e **Planos de Pagamento** (pacote serviços × parcelas × valores ofertado na matrícula), com a série/turma carregando o serviço de mensalidade.
**Caminho — coexistência por camadas:** (a) o conceito unificador é o **Título** (a base "Boleto" é um título com forma de recebimento boleto) — generalização necessária de toda forma para acomodar PIX/cartão; (b) **tipo de cota (CMAE/CC/CRV) vira classificação obrigatória do título no segmento da base** — não some; (c) o Serviço por atributos + Valor por vigência da base permanece como motor de precificação; (d) Plano de Pagamento entra como **camada de oferta** opcional acima (empacota serviços e parcelamento para matrícula online); (e) invariantes da base prevalecem: valores em centavos, vencimento em dia útil, boleto liquidado imutável.

### 7.3 C3 — Efetivação de matrícula: assinatura do contrato × pagamento da pré-matrícula
**Colisão:** na base, a matrícula se efetiva quando **todos assinam o contrato** (requerimento contratual); no Activesoft, a situação do aluno muda **mediante pagamento** do título de pré-matrícula (qualquer parcela ou todas) e o contrato (Clicksign) corre em paralelo com SLA de 14 dias.
**Caminho — gatilhos componíveis:** o fluxo de Requerimento da base permanece o mecanismo formal único; a efetivação ganha **condições configuráveis por escola**: assinatura (default atual), pagamento, ou ambos. A pré-matrícula por pagamento entra como estado intermediário opcional ("reserva de vaga paga") antes do contrato — aliás compatível com a Cota de Reserva de Vaga (CRV) que a base já cobra. SLA e cancelamento automático por prazo de assinatura entram como aprimoramento do painel de contratos não assinados já existente.

### 7.4 C4 — Permissões da família: atribuições por parente×aluno × matriz situação×recurso
**Colisão:** dois sistemas de autorização diferentes para o mesmo sujeito (responsável/parente).
**Caminho — composição em camadas (sem substituição):** a **atribuição por parente×aluno permanece a fonte primária** ("a avó vê fotos, não vê financeiro"); a matriz do Activesoft entra como filtro adicional por **situação do aluno** (ex.: cancelado-finalizado desliga recursos) e por **recurso global** (escola desliga um módulo do portal inteiro). Regra de precedência: o mais restritivo vence, **exceto** para o responsável legal, cujo acesso pleno aos dados do próprio filho (princípio da base) não pode ser desligado por configuração — ver também C7.

### 7.5 C5 — Frequência: presença diária com horários reais × frequência por aula
**Colisão:** a base registra presença **diária** com entrada/saída reais e o conceito de presença parcial (que carimba todos os registros do diário do dia); o Activesoft registra frequência **por aula** dentro de diários de classe, com falta justificada/dispensada e peso na aprovação.
**Caminho — dois modos por tipo de turma:** turmas do segmento infantil usam exclusivamente o modo da base (intocado, incluída a presença parcial e o fechamento noturno); turmas de segmentos formais usam o modo por aula. Os dois alimentam um mesmo conceito de "apuração de frequência" para relatórios. Não há fusão de modelos — há seleção por segmento.

### 7.6 C6 — Estados do aluno: fixos e reativáveis × meta-modelo configurável
**Colisão:** o cancelamento da base tem semântica fina (reativável enquanto não finalizado; acesso do parente até a data efetiva; interrompe boletos futuros; cancela requerimentos pendentes) que um meta-modelo genérico pode quebrar.
**Caminho — meta-modelo com seeds protegidos:** adotar o cadastro de situações (A1) com as situações da base criadas como **registros de sistema não removíveis**, cujos efeitos (reativação, acesso temporário, corte de cobrança) são propriedades declarativas. Escolas podem acrescentar situações, não remover as semânticas da base. O mapeamento para Educacenso entra como atributo adicional de cada situação.

### 7.7 C7 — Impedimentos por inadimplência × acesso pleno do responsável e limites legais
**Colisão:** o Activesoft bloqueia boletim/matrícula/caixa/portal/agendamento por pendência (configurável por canal); a base garante acesso do parente até a data efetiva do cancelamento e acesso irrestrito do responsável legal. Além da colisão de princípio, há limite legal: a **Lei 9.870/1999** (e o ECA) veda reter documentos escolares e aplicar penalidades pedagógicas por inadimplência; é lícito recusar a **renovação** da matrícula.
**Caminho — adotar com guard-rails:** o mecanismo de impedimento entra (badges OBS/FNC/COB/PDM, overrides auditados), mas com restrições de fábrica não configuráveis: (a) nunca bloquear o acesso do responsável legal aos **dados do filho** (diário, saúde, comunicados); (b) nunca reter documentos obrigatórios (declaração de transferência, histórico); (c) bloqueios permitidos: **rematrícula**, contratação de serviços opcionais, agendamentos comerciais; (d) bloqueio de boletim no portal: desaconselhado por risco jurídico — se oferecido, desligado por default com aviso legal `[SUPOSIÇÃO: leitura conservadora da norma; validar com jurídico]`. Badge informativo é sempre permitido; bloqueio é exceção configurada.

### 7.8 C8 — Provedor de assinatura eletrônica: Autentique × Clicksign
**Colisão:** mesmo papel, fornecedores diferentes; a base tem fluxo multi-signatário com testemunhas e rotinas de verificação acopladas ao Autentique.
**Caminho — abstração de conector:** interface única de assinatura (criar documento, signatários, status, webhook) com o Autentique como implementação default (preserva contratos e credenciais por colaborador da base); Clicksign ou outros entram como adaptadores se necessário. Nenhum comportamento da base muda.

### 7.9 C9 — Push/app de comunicação: app próprio + OneSignal × ClassApp
**Colisão:** o Activesoft terceiriza o canal mobile (ClassApp); a base tem app próprio com OneSignal e atribuições finas.
**Caminho — descarte do ClassApp:** manter o app próprio (vantagem competitiva e de privacidade). O que se importa do ClassApp é o requisito implícito: **log auditável de entrega por canal** — atendido pela caixa de saída (§5.1.3).

### 7.10 C10 — Gerador de consultas SQL ad-hoc × postura de segurança
**Colisão:** capacidade real de autonomia da escola, mas o próprio BP do Activesoft a marca como superfície de ataque (SQL via Monaco sem whitelist), e a base carrega dados de saúde de menores.
**Caminho — recomendação de descarte na v1:** atender a necessidade com a central de relatórios + BI embarcado com row-level security. Se a demanda persistir, versão restrita: somente leitura, catálogo de objetos permitidos, mascaramento de PII por default e auditoria por execução.

---

## 8. Fora de escopo (com justificativa)

| Item do Activesoft | Justificativa |
|---|---|
| **ISAAC (garantidora de recebíveis)** | Parceria comercial do Grupo Arco indisponível ao SIGE; acopla o ciclo de vida do título a um terceiro. O conceito abstrato "conector de garantidora" fica registrado para o futuro, sem entrar no escopo |
| **ClassApp** | A base tem app próprio; terceirizar o canal removeria o diferencial (ver C9) |
| **Tiers comerciais (Light/Basic), whitelist/feature-flag por cliente para venda** | Recurso de comercialização de plataforma multitenant, não de uso da escola |
| **Usuários de suporte do fornecedor no ambiente do cliente; parâmetros "só Activesoft"** | Operação do fornecedor; substituído pela impersonação auditada (acréscimo adaptado) |
| **Central de novidades, roteiro de treinamento gamificado, assistente virtual/Zendesk** | Sucesso do cliente do fornecedor SaaS multitenant; sem função no contexto do SIGE |
| **Ranking de alunos** | `[SUPOSIÇÃO]` incompatível com a linha pedagógica montessoriana da operação da base; reavaliar apenas se os segmentos-alvo mudarem |
| **Filantropia/CEBAS** | Exclusivo de instituições com certificação de filantropia `[SUPOSIÇÃO: não é o caso]` |
| **Novo Ensino Médio / itinerários formativos; simulado ENEM; leitor de gabarito** | Segmento médio, fora do perfil atual; sinalizados na matriz caso os segmentos-alvo se ampliem |
| **Microsoft Teams / Edebê (editoras)** | Dependem de parcerias inexistentes; o caso de uso fica coberto genericamente pela API pública de parceiros |
| **Biblioteca / livraria** | Sem demanda declarada na base; condicional por segmento |
| **Banco por cliente / versionamento de API por migration / endpoints por versão de schema** | Decisões de implementação do fornecedor (e parcialmente anti-padrões); irrelevantes nesta etapa funcional |
| **Progressão parcial/dependência** | Mecânica de fundamental II/médio (por segmento) |

---

## 9. Lacunas e suposições

### 9.1 [LACUNA]

1. **Segmentos-alvo do SIGE não declarados** (campo opcional do processo em branco) — ~14 capacidades ficaram condicionais "(por segmento)"; é a decisão mais estruturante para a etapa 2.
2. **Educacenso na base** — o BP atual não menciona como a obrigação é cumprida hoje.
3. **Recebimento presencial** — a base não diz se a escola opera caixa físico; define a adoção do módulo de caixa/cheques/cartão.
4. **Multi-empresa** — "certificados renovados anualmente por CNPJ" (plural) sugere mais de um CNPJ, mas a base não modela Empresa; confirmar estrutura societária.
5. **Lado família dos portais do Activesoft** — nunca capturado por dentro (declarado no próprio BP); tudo sobre portal de responsável/aluno/professor do Activesoft é inferência. Para essas áreas, **a base é a fonte primária**.
6. **Renderização condicional do financeiro per-aluno no Activesoft** — marcada 🔴 (hipótese) na fonte.
7. **Entrega efetiva de e-mail/SMS/push no Activesoft** — inferida de release notes/schemas, não observada.
8. **Provedor de SMS da base** — não nomeado no BP.
9. **Ficha médica na base** — alergias/saúde citadas como NFR de sigilo, sem entidade consolidada (confirmado como acréscimo/aprimoramento).
10. **Valores concretos do Activesoft** (multa 2%, régua 14h, fases Fund II, R$ 26.158/ano) — parametrização do tenant CEMP; **nenhum** vale como default do SIGE.

### 9.2 [SUPOSIÇÃO]

1. Ranking de alunos é incompatível com a proposta pedagógica da operação atual (Montessori) — base da classificação FORA DE ESCOPO.
2. A escola da base não possui certificação de filantropia (CEBAS) — base do FORA DE ESCOPO condicional.
3. Leitura conservadora da Lei 9.870/1999 para o bloqueio de boletim por inadimplência (§7.7) — validar com jurídico.
4. A migração do dia de vencimento do aluno para o responsável financeiro (A4) assume relação 1:1 predominante hoje.
5. O Educacenso é cumprido hoje manualmente fora do sistema.

### 9.3 [SUGESTÃO ALÉM DAS FONTES]

1. Incluir **carteira de vacinação** entre os procedimentos de matrícula típicos do infantil.
2. Avaliar **hub fiscal multi-município** (eNotas/Focus/PlugNotas) se o SIGE for multi-cidade — citado no BP do Activesoft apenas como recomendação, não como capacidade.
3. Condicionar a **exportação publicitária** existente na base ao consentimento LGPD quando os termos digitais existirem.

---

## 10. Insumos para o blueprint (consolidado e priorizado)

Sem arquitetura, modelo de domínio ou roadmap — apenas o **que** segue para a etapa 2, em ordem de prioridade.

### P0 — Fundações transversais (condicionam o desenho de tudo)

1. Base integral do §2 como núcleo intocável (MANTER), já com as correções do §12 do BP atual incorporadas como requisitos.
2. Meta-modelo de **situação do aluno** com seeds protegidos da base (A1/C6).
3. **Estrutura acadêmica unificada** com agrupamento etário preservado e série/disciplina como extensão por segmento (C1).
4. **Título** como generalização do boleto, com cotas CMAE/CC/CRV preservadas e plano de pagamento como camada de oferta (C2).
5. **Auditoria onipresente + overrides justificados** (A7/A19) e **soft delete com motivo/destino**.
6. Framework de **operações em lote com simulação** + **fila assíncrona visível** (A8, §5.1.11).
7. **Caixa de saída** como trilha universal da comunicação (§5.1.3).
8. **Impedimentos com guard-rails legais** (C7) — desenho transversal que toca financeiro, portais e matrícula.
9. Requisitos negativos consolidados: §12 da base + anti-padrões do Activesoft (§3.4) — segurança de cookies/sessão, zero PII em URL/docs, tenant fora do path, política de senha moderna, stack de UI única com tokens, a11y como gate, governança de rotas/nomenclatura, jornadas como fluxos (não telas-ilha).

### P1 — Valor imediato para o perfil atual (infantil multiunidade)

10. Termos LGPD (consentimento + uso de imagem) integrados a fotos, matrícula e exportações.
11. Educacenso + dados censitários/códigos INEP nos cadastros.
12. PIX + registro online de boleto + agente de cobrança como cadastro (A15).
13. Régua de cobrança configurável (A2), multa/juros nomeados (A3), dia preferencial por responsável (A4), descontos estruturados (A5).
14. Negociação de dívida; pagamento a menor/a maior; data de bloqueio contábil.
15. Ficha médica estruturada + edição pelo responsável.
16. Matrícula online self-service + procedimentos de matrícula + pré-matrícula por pagamento como gatilho componível (C3).
17. Avaliação por relatório (descritiva).
18. Portal web do responsável (paridade com o app).
19. Convite/redefinição de senha (A12) e impersonação auditada.
20. Templates de e-mail (A11), audiência estruturada de comunicados (A17), ocorrências tipadas (A10), agendamento self-service (A16), feriados aprimorados (A9).
21. Central de relatórios (A14), KPIs acionáveis (A18), BI embarcado.
22. Chat 1:1 (com definição operacional), avaliação institucional, contato por setor, porta-arquivos, carteirinha, banner, ViaCEP, sequenciais de matrícula, campos dinâmicos, nome social.
23. API pública de parceiros com OAuth2/escopos e endpoints de dados sensíveis segregados.

### P2 — Condicionais (resolver as lacunas antes de especificar)

24. Caixa físico completo (depende da lacuna 3).
25. Multi-empresa (lacuna 4).
26. Catraca/acesso físico (hardware).
27. Contas a pagar aprimorado, plano de contas hierárquico, conciliação bancária, DFC, exportação contábil — dimensionar pela ambição contábil do produto.
28. Google for Education; secretaria digital.
29. Gerador de consultas — recomendação: fora da v1 (C10).

### P3 — Por segmento (ativar conforme decisão dos segmentos-alvo)

30. Motor de avaliação completo, diário de classe, frequência por aula (C5), conceitos, competências, boletim com regras de publicação, histórico escolar, promoção/finalização em lote, portais de aluno e professor, quadro de horários e alocação didática, grade curricular.

### Decisões em aberto para a etapa 2

> **Todas resolvidas em 2026-06-11 — ver §11.**

- **D1.** Segmentos-alvo do SIGE (destrava P3 e várias condicionais).
- **D2.** Recebimento presencial → caixa físico (P2.24).
- **D3.** Estrutura societária → multi-empresa (P2.25).
- **D4.** Postura jurídica sobre bloqueio de boletim por inadimplência (C7).
- **D5.** Adoção (e regras operacionais) do chat escola↔família.
- **D6.** Ambição do módulo contábil (P2.27).
- **D7.** SIGE mono-escola evoluível ou multi-escola desde o início (define se "Instituição" existe como nível).

---

## 11. Decisões validadas com o Fabio (2026-06-11) — vinculantes para o blueprint final

### 11.1 Escopo e plataforma

| # | Decisão | Resolução |
|---|---|---|
| D1 | Segmentos-alvo | **Da creche ao ensino médio.** Entram: motor de avaliação completo, diário de classe, boletim, histórico escolar, portais de aluno/professor, disciplinas/grade, Novo Ensino Médio/itinerários formativos, progressão parcial, quadro de horários e alocação didática |
| D7 | Multi-escola | **SaaS multi-escola desde o início.** Instituição→Unidade→Empresa; isolamento por tenant; configurabilidade profunda por escola; feature flags por tenant; impersonação de suporte auditada — recursos antes marcados "plataforma do fornecedor" voltam ao escopo como plataforma do SIGE |
| D3 | Multi-empresa | **Sim, completo** — múltiplos CNPJs por unidade, contas/caixa/NFS-e por empresa, aglutinação parametrizável |
| D2 | Caixa físico | **Sim, completo** — operador, dinheiro, cartão com operadoras e repasse D+n, PIX presencial, cheques e custódia, travas de desconto |
| D6 | Contábil | **Retaguarda completa** — contas a pagar, plano de contas hierárquico, centro de resultado, conciliação bancária, DFC, exportação contábil |
| D4/C7 | Impedimentos | **Tudo configurável pela escola** (modelo Activesoft), incluindo bloqueio de boletim. O blueprint deve prever avisos/disclaimers legais no produto (Lei 9.870/1999); a responsabilidade da configuração é da escola |
| D5 | Chat escola↔família | **Não entra na v1** — comunicação segue pelos canais estruturados (comunicado, circular, ocorrência, diário) |

### 11.2 Estratégias estruturais (conflitos C1–C10)

| Conflito | Resolução |
|---|---|
| C1 Estrutura acadêmica | **Configurável por escola**: cada tenant monta sua hierarquia acadêmica; o SIGE oferece templates prontos (infantil por agrupamento etário com permanência/horário; fundamental/médio seriado com disciplinas/grade) e a escola adapta. A semântica da base (derivação por idade, permanência, horário) é um dos templates, intocada |
| C2 Modelo financeiro | **Título + camadas**: Título é a entidade central; tipo de cota vira classificação configurável (seeds CMAE/CC/CRV); serviços com preço por vigência seguem como motor de preço; plano de pagamento como camada de oferta opcional |
| C3 Efetivação de matrícula | **Gatilho configurável por escola**: assinatura, pagamento da pré-matrícula, ou ambos. Requerimento permanece o mecanismo formal; reserva de vaga paga como estado intermediário opcional (compatível com CRV) |
| C4 Permissões da família | **Camadas compostas**: atribuições por parente×aluno (quem vê o quê) × situação do aluno (quando) × configuração da escola (quais recursos). O mais restritivo vence; **regra fixa: responsável legal sempre acessa os dados do próprio filho** |
| C5 Frequência | **A escola define o modo** (por turma/segmento): presença diária com entrada/saída e presença parcial, e/ou chamada por aula. Ambos disponíveis na plataforma, alimentando apuração única |
| C6 Estados do aluno | **Meta-modelo com seeds protegidos**: cadastro de situações configurável por escola, mapeado a domínios de sistema/acadêmico/Educacenso + flags; comportamentos finos da base como propriedades declarativas; conjunto seed de sistema não removível |
| C8 Assinatura eletrônica | **Conector abstrato com Autentique E Clicksign homologados já na v1** |
| C9 Push/app | Mantido o app próprio + OneSignal; ClassApp descartado (já decidido na análise) |
| C10 Gerador SQL | **Não entra na v1** (nem em versão restrita) |

### 11.3 Módulos e integrações — o que entra e o que fica fora da v1

**Entram na v1 (além de tudo da base e dos itens P0/P1):** secretaria digital; API pública de parceiros (OAuth2 + escopos + auditoria, sem nenhum parceiro integrado); PIX (QR dinâmico + boleto híbrido).

**Ficam FORA da v1:** biblioteca (e o badge BIB de impedimento sai junto); filantropia/CEBAS; ranking de alunos; catraca/controle de acesso físico; Google for Education; gerador de consultas; conector de garantidora; chat escola↔família; BI embarcado (v1 atende com KPIs nativos + central de relatórios); registro online de boleto por API bancária (BB/CAIXA/Itaú) e gateways — cobrança bancária na v1 = **CNAB Santander/Sicoob + PIX**, sobre arquitetura de adaptadores que permita acrescentar bancos depois.

### 11.4 Critérios de garantia adicionais (validados em 2026-06-11)

| # | Critério | Resolução |
|---|---|---|
| G1 | Cobertura integral da base | **Garantido**: toda capacidade do `bp_sige_atual` entra na v1 (MANTER literal ou preservada como seed/default de mecanismo generalizado). Únicas exclusões: os defeitos que o próprio §12 da base condena. O BP final terá **seção de rastreabilidade da base** — tabela mapeando cada item do `bp_sige_atual` para a seção do novo blueprint onde vive |
| G2 | Administração do SaaS | **v1 inclui backoffice operacional essencial**: onboarding de escolas (tenant, unidades, empresas, usuário inicial, templates de estrutura acadêmica), feature flags por tenant, impersonação de suporte auditada, monitoramento de filas/rotinas por tenant, suspensão/desativação de escola. FORA da v1: billing do SaaS e tiers/planos comerciais (gestão manual) |
| G3 | API do App iOS/Android (desenvolvido em Flutter/Dart) | A API atual **não tem documentação formal** — vive em `config/routes.php` + controllers do CakePHP (`C:\Users\Fabio\Documents\sige`): geração legada `/api/*` (~75 endpoints, incl. toda a rotina de creche) + geração atual `/api/v2/*` (~30 endpoints), consumidas simultaneamente pelo app. Decisão: **contrato novo unificado no BP final + inventário auditável dos ~105 endpoints atuais + tabela de rastreabilidade antigo→novo, com virada coordenada do app** (sem camada de compatibilidade). Confirmado no código o lixo do §12.10 (`AlsunosAPIController`, `*BKP`, `TokenControllera` etc.) — sem destino no contrato novo |
| G4 | Arquitetura/linguagem/banco | Não existem especificados em lugar nenhum hoje (os BPs são deliberadamente independentes de tecnologia; o SSD é a etapa própria). Stack do legado confirmado no código: PHP ≥7.2 + CakePHP 4.0.4 + MySQL 8.0 + Docker + mPDF + App iOS/Android (Flutter/Dart). Decisão: **o BP final inclui um capítulo de diretrizes tecnológicas** — recomendações não-vinculantes de linguagem, banco e arquitetura de referência, além dos requisitos que constrangem o SSD (multi-tenant, filas, OAuth2, adaptadores). A decisão vinculante de stack permanece no SSD |
| G5 | Integração Santander | **Coberta**: confirmado no código que a integração atual é remessa/retorno CNAB 240 (arquivos `.rem`, layouts Santander 033 e Sicoob gerados campo a campo em `FinanceiroController` + baixa via `BaixaBoletosCommand`). Na v1 = MANTER, evoluindo para o cadastro de "agente de cobrança" com Santander/Sicoob como seeds. Registro online via API do banco continua fora da v1 (o atual também não tem) |
| G6 | NFS-e via hub fiscal | **Focus NFe escolhida** como hub de emissão (API REST/JSON, DPS/NFS-e Nacional, dedup por `ref`, multi-CNPJ, painel para o contador). Alternativas avaliadas: NFE.io (2º — API equivalente, painel fraco), PlugNotas e Nuvem Fiscal (plano B, não aprofundadas). Desenho na v1: **conector fiscal abstrato com Focus como adaptador default, substituindo a integração direta com o webservice da prefeitura** — a direta não é reconstruída (é mecanismo, não funcionalidade: RPS/simulação/controle mensal/vínculo a alunos permanecem; assinatura XML, mTLS, certificado e layout municipal passam a ser problema do hub). Alinhado à NFS-e Nacional da reforma tributária (IBS/CBS) |
| G7 | Regra de Cobrança como conceito de 1ª classe | O conceito **não existe nomeado em nenhuma das fontes** (na base a geração é processo implícito no código; no Activesoft está fatiado em régua de comunicação, cálculo de multa/juros, plano de pagamento e automação de matrícula). Decisão: o BP final formaliza **'Regra de Cobrança'** como entidade configurável por escola — origem (serviço contratado, plano, matrícula, avulso) × alvo (responsável financeiro) × gatilho (mensal, matrícula, renovação, manual) × valor (vigência/plano) × cota × recorrência × vencimento. O comportamento da base (geração mensal por serviços contratados) vira a regra seed. Régua de comunicação, multa/juros e descontos seguem como cadastros próprios referenciados pela regra |
| G8 | White-label por escola | A interface de gestão (backoffice do SaaS + configurações da escola) inclui personalização mínima de marca por tenant: **upload de logomarca, favicon e paleta de cores** (aplicada via tokens do design system — RQ-TOK-02) e **subdomínio próprio por escola** (tenant resolvido por subdomínio, nunca por path — pilar herdado da análise). Web administrativa e portais usam a marca da escola. **App iOS/Android: app único nas lojas com tema dinâmico por tenant** (logo, cores e splash carregados ao identificar a escola no login) — sem ícone/nome personalizado na loja (builds white-label dedicados ficam fora do produto) |
| G9 | Gestão do desenvolvimento no Linear.app | Backlog, issues e todo o fluxo de desenvolvimento serão geridos via **Linear.app**. O BP final sai acompanhado do **desdobramento estruturado para importação no Linear**: módulos → Projects, capacidades → Issues (descrição + critérios de aceite derivados do BP + prioridade P0–P3 da análise), marcos da v1 → Milestones. A criação efetiva no Linear acontece quando a integração (MCP/API) estiver conectada — o BP nasce rastreável até a issue |
| G10 | Interface: shadcn/ui + shell de 5 áreas | A interface web será construída sobre os componentes do **shadcn/ui** (https://ui.shadcn.com/) como biblioteca-base do design system — atendendo os requisitos de stack única de UI/tokens (RQ-CSS/RQ-TOK herdados da análise). A **shell do SIGE** segue estrutura persistente de 5 áreas: (1) **NavSidebar** full-height colapsável (290px↔80px), com navegação própria por persona (secretaria/direção, professor, responsável, aluno, admin do SaaS); (2) **AppHeader** fixo 64px — botão de colapsar menu, **seletor de perfil/visão** (alternância entre perfis do usuário), breadcrumb da área, busca global, alternador claro/escuro, notificações e perfil; (3) **ContextBar** condicional ~50px — filtros de contexto, seletor de período/ano letivo, ações e tabs locais da página; (4) **painel lateral direito colapsável** (290px↔80px) — área reservada na v1 para o assistente de IA da v2 (G11); (5) **MainContent** fluido, subdividido em 1–3 colunas auto-ajustáveis. As larguras se redistribuem automaticamente ao colapsar as laterais (base de referência: viewport 1600px). Implicações: ecossistema React (alimenta as diretrizes do G4) e **dark mode completo desde o início** |
| G11 | Inteligência transversal (agentes) — mapeado para v2 | A v2 introduz uma **camada transversal de inteligência** que atravessa todos os módulos e opera em segundo plano — não é um módulo isolado. Ver detalhamento em §11.5. A v1 não a implementa, mas o desenho (filas, eventos, API, painel direito da shell — G10) não pode criar obstáculo |

### 11.5 Visão v2 — Camada transversal de inteligência (SIGE Intelligence)

> Mapeada agora para orientar o desenho da v1; implementação na v2. Nome de trabalho: **SIGE Intelligence**.

**Conceito.** Camada de inteligência composta por **agentes especializados por domínio**, em duas classes com naturezas distintas:

**Classe 1 — Agentes determinísticos (CRON): sem LLM, sempre ativos, custo zero.** Rodam em background, calculam/monitoram/alertam com base em regras — não podem ser desligados pois alimentam dados críticos. **As ~20 rotinas agendadas da base (§9 do BP atual) são a semente desta classe**, reorganizadas por domínio:

| Agente (v2) | Absorve da base (§9) | Evolução |
|---|---|---|
| Agente de Cobrança | Baixa de boletos, notificações de vencimento | Executa a Regra de Cobrança (G7) e a régua multicanal |
| Agente de Conciliação | Conciliação movimentos×boletos, prova real | + conciliação bancária e PIX |
| Agente de Bolsas e Descontos | Vencimento de bolsas, verificação de descontos | + workflow de solicitação |
| Agente Fiscal | Envio/consulta/verificação de RPS, controle mensal de notas | Opera via hub Focus NFe (G6) |
| Agente de Matrícula | Renovações, requerimentos, cancelamentos | + gatilhos configuráveis (C3), promoção em lote |
| Agente de Contratos | Verificação de assinaturas, testemunhas | + SLA de assinatura com alertas e cancelamento por prazo |
| Agente de Presença | Fechamento de presenças, comunicado de presença | + apuração nos dois modos de frequência (C5) |
| Agente de Ocupação | — (novo) | Monitora vagas×matrículas por turma, alerta lotação/ociosidade |
| Agente de Funil | Relatórios de apoio (interações) | Monitora interações atrasadas, leads parados, indicações |

**Classe 2 — Agentes LLM: opcionais, custo por uso, ativados por quem governa cada domínio.** Candidatos para o domínio escolar (lista de partida; cada um redige/sugere — nunca executa sozinho):

| Agente | Função | Ativação |
|---|---|---|
| Agente de Relatório Pedagógico | Auxilia o professor a redigir avaliações descritivas a partir dos registros do diário e da rotina | Escola |
| Agente de Comunicação | Redige comunicados, circulares e respostas para aprovação da secretaria/coordenação | Escola |
| Agente de Cobrança Assistida | Redige mensagens de cobrança personalizadas (tom, histórico, acordo) para aprovação do financeiro | Escola |
| Agente de Insights de Gestão | Narra KPIs (inadimplência, ocupação, evasão, funil) em linguagem natural para a direção | Escola |
| Agente de Retenção | Cruza sinais de evasão (frequência caindo, inadimplência, ocorrências) e propõe plano de retenção | Escola |
| Agente de Captação | Qualifica prospects, sugere próxima interação e redige follow-ups para o comercial aprovar | Escola |
| Agente de Onboarding | Conduz a configuração inicial de uma escola nova de forma conversacional | Operador do SaaS |
| Agente de Suporte | Analisa tickets das escolas e sugere respostas com base na base de conhecimento | Operador do SaaS |

**Governança de motorização (3 níveis):**
1. **Operador do SaaS** — define os modelos disponíveis via **OpenRouter**; liga/desliga globalmente os agentes de plataforma (onboarding, suporte).
2. **Escola (tenant)** — insere seu **token OpenRouter próprio**, escolhe o modelo de IA e liga/desliga cada agente LLM do seu domínio — **cada escola decide como conectar seu modelo e arca com seu custo**.
3. **Usuário final** — preferências individuais onde aplicável.

**Evolução futura (v3+):** outros provedores além do OpenRouter e **modelos locais** (self-hosted), mantendo a mesma camada de administração de LLMs como abstração.

**Princípios invioláveis:**
- A inteligência **nunca age sozinha**: toda comunicação a famílias e todo ato administrativo passam por aprovação humana (encaixa no fluxo de aprovação que a base já tem para comunicados/fotos).
- A escola é sempre a autoridade final.
- **A camada é invisível para as famílias** — pais e alunos nunca interagem com a IA; tudo chega como mensagem da escola, após aprovação.
- Toda ação de agente é auditada (autor "agente X", aprovador humano, data) e logada na caixa de saída quando gera comunicação.

**Interface:** o painel lateral direito da shell (G10, área 4) é a casa do SIGE Intelligence — sempre presente, expansível/recolhível, com nome e avatar personalizáveis por escola; recolhido, exibe indicador discreto de atividade.

**Implicações para a v1 (anti-obstáculos):** rotinas como serviços nomeados por domínio (não scripts soltos); eventos de domínio publicáveis (matrícula efetivada, boleto vencido, presença fechada…) para os agentes assinarem na v2; painel direito reservado na shell; modelo de aprovação humana já generalizado (aproveitando o fluxo de aprovação existente); auditoria com autoria não-humana prevista no modelo de log.
