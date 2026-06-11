# Inventário da API atual — App iOS/Android (extraído do código legado)

> Fonte: `config/routes.php` + controllers do projeto CakePHP 4 legado. Gerado por engenharia reversa em 2026-06-11. Este inventário alimenta o doc 06 (contrato novo) e o doc 09 (rastreabilidade) do BP do novo SIGE. Nenhum destes endpoints será replicado literalmente — eles mapeiam para o contrato unificado novo.

## 1. Autenticação e validação de token

O `CheckTokenMiddleware` (`src/Middleware/CheckTokenMiddleware.php`) é aplicado aos dois scopes (`/api` e `/api/v2`). O token é lido do header HTTP `Authorization` (valor cru, sem prefixo `Bearer`) e validado por busca direta na tabela `login` (`WHERE api_token = <header>`). O token em si é um pseudo-JWT HS256 gerado no login (chave fixa `aldeiasecret`), mas a assinatura e a expiração **nunca são verificadas** — o claim `exp` é gravado com o timestamp do momento da emissão (já "expirado" ao nascer) e o que vale é apenas a igualdade da string persistida no banco. Cada novo login sobrescreve `api_token`, derrubando sessões anteriores do mesmo usuário.

Rotas isentas (públicas): no scope legado, qualquer path que **comece com** `token`, `recuperar-senha`, `circulares/`, `financeiro/boleto/`, `login/recuperar_senha`, `login/checar_codigo/`, `login/atualizar_senha/`, `alunos/foto-jpg` ou `verificar-versao` (dígitos são removidos do path antes da comparação); no scope v2: `autenticar`, `auth/apple-for-android`, `auth/apple-sign-in`, `auth/google-sign-in`, `auth/facebook-sign-in`, `recuperar-senha/email`, `recuperar-senha/sms`, `verificar-codigo`, `redefinir-senha`. Isso torna públicos, por efeito do prefixo, **todos** os endpoints `/api/circulares/*` (inclusive marcação de leitura) e o download de fotos por ID (`/api/alunos/foto-jpg/{id}`).

Em caso de header ausente, a resposta é `403 {"mensagem": "missing Authorization Header", "code": 1}`; token não encontrado no banco, `403 {"mensagem": "Invalid Token", "code": 2}`. Ponto crítico: o middleware chama `$handler->handle($request)` **antes** de validar o token — a action do controller é executada integralmente (incluindo escritas no banco) e só a resposta é descartada/substituída pelo 403. Não há autorização por recurso no middleware; cada action revalida (ou não) se o `id` enviado no corpo pertence ao usuário do token — em vários endpoints legados o `id` da pessoa vem no body e não é cruzado com o token.

## 2. Geração atual — /api/v2 (36 endpoints)

### Autenticação e conta

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/v2/autenticar` | POST | Público | Autentica por e-mail ou CPF + senha e emite `api_token` (aceita senha-mestre fixa no código) | body: `username`, `password` | JSON `{userData: {token, id, person_id, roles[], name, avatar, email, accessType, parentShortcuts, employeeShortcuts}, message}`; HTTP 400 em falha |
| `/api/v2/auth/apple-for-android` | POST | Público | Redireciona o callback do Sign in with Apple para o intent do app Android | body: payload do callback da Apple | HTTP 307 redirect `intent://callback?...` |
| `/api/v2/auth/apple-sign-in` | POST | Público | Troca o `code` da Apple por token (client_secret ES256 gerado com `key.p8`) e autentica o usuário | body: `code`, `useBundleId`, `username` | Mesmo envelope de `/autenticar`; HTTP 500 com corpo da Apple em falha |
| `/api/v2/auth/facebook-sign-in` | POST | Público | Autentica usuário já validado pelo Facebook (sem checagem de senha) | body: `username` | Mesmo envelope de `/autenticar` |
| `/api/v2/auth/google-sign-in` | POST | Público | Autentica usuário já validado pelo Google (sem checagem de senha) | body: `username` | Mesmo envelope de `/autenticar` |
| `/api/v2/validar-identidade` | GET | Ambos | Verifica se o token do header corresponde a um login válido | header `Authorization` | `{message: "Identidade válida!"}`; 403 se inválido |
| `/api/v2/permissoes` | POST | Ambos | Calcula as atribuições (módulos liberados) da pessoa por papel — códigos `0..7` e `dados` | body: `id` (pessoa), `role` | Array JSON simples de atribuições (ex.: `["0","1","3","dados"]`) |
| `/api/v2/login/atribuir-atalhos-parente` | POST | Parente | Salva os atalhos personalizados do parente no login | body: `atalhos` (JSON string) | `{message, shortcuts}` |
| `/api/v2/recuperar-senha/email` | POST | Público | Gera código de 6 dígitos (validade 1h) e envia por e-mail | body: `email` | `{status: true, id}` (id da pessoa); 404/500 em erro |
| `/api/v2/recuperar-senha/sms` | POST | Público | Gera código de 6 dígitos e envia por SMS (gateway smsdev, chave fixa no código) | body: `phone` | `{status: true, id}`; 404/500 em erro |
| `/api/v2/verificar-codigo` | POST | Público | Confere o código de recuperação informado | body: `contactInfo`, `type` (`email`/`sms`), `code` | `{message}`; 400/404 em erro |
| `/api/v2/redefinir-senha` | POST | Público | Redefine a senha (mín. 6 caracteres), inativa o código e já autentica | body: `contactInfo`, `type`, `code`, `password`, `repeatPassword` | `{userData, message}` |
| `/api/v2/permissoes/{role}` | GET | Ambos | Mesma lógica de permissões, mas identifica a pessoa pelo token (não pelo body) | rota: `role` | `{permissoes: [...]}` |

### Alunos e dados cadastrais

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/v2/alunos/alunos-diario` | GET | Parente | Lista alunos do parente com atribuição de diário (`4`) ou dos quais é responsável | header `Authorization` | `{alunos: [{id, personId, name, avatarUri, unity, scholarityGroup}]}` |
| `/api/v2/alunos/alunos-circulares` | GET | Parente | Lista alunos com atribuição de circulares (`2`) | idem | idem |
| `/api/v2/alunos/alunos-financeiro` | GET | Parente | Lista alunos com atribuição de financeiro (`3`) | idem | idem |
| `/api/v2/alunos/alunos-dados` | GET | Parente | Lista alunos com atribuição `2`, `3` ou `4` (acesso a dados cadastrais) | idem | idem |
| `/api/v2/alunos/alunos-fotos` | GET | Parente | Lista alunos com atribuição de fotos (`7`) | idem | idem |
| `/api/v2/alunos/dados-pessoais/{id}` | GET | Parente | Dados pessoais do aluno (sexo, nascimento, naturalidade, matrícula) — valida vínculo parente-aluno | rota: `id` (aluno) | `{atendimento: {sex, dateOfBirth, placeOfBirth, nationality, inscriptionNumber, inscriptionDate, startingDate, accessCode, sigeCode}}` (`accessCode`/`sigeCode` fixos em `000000`/`000`) |
| `/api/v2/alunos/dados-atendimento/{id}` | GET | Parente | Dados de atendimento do aluno (unidade, curso, turno, agrupamento, permanência, nível, horário) | rota: `id` | `{atendimento: {studentId, year, school, course, turn, scholarityGroup, group, permanency, level, schedule}}` |
| `/api/v2/alunos/parentes/{id}` | GET | Parente | Lista os parentes vinculados ao aluno, com parentesco e contato | rota: `id` | `{parentes: [{id, personId, name, avatarUri, kinship, email, phone, address}]}` |
| `/api/v2/alunos/inserir-avatar/{id}` | POST | Parente | Salva avatar (base64) na pessoa do aluno | rota: `id`; body: `image64`, `fileName` | `{success: true}`; 400/500 em erro |

### Circulares

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/v2/circulares/ultima-circular` | GET | Parente | Retorna a circular mais recente da pessoa do token | header `Authorization` | `{circular: {id, bundleId, studentId, type, date, title, fileUri, htmlContent, readDate}}` |
| `/api/v2/circulares/por-aluno/{id}` | GET | Parente | Lista circulares do aluno para a pessoa do token (valida vínculo e matrícula ativa) | rota: `id` | `{circulares: [...mesma estrutura...]}` |

### Financeiro/boletos

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/v2/financeiro/boletos/{id}` | GET | Parente | Lista até 30 boletos da pessoa do aluno (não excluídos, valor > 0), com tipo descritivo | rota: `id` (aluno) | `{boletos: [{id, studentId, typeDescription, value, paymentDate, paidValue, fileUri, dueDate, createdAt}]}` |
| `/api/v2/financeiro/pdf-boleto/{id}` | GET | Parente | Gera o PDF do boleto via scraping interno (`/boleto/imprimir.php` + mPDF) | rota: `id` (boleto) | Binário `application/pdf` |
| `/api/v2/financeiro/linha-digitavel/{id}` | GET | Parente | Extrai a linha digitável do HTML do boleto | rota: `id` (boleto) | `{linha: "<dígitos>"}` |

### Fotos

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/v2/fotos/por-aluno/{id}` | GET | Parente | Lista fotos aprovadas (dupla aprovação) em que o aluno aparece | rota: `id`; body: `numero` (limite) [VERIFICAR: usa `getData` em GET] | `{fotos: [{id, text, fileUri, fileTitle, mentioned[], mime, hasRestriction, date}]}` |
| `/api/v2/fotos/inserir-log` | POST | Parente | Registra log de compartilhamento de foto | body: `id` (foto), `student` | `{sucesso, mensagem}` |

### Turmas

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/v2/turmas/turmas-frequencia` | GET | Colaborador | Lista turmas do colaborador (admins funções 1–3 veem por unidade de acesso) com alunos matriculados | header `Authorization` | `{turmas: [{id, unitId, unit, serviceId, service, name, startingTime, endingTime, avatarUri}]}` |
| `/api/v2/turmas/alunos/{id}` | GET | Colaborador | Lista alunos da turma (valida que o colaborador está atribuído nela) | rota: `id` (turma) | `{alunos: [{id, personId, name, avatarUri, unity, scholarityGroup}]}` |

### Rotina diária (chamada)

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/v2/diarios/alunos-frequencia/{id}` | POST | Colaborador | Lista alunos da turma com o registro de presença do dia informado (cria o diário do dia se não existir) | rota: `id` (turma); body: `data` | `{alunos: [{id, personId, name, avatarUri, unity, scholarityGroup, presence: {id, diaryId, classId, presence, entryTime, lateTime, exitTime}}]}` |
| `/api/v2/diarios/registrar-ausencia` | POST | Colaborador | Registra falta do aluno (somente para hoje) | body: `student`, `class`, `date` | `{presence: {...}}`; 400 se data ≠ hoje |
| `/api/v2/diarios/registrar-presenca` | POST | Colaborador | Registra entrada do aluno (somente hoje; valida horário ≥ início da turma) | body: `student`, `class`, `date`, `time` | `{presence: {...}}`; 400 em validação |
| `/api/v2/diarios/registrar-saida` | POST | Colaborador | Registra horário de saída numa presença existente (somente hoje) | body: `id` (presença), `time` | `{presence: {...}}` |
| `/api/v2/diarios/excluir-presenca` | POST | Colaborador | Exclui um registro de presença (somente do dia de hoje) | body: `id` (presença) | `[]` |

## 3. Geração legada — /api (99 endpoints)

### Autenticação e conta

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/token` | POST | Público | Autentica por e-mail/CPF + senha, gera `api_token` e devolve credenciais (com lista de alunos se parente) | body: `username`, `password` | `{credentials: {token, id, id_pessoa, role, name, avatar, email, alunos|turmas}, message}`; HTTP 400 em falha |
| `/api/validar-autenticacao` | POST | Ambos | Confirma validade do token (a recusa acontece no middleware) | header `Authorization` | `{success: true}` |
| `/api/login/recuperar_senha` | POST | Público | Gera código de recuperação (6 dígitos, 1h) e envia por e-mail | body: `email` | `{status, id}` ou `{status: false, message}` |
| `/api/login/recuperar_senha_sms` | POST | Público | Idem, por SMS (varre todos os logins ativos comparando telefones) | body: `telefone` | `{status, id}` ou `{status: false, message}` |
| `/api/login/checar_codigo/{id}` | POST | Público | Valida o código de recuperação da pessoa e o inativa | rota: `id` (pessoa); body: `codigo` | `{status, message}` |
| `/api/login/atualizar_senha/{id}` | POST | Público | Define nova senha após recuperação (mín. 6 caracteres) | rota: `id` (pessoa); body: `password`, `confirm_password` | `{status, message}` |
| `/api/login/trocar_senha/{id}` | POST | Ambos | Troca de senha autenticada (confere senha atual) | rota: `id` (login); body: `password`, `confirm_password`, `new_password`, `confirm_new_password` | `{status, message}` |
| `/api/permissoes` | POST | Ambos | Versão antiga do cálculo de atribuições (parente: união simples; colaborador: vazio) | body: `id`, `role` | Array JSON de atribuições |
| `/api/permissoes-novas` | POST | Ambos | Cálculo atual de atribuições por papel, com renovação (`5`), fotos (`6`/`7`), tipo de login restrito e exclusão de matrículas canceladas | body: `id`, `role` | Array JSON de atribuições (ex.: `["0".."5","dados"]`) |
| `/api/usuario-troca-perfil` | POST | Ambos | Verifica se a pessoa é parente E colaborador e devolve o papel alternativo | body: `id`, `role` | `{troca_perfil: bool, nova_role}` |
| `/api/login/confirmar-cpf` | POST | Ambos | Confere se o CPF informado bate com o da pessoa | body: `id`, `cpf` | `{status, confirmado, cpf}` |
| `/api/alunos/token` | GET | Ambos | Ping autenticado (sem lógica) | — | `{sucesso: true}` |

### Notificações

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/login/notificacoes-usuario` | POST | Ambos | Lista notificações do sino com metadata por módulo (Circulares, Financeiro, Comunicados, Fotos, Custom) | body: `id` (login), `role` | `{status, notificacoes: [{id, titulo, texto, lida, modulo, horario, metadata}]}` |
| `/api/login/marcar-notificacoes` | POST | Ambos | Marca todas as notificações do usuário como lidas e removidas | body: `id` | `[]` |
| `/api/login/contar-notificacoes` | POST | Ambos | Conta notificações não lidas por tipo de usuário (Parente=3, Colaborador=2) | body: `id`, `role` | `{status, quantidade}` |
| `/api/login/ler-notificacao` | POST | Ambos | Marca uma notificação como lida | body: `id` (notificação) | `[]` |

### Alunos e dados cadastrais

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/alunos/listar-por-parente` | POST | Parente | Lista alunos vinculados ao parente (exclui matrículas canceladas) | body: `id` (pessoa), `primeiro` (opcional), `responsavel` (opcional) | `{status, mensagem, alunos: [{id, nome, avatar}]}` |
| `/api/alunos/listar-por-colaborador` | POST | Colaborador | Lista alunos das turmas do colaborador (admins 1–3: por unidades de acesso) | body: `id` | `{status, mensagem, alunos: [{id, nome, avatar}]}` |
| `/api/alunos/listar-por-turma` | POST | Colaborador | Lista alunos de uma turma, com flags de fralda/educação infantil | body: `id`, `turma` | `{status, mensagem, alunos: [{id, nome, avatar, fraldas, troca_fraldas, educacao_infantil}]}` |
| `/api/alunos/dados-aluno` | POST | Parente | Dados cadastrais dos alunos do parente (sem atendimento) | body: `id` | `{status, mensagem, alunos: [{nome, id, avatar, pessoa_id, sexo_formatado, data_nascimento_formatada, naturalidade, matricula, data_matricula_formatada, data_inicio_formatada, atendimento: false}]}` |
| `/api/alunos/dados-atendimento` | POST | Parente | Idem, incluindo bloco de atendimento (curso, turno, permanência, agrupamento, nível, horário, unidade, turmas) | body: `id` | Mesmo envelope com `atendimento: {...}` |
| `/api/alunos/parentes` | POST | Parente | Parentes de cada aluno do parente, com parentesco, contatos e endereço | body: `id` | `{status, mensagem, alunos: [{..., parentes: [{nome, parentesco, email, telefones, endereco_formatado, avatar}]}]}` |
| `/api/alunos/verificar-responsavel` | POST | Parente | Indica se o parente é o responsável financeiro do aluno | body: `id`, `aluno` | `{responsavel: bool}` |
| `/api/alunos/inserir-avatar` | POST | Parente | Salva avatar (base64) na pessoa do aluno | body: `id` (aluno), `arquivo` (base64), `titulo` | `{status, mensagem, avatar}` |
| `/api/alunos/listar-para-diario` | POST | Parente | Lista alunos com atribuição de diário e indica se já estão "em aula hoje" (libera edição do diário pelo parente) | body: `id`, `data`, `primeiro`/`responsavel` (opcionais) | `{status, mensagem, alunos: [{id, nome, avatar, em_aula_hoje, horarios: {entrada, saida}}], liberado}` |

### Rotina diária (chamada/refeições/sono/medicação/fralda/saída)

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/alunos/inserir-diario-colaborador` | POST | Colaborador | Grava o diário completo do aluno em lote (medicações ministradas, refeições, trocas de fralda, repousos, comunicados, exclusões); funções ≥4 só hoje e com aluno em aula | body: `id`, `aluno`, `data`, `turma`, `medicacoes[]`, `refeicoes[]`, `trocas_fralda[]`, `repousos[]`, `comunicados[]`, `excluidos{repousos[], trocas[]}` | `{status: true}` ou `{status: false, mensagem}` |
| `/api/alunos/inserir-diario-parente` | POST | Parente | Grava mensagens e medicações do diário enviadas pelo parente (bloqueia fins de semana) | body: `usuario`, `aluno`, `data`, `mensagens[]` (id, conteudo, destinatarios), `medicacoes[]` (nome, dosagem, medida, descricao, horarios[], arquivo-receita, data_repeticao) | `{status, mensagem}` |
| `/api/alunos/inserir-refeicoes` | POST | Colaborador | Insere/atualiza consumo de uma refeição no diário (somente hoje) | body: `id`, `aluno`, `data`, `turma`, `refeicao`, `consumo` | `{status, mensagem}` |
| `/api/alunos/excluir-refeicoes` | POST | Colaborador | Exclui o registro de refeição do diário de hoje | body: `aluno`, `data`, `refeicao` | `{status, mensagem}` |
| `/api/alunos/inserir-repouso` | POST | Colaborador | Insere/edita repouso (sono) validando conflito de horários e o horário contratado do aluno (somente hoje) | body: `id`, `aluno`, `data`, `inicio`, `final`, `id_edicao` (opcional) | `{status, mensagem}` |
| `/api/alunos/excluir-repouso` | POST | Colaborador | Exclui repouso do diário de hoje | body: `data`, `id_edicao` | `{status, mensagem}` |
| `/api/alunos/inserir-chamada` | POST | Colaborador | Insere/edita presença/falta do aluno na turma (somente hoje; validação de horário de entrada desativada no código) | body: `id`, `aluno`, `data`, `turma`, `presenca`, `horario`/`entrada` | `{status, mensagem}` |
| `/api/alunos/inserir-saida` | POST | Colaborador | Registra saída numa presença; bloqueia se houver comunicado pendente de aprovação | body: `presenca`, `saida` | `{status, mensagem}` |
| `/api/alunos/excluir-chamada` | POST | Colaborador | Exclui presença do diário de hoje | body: `data`, `id_edicao` | `{status, mensagem}` |
| `/api/alunos/inserir-medicacao` | POST | Ambos | Insere/edita medicação para hoje ou o próximo dia útil; bloqueia com aluno em aula; aceita receita em base64 e repetição | body: `id`, `aluno`, `data`, `nome`, `dosagem`, `medida`, `descricao`, `horarios[]`, `arquivo_receita{arquivo, formato}`, `usuario`, `data_repeticao` | `{status, mensagem, nova: {...medicação...}}` |
| `/api/alunos/excluir-medicacao` | POST | Ambos | Exclui uma medicação | body: `id` | `{status, mensagem}` |
| `/api/alunos/listar-medicacoes-por-data` | POST | Ambos | Lista medicações do aluno na data, com flag `em_aula` | body: `aluno`, `data` | `{medicacoes: [{id, aluno, nome, dosagem, medida, arquivo_receita, observacao, horarios, ministrado, data}], em_aula}` |
| `/api/alunos/possui-medicacoes-anteriores` | POST | Ambos | Verifica se há medicação nos últimos 5 dias (para "repetir medicação") | body: `aluno`, `data` | `{status, possui, data_anterior, ja_repetida}` |
| `/api/alunos/ministrar-medicacao` | POST | Colaborador | Marca um horário de medicação como ministrado pela pessoa | body: `id` (medicação), `horario`, `pessoa` | `{status, mensagem, nova: {...}}` |
| `/api/alunos/remover-medicacao` | POST | Colaborador | Desfaz a marcação de ministrado de um horário | body: `id`, `horario`, `pessoa` | `{status, mensagem, nova: {...}}` |
| `/api/alunos/inserir-troca-fralda` | POST | Colaborador | Insere/edita troca de fralda (somente hoje) | body: `id`, `aluno`, `data`, `turma`, `tipo`, `horario`, `troca` (opcional p/ edição) | `{status, mensagem, id, tipo}` |
| `/api/alunos/editar-horario-troca-fralda` | POST | Colaborador | Edita horário da troca (apenas o colaborador que cadastrou) | body: `id`, `data`, `troca`, `horario` | `{status, mensagem, id}` |
| `/api/alunos/excluir-troca-fralda` | POST | Colaborador | Exclui troca de fralda (apenas quem cadastrou; somente hoje) | body: `id`, `data`, `troca` | `{status, mensagem, id}` |
| `/api/alunos/diario-completo` | POST | Parente | Retorna o diário consolidado do aluno no dia (medicações, refeições, repousos, trocas, comunicados), ocultando itens parciais com diário aberto | body: `aluno`, `data` | `{status, diario: {aberto, fraldas, tipo_fralda, medicacoes[], refeicoes[], repousos[], trocas_fralda[], comunicados_parente[], comunicados_colaborador[]}}` |
| `/api/alunos/refeicoes-por-aluno` | POST | Colaborador | Tipos de refeição do aluno com o consumo registrado na data | body: `aluno`, `data` | `{status, mensagem, tipos: [{id, nome, refeicao_servida, servida}]}` |
| `/api/alunos/repousos-por-aluno` | POST | Colaborador | Repousos do aluno na data | body: `aluno`, `data` | `{status, mensagem, repousos: [{id, horarios, colaborador, inicio, final}]}` |
| `/api/alunos/evacuacao-por-aluno` | POST | Colaborador | Trocas de fralda/evacuações do aluno na data | body: `aluno`, `data` | `{status, mensagem, trocas: [{id, horario, colaborador, tipo}]}` |
| `/api/turmas-por-refeicao` | POST | Colaborador | Lista outras turmas do colaborador com alunos que fazem a refeição informada | body: `id`, `turma`, `refeicao` | `{status, mensagem, turmas: [{id, nome, avatar}]}` |
| `/api/turmas/refeicoes` | POST | Colaborador | Tipos de refeição da turma na data, com status de conclusão | body: `id` (turma), `data` | `{status, mensagem, refeicoes: [{id, nome, concluido}]}` |
| `/api/turmas/alunos-por-refeicao` | POST | Colaborador | Alunos da turma que fazem a refeição, com consumo do dia (filtra presentes se data = hoje) | body: `turma`, `refeicao`, `data` | `{status, mensagem, alunos: [{id, nome, avatar, refeicao_diario}]}` |
| `/api/turmas/alunos-sono` | POST | Colaborador | Alunos presentes da turma com os repousos do dia | body: `turma`, `data` | `{status, mensagem, alunos: [{id, nome, avatar, horario, repousos_diario[]}]}` |
| `/api/turmas/alunos-chamada` | POST | Colaborador | Alunos da turma com o registro de chamada do dia e flag `em_aula` em outra turma | body: `turma_id`, `data` | `{status, mensagem, alunos: [{id, nome, avatar, em_aula, chamada_diario}]}` |
| `/api/turmas/alunos-evacuacao` | POST | Colaborador | Alunos com troca de fralda da turma e os registros do dia | body: `turma`, `data` | `{status, mensagem, alunos: [{id, nome, avatar, evacuacao_diario[], vezes, fralda}]}` |
| `/api/turmas/medicacoes-por-data` | POST | Colaborador | Medicações de todos os alunos da turma na data, com flags `em_aula` e `medicacao_atrasada` | body: `turma`, `data` | `{status, mensagem, alunos: [{id, nome, avatar, medicacoes[], em_aula, medicacao_atrasada}]}` |

### Comunicados do diário

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/alunos/comunicados` | POST | Ambos | Lista comunicados do diário do aluno no dia, filtrados por papel (parente vê os de parente; colaborador vê aprovados ou próprios) | body: `id`, `aluno`, `data`, `funcao_remetente` | `{success, message, messages: [{id, remetente{}, editavel, aluno, mensagem, destinatarios[], horario, data_extenso}]}` |
| `/api/alunos/inserir-comunicado` | POST | Ambos | Cria/edita comunicado para hoje ou amanhã; status inicial depende da função (coordenação aprova direto, professores ficam pendentes); gera notificação de sino | body: `remetente`, `aluno`, `data`, `conteudo`, `destinatarios[]`, `funcao_remetente`, `id` (edição) | `{success, message}` |
| `/api/alunos/excluir-comunicado` | POST | Ambos | Exclui um comunicado | body: `id` | `{status, mensagem}` |
| `/api/alunos/destinatarios-por-aluno` | POST | Ambos | Lista possíveis destinatários: parente vê professores + coordenação; colaborador vê parentes com atribuição de diário | body: `id`, `aluno_id`, `role` | `{status, mensagem, destinatarios: [{id, nome, avatar, funcao|parentesco}]}` |
| `/api/alunos/comunicados-por-usuario` | POST | Ambos | Histórico de comunicados enviados/recebidos do usuário (com filtros por data, aluno e status; marca recebidos como lidos) | body: `id`, `status_comunicados`, `data`, `aluno_id`, `is_admin` (opcionais) | `{status, mensagem, comunicados: [{id, remetente{}, horario, data_extenso, destinatarios[], data, enviada, recebida, conteudo, aluno{}}]}` |
| `/api/alunos/info-comunicados` | POST | Ambos | Resumo por aluno: última mensagem do dia e contagem de não lidas | body: `id`, `alunos_id[]`, `data` | `{status, mensagem, info_comunicados: {<aluno_id>: {horario_ultima_mensagem, conteudo_ultima_mensagem, quantidade_nao_lidas}}}` |
| `/api/alunos/trocar-status-comunicados` | POST | Colaborador | Aprova/arquiva comunicado (status 0–3); aprovação plena exige função 1–3 | body: `admin`, `comunicado`, `status` | `{status, mensagem}` |
| `/api/alunos/alunos-comunicado-geral` | POST | Ambos | Lista alunos para a tela de comunicados gerais, com última mensagem e não lidas; colaborador tem busca por termo e paginação | body: `id`, `role`, `termo`, `offset`, `limit` | `{status, alunos: [{id, nome, avatar, horario_ultima_mensagem, conteudo_ultima_mensagem, quantidade_nao_lidas}], turmas}` |
| `/api/parentes/destinatarios` | POST | Parente | Destinatários (colaboradores das turmas do aluno) para o parente enviar comunicado | body: `id` (pessoa), `aluno` | `{success, destinatarios: [{id, funcao, nome, avatar}], destinatarios_obrigatorios: []}` |

### Circulares

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/circulares/{id}` | GET | Público | Converte circular HTML em PDF (mPDF) e serve o binário | rota: `id` (circular) | Binário `application/pdf`; 404/403 |
| `/api/circulares/data_leitura` | POST | Público | Marca a circular como lida (se ainda não) e retorna a data de leitura por extenso | body: `id` (circular) | `text/plain` "Lido às HH:mm de dd de Mês de aaaa" |
| `/api/circulares/dados-circular/{id}` | POST | Público | Dados de uma circular, validando que pertence à pessoa informada no body | rota: `id`; body: `id` (pessoa) | `{valid, id, date, fullDate, title, read, type, content}` |
| `/api/alunos/circulares` | POST | Parente | Circulares por aluno do parente, agrupadas por "Mês de Ano" | body: `id` | `{status, mensagem, alunos: [{nome, id, pessoa_id, avatar, circulares: {"Mês de Ano": [{id, data, data_completa, titulo, lido, tipo, conteudo}]}}]}` |
| `/api/turmas/buscar-circulares` | POST | Colaborador | Lotes de circulares da turma com percentual de leitura, agrupados por mês | body: `id` (pessoa), `turma` | `{status, mensagem, circulares: {"Mês de Ano": [{id, data, data_completa, titulo, tipo, conteudo, numero_alunos, percentual_leitura}]}}` — [VERIFICAR] o link `conteudo` aponta para a action `pdfCircularLote`, que não existe em `CircularesAPIController` (link quebrado para lotes HTML) |

### Fotos

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/alunos/inserir-foto` | POST | Colaborador | Cria/edita foto do módulo de fotos (imagem base64, menções, status inicial por função) | body: `autor`, `id` (edição), `imagem{conteudo, formato}`, `imagem_alterada`, `mencoes{separadas{alunos[], colaboradores[]}}`, `texto`, `data` | `{sucesso, mensagem}` |
| `/api/alunos/fotos-por-usuario` | POST | Colaborador | Fotos aprovadas do autor (ou da unidade, se admin) na data | body: `autor`, `admin`, `data` | `{sucesso, fotos: [{id, texto, arquivo, mencionados[], primeira_aprovacao, segunda_aprovacao, restritos[]}]}` |
| `/api/alunos/fotos-pendentes` | POST | Colaborador | Fotos com status pendente (1) do autor/unidade | body: `autor`, `admin` | idem |
| `/api/alunos/fotos-lixeira` | POST | Colaborador | Fotos excluídas (status 2) do autor/unidade | body: `autor`, `admin` | idem |
| `/api/alunos/alterar-status-foto` | POST | Colaborador | Aprova (exige dupla aprovação por usuários distintos, identificados pelo token) ou rejeita foto | body: `id` (foto), `status` | `{sucesso, trocou, atualizou, template, mensagem}` |
| `/api/alunos/listar-com-fotos` | POST | Parente | Alunos do parente (responsável ou atribuição 7) com suas fotos aprovadas | body: `id`, `numero` (limite) | `{status, alunos: [{id, pessoa, nome, avatar, fotos: [{id, texto, arquivo, titulo_arquivo, mencionados[], mime, possui_restricao, data{}}]}]}` |
| `/api/alunos/carregar-mais-fotos` | POST | Parente | Paginação de fotos do aluno (offset/limit) | body: `id` (pessoa do aluno), `numero`, `carregadas` | `{status, fotos: [...]}` |
| `/api/alunos/inserir-log-compartilhamento` | POST | Ambos | Grava log de compartilhamento de foto | body: campos da entidade (`foto`, `pessoa`, `aluno_aba`) | `{sucesso, mensagem}` |
| `/api/alunos/foto-jpg/{id}` | GET | Público | Converte a foto para JPG (Imagick, 1000px, qualidade 75) e serve o binário — sem token | rota: `id` (aceita `foto_<id>.jpg`) | Binário `image/jpg` |
| `/api/usuarios-mencoes/{id}` | GET | Colaborador | Pessoas mencionáveis (alunos ativos e colaboradores das unidades de acesso) | rota: `id` (pessoa do colaborador) | `{sucesso, usuarios: [{id, nome, avatar, tipo}]}` |

### Financeiro/boletos

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/alunos/boletos` | POST | Parente | Boletos por aluno (atribuição 3): pendentes ou pagos no último ano, com contagem | body: `id` | `{status, mensagem, alunos: [{nome, id, avatar, pessoa_id, quantidade_boletos, boletos: [{id, data_vencimento, valor, valor_pago, data_pagamento, status, tipo}]}]}` — atenção: `status` é sobrescrito com o tipo de login do parente (int) |
| `/api/financeiro/boleto/{id}` | GET | Público | Gera o PDF do boleto via scraping interno + mPDF — sem token | rota: `id` (boleto) | Binário `application/pdf` |
| `/api/financeiro/linha/{id}` | GET | Parente | Extrai a linha digitável do boleto | rota: `id` | `{linha}` |

### Renovação de matrícula

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/alunos/parcelas-crv` | POST | Parente | Simula as parcelas da Cota de Reserva de Vaga: estima o próximo atendimento do aluno, busca o valor do próximo ano e monta 1–4 parcelas com vencimentos do ano letivo (desconto de 10% até a data limite) | body: `aluno` | `{status, parcelas: [[["valor", "dia", "mês", "ano"], ...], ...]}` |
| `/api/alunos/renovar-matricula` | POST | Parente | Cria requerimento de renovação (tipo 7) e gera os boletos de CRV conforme as parcelas escolhidas | body: `aluno`, `parcelas`, `modalidade`, `id` (requerente) | `{status, mensagem}` — [VERIFICAR] a checagem de duplicidade usa `a_partir_de = '2026-01-01'` fixo no código |
| `/api/alunos/nao-renovar-matricula` | POST | Parente | Cria requerimento de não renovação com motivo | body: `aluno`, `motivo` | `{status, mensagem}` |
| `/api/alunos/verificar-renovacao` | POST | Parente | Estado da renovação do aluno: `devendo` (boletos vencidos), `renovada`, `nao-renovada` ou `false` | body: `aluno` | `{status, existente}` |

### Turmas

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/turmas` | POST | Colaborador | Turmas do colaborador no ano letivo corrente (admins por unidade), só as com alunos matriculados | body: `id` (pessoa) | `{status, mensagem, turmas: [{id, nome ("Unidade - Serviço - Turma"), avatar, horario_entrada, ano}]}` |

### Calendário

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/alunos/feriados` | GET | Ambos | Retorna a propriedade `feriados` do controller — **lista vazia fixa** (endpoint inócuo, superado por `dias-letivos`) | — | `{sucesso, feriados: []}` |
| `/api/alunos/dias-letivos` | POST | Ambos | Lista "feriados" do ano corrente: feriados cadastrados + todos os fins de semana (com 2 exceções fixas no código) | — | `{sucesso, feriados: ["mm-dd", ...]}` |
| `/api/alunos/dias-letivos-atualizados` | POST | Ambos | Feriados e dias letivos do ano corrente, filtrados pelos cursos da turma ou dos alunos informados | body: `cursos[]` ou `turma` ou `alunos[]` | `{sucesso, feriados: ["aaaa-mm-dd"], diasLetivos: ["aaaa-mm-dd"]}` |
| `/api/alunos/proximas-aulas` | POST | Ambos | Próximo dia letivo (módulo de dias letivos + feriados) por aluno a partir de uma data | body: `alunos[]`, `atual` (opcional) | `{proximos_dias: {<aluno_id>: "aaaa-mm-dd"}}` |

### Parentes/perfil

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/parentes/configuracoes` | POST | Parente | Resumo do perfil (e-mail, endereço — do parente ou do aluno —, telefone) | body: `id` (pessoa) | `{status, mensagem, parente: {email, endereco, telefone}}` |
| `/api/parentes/dados_pessoais` | POST | Parente | Dados pessoais completos + lista de estados para os selects | body: `id` | `{status, mensagem, parente: {nome, sexo, cpf, rg, orgao_expeditor, data_nascimento, naturalidade, nacionalidade, email, email_secundario, telefones[], endereco{}, empresa, ocupacao}, opcoes: {estados}}` |
| `/api/parentes/inserir_dados_pessoais` | POST | Parente | Atualiza dados pessoais e endereço (validação `atualizarPorMobileApp`; `mesmo_endereco` apaga endereços próprios) | body: `id`, `nome`, `sexo`, `cpf`, `rg`, `orgao`, `nacionalidade`, `naturalidade`, `email`, `email_secundario`, `data_nascimento`, `empresa`, `ocupacao`, `mesmo_endereco`, `endereco{...estado_endereco}` | `{status, mensagem, campo}` (campo com erro de validação) |
| `/api/parentes/inserir_avatar` | POST | Parente | Salva avatar do parente (base64) | body: `id`, `arquivo`, `titulo` | `{status, mensagem, avatar}` |

### Colaboradores/perfil

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/colaboradores/configuracoes` | POST | Colaborador | Resumo do perfil (função, e-mail, endereço, telefone) | body: `id` (pessoa) | `{status, mensagem, colaborador: {funcao, email, endereco, telefone}}` |
| `/api/colaboradores/dados_pessoais` | POST | Colaborador | Dados pessoais completos + estados | body: `id` | `{status, mensagem, colaborador: {...}, opcoes: {estados}}` |
| `/api/colaboradores/inserir_dados_pessoais` | POST | Colaborador | Atualiza dados pessoais e endereço do colaborador | body: `id`, `nome`, `sexo`, `cpf`, `rg`, `orgao`, `nacionalidade`, `naturalidade`, `email`, `email_secundario`, `data_nascimento`, `endereco{}` | `{status, mensagem, campo}` |

### Infraestrutura (versão e avatar)

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/api/verificar-versao` | POST | Público | Compara a versão do app com a mínima fixada no código (Android 3.1.2 / iOS 5.1.2) | body: `platform`, `version` | `{valid: bool}` |
| `/api/pessoas/exibir_avatar/{id}` | GET | Ambos | Serve o arquivo de avatar da pessoa (jpg/png/gif) | rota: `id` (pessoa) | Binário de imagem |

## 4. Rotas especiais (fora dos scopes de API)

| Endpoint | Verbo | Consumidor | O que faz | Entradas principais | Saída |
|---|---|---|---|---|---|
| `/autentique-webhook` | POST | Público (webhook) | Recebe eventos de assinatura do Autentique; quando todas as partes assinam, localiza o requerimento por `autentique_id` (status 3) e finaliza o fluxo: para requerimentos tipo 0/1 (mudança de serviço/horário) calcula a diferença de valores e gera boleto complementar se necessário; para tipo 3 (matrícula) gera a cota de composição | body: `partes[]` (funcao, assinado), `documento.uuid` | Sem corpo estruturado (processamento + logs) |
| `/.well-known/assetlinks.json` | GET | Público | Serve o JSON de deep links do Android (`webroot/deep-links-json/android.json`) | — | JSON estático |
| `/.well-known/apple-app-site-association` | GET | Público | Serve o JSON de universal links do iOS (`ios.json`) | — | JSON estático |

## 5. Observações estruturais

- **Envelopes inconsistentes.** O legado usa majoritariamente `{status: bool, mensagem, <payload>}` em PT-BR, mas convive com `{sucesso, ...}`, `{success, message, messages}`, arrays JSON crus (`/api/permissoes*`), `text/plain` (`circulares/data_leitura`) e binários (PDF, JPG). O v2 padroniza chaves em inglês/camelCase voltadas ao Dart (`{alunos: [...]}`, `{userData, message}`, `{presence}`) e sinaliza erro por código HTTP + exceções (400/403/404/500), enquanto o legado quase sempre devolve HTTP 200 com `status: false`.
- **`status` sobrecarregado.** Em `/api/alunos/boletos`, o campo `status` do envelope é sobrescrito com o inteiro `statusLoginParente` (0/1/2), misturando semânticas no mesmo campo.
- **POST para leitura.** Praticamente toda a geração legada usa POST para consultas (listagens, dados cadastrais, circulares, diário), passando o `id` da pessoa no body — sem cruzamento com o token. O v2 migra leituras para GET com identidade derivada do header, mas ainda tem POST de leitura (`/api/v2/diarios/alunos-frequencia/{id}`) e leitura de body em GET (`/api/v2/fotos/por-aluno/{id}` lê `numero` via `getData`).
- **Middleware valida depois de executar.** `CheckTokenMiddleware` roda a action antes de checar o token; requisições sem token válido produzem efeitos colaterais (gravações) mesmo recebendo 403.
- **Endpoints públicos sensíveis.** Por prefixo de isenção, são públicos: PDF de qualquer circular por ID sequencial, marcação de leitura de circular, PDF de boleto por ID (`/api/financeiro/boleto/{id}`) e qualquer foto do módulo de fotos (`/api/alunos/foto-jpg/{id}`).
- **Duplicações /api ↔ /api/v2 (mesma capacidade nas duas gerações):**
  - Login: `/api/token` ↔ `/api/v2/autenticar`;
  - Validação de token: `/api/validar-autenticacao` ↔ `/api/v2/validar-identidade`;
  - Recuperação de senha: `/api/login/recuperar_senha`, `recuperar_senha_sms`, `checar_codigo/{id}`, `atualizar_senha/{id}` ↔ `/api/v2/recuperar-senha/email`, `recuperar-senha/sms`, `verificar-codigo`, `redefinir-senha`;
  - Permissões: `/api/permissoes-novas` ↔ `/api/v2/permissoes` e `/api/v2/permissoes/{role}` (três implementações quase idênticas da mesma regra);
  - Listagem de alunos do parente: `/api/alunos/listar-por-parente` ↔ `/api/v2/alunos/alunos-diario|circulares|financeiro|dados|fotos`;
  - Dados cadastrais do aluno: `/api/alunos/dados-aluno`, `dados-atendimento`, `parentes` ↔ `/api/v2/alunos/dados-pessoais/{id}`, `dados-atendimento/{id}`, `parentes/{id}`;
  - Avatar do aluno: `/api/alunos/inserir-avatar` ↔ `/api/v2/alunos/inserir-avatar/{id}`;
  - Circulares por aluno: `/api/alunos/circulares` ↔ `/api/v2/circulares/por-aluno/{id}`;
  - Boletos: `/api/alunos/boletos` + `/api/financeiro/boleto|linha/{id}` ↔ `/api/v2/financeiro/boletos|pdf-boleto|linha-digitavel/{id}`;
  - Fotos do aluno: `/api/alunos/listar-com-fotos` + `carregar-mais-fotos` ↔ `/api/v2/fotos/por-aluno/{id}`; log de compartilhamento: `inserir-log-compartilhamento` ↔ `fotos/inserir-log`;
  - Turmas/chamada do colaborador: `/api/turmas`, `/api/alunos/listar-por-turma`, `inserir-chamada`, `inserir-saida`, `excluir-chamada`, `/api/turmas/alunos-chamada` ↔ `/api/v2/turmas/turmas-frequencia`, `turmas/alunos/{id}`, `diarios/*`.
- **Regras de negócio embutidas relevantes:** registros de rotina diária só podem ser inseridos/excluídos no dia corrente (chamada, refeição, repouso, fralda); medicação aceita hoje ou o próximo dia útil e bloqueia com aluno em aula; comunicado de professor nasce "pendente" e exige aprovação da coordenação (e bloqueia o registro de saída do aluno enquanto pendente); foto exige dupla aprovação por usuários distintos; renovação só aparece dentro da janela `inicio_renovacao`–`final_renovacao` do ano letivo; CRV em até 4 parcelas com 10% de desconto até a data limite; boletos v2 limitados a 30; paginação só em fotos (`numero`/`carregadas`) e em alunos do comunicado geral (`offset`/`limit`).
- **Valores fixados no código (riscos de migração):** ano letivo travado em `2024` no `ApiV2Controller::anoLetivoCorrente()` e em `2026` nos controllers legados (`AlunosAPI`, `TurmasAPI`); checagem de duplicidade de renovação com data `2026-01-01` literal; senha-mestre global e chave JWT fixas; chave do gateway de SMS na URL; lista fixa de ~70 Ids de pessoas autorizadas ao módulo de fotos (`PESSOAS_FOTOS`) e bypass de renovação para um `pessoa_id` específico; e-mails de cópia de suporte fixos nos envios; geração de PDF de boleto por scraping de `/boleto/imprimir.php` via cURL.
- **Rotas órfãs:** nenhuma — todas as 135 actions roteadas existem nos controllers. Há, porém, uma referência interna quebrada: `TurmasAPIController::buscarCirculares` monta URL para `CircularesAPI::pdfCircularLote`, método inexistente (links de lote HTML quebrados).

## 6. Código morto (sem destino no contrato novo)

- `AlsunosAPIController.php` — cópia antiga do AlunosAPI com erro de digitação no nome; sem nenhuma rota.
- `AlunosAPIControllerBKP.php` — backup manual do AlunosAPI; sem rota.
- `CircularesApiController(1).php` — duplicata de download (sufixo `(1)`); nome de classe nem corresponde a controller roteável.
- `TokenControllera.php` — cópia do TokenController com sufixo acidental; sem rota.
- `fFinanceiroController.php` — cópia com prefixo acidental; sem rota.
- `FinanceiroControllerBKP.php` — backup manual do Financeiro; sem rota.
- `_AlunosController.php` — versão desativada por prefixo `_`; sem rota.
- `_CircularesController.php` — versão desativada por prefixo `_`; sem rota.
- `EnderecoApiController.php` — verificado: nenhuma rota em `routes.php` referencia `EnderecoApi`; morto.
- `PessoaApiController.php` — verificado: nenhuma rota referencia `PessoaApi` (a rota de avatar usa `PessoasAPI`); morto.

## 7. Totais

| Métrica | Contagem |
|---|---|
| Endpoints `/api/v2` | 36 |
| Endpoints `/api` (legado) | 99 |
| **Total nos dois scopes** | **135** |
| Rotas especiais fora dos scopes | 3 (webhook Autentique + 2 deep links) |
| Públicos (isentos de token) | 20 (11 no legado + 9 no v2) + as 3 rotas especiais |
| Consumidor Parente | 40 (17 v2 + 23 legado) |
| Consumidor Colaborador | 44 (7 v2 + 37 legado) |
| Consumidor Ambos | 31 (3 v2 + 28 legado) |
| Rotas órfãs | 0 (1 referência interna quebrada: `pdfCircularLote`) |
| Arquivos de código morto | 10 |
