# 01 — Domínio Funcional

> Mapa módulo a módulo do domínio, extraído da spec Janus do ActiveSoft SIGA
> (instância siga04 / tenant CEMP — Grupo Arco Educação).
> **Fontes:** `_janus_spec/_sidebar.md`, `_janus_spec/ui/SDD.md` (318 telas, 44 entidades, 436 endpoints),
> `_janus_spec/ui/screens/*.md`, `_janus_spec/ui/comunicacao-familia.md`, `_janus_spec/produto-estrategico.md`,
> `_janus_spec/api/*` (40 endpoints públicos + 931 release notes) e DOMs capturados em `.janus/captures/`.

**Visão geral:** ERP escolar multi-tenant (instâncias numeradas `sigaNN`, banco por cliente — `dbSigaCEMP`), multi-unidade dentro do tenant, cobrindo o ciclo completo: captação → matrícula online → contrato/assinatura eletrônica → enturmação → diário/notas/boletim → cobrança/boleto/PIX → comunicação família → compliance (Educacenso, NFS-e, LGPD). SPA React sobre API Django (`/api/v1/` privada da UI; `/api/v0/` pública para parceiros via Bearer token por instituição). Produto tem **tiers comerciais** (release note cita "clientes Light e Basic") e **feature flags por instituição** (ex.: "Integração ReCB apenas para o Colégio CPI").

---

## 1. Módulo Alunos

**Propósito:** cadastro mestre do aluno e hub 360º — dados pessoais, saúde, vínculos com turmas, ocorrências, histórico escolar, descontos e cobranças, tudo em abas de um mesmo detalhe.

**Telas/funcionalidades:**
- `alunos` — listagem com busca, paginação e filtro por status (ativo/inativo)
- `alunos/<id>/detalhe` — cadastro completo do aluno (abas: Cadastro, Ficha médica, Turmas, Ocorrências, Histórico, Descontos, Cobranças)
- `alunos/<id>/ficha_medica` — dados de saúde (tipo sanguíneo, SUS, médico, hospital de emergência, alergias, restrições alimentares, doenças, medicação)
- `alunos/<id>/enturmacao` — matrículas/vínculos do aluno em turmas, com "Nova matrícula" e exibição de turmas inativas
- `alunos/<id>/observacoes` — ocorrências/observações por aluno
- `alunos/<id>/historico` — histórico escolar por anos cursados, com "Configurar" e "Imprimir"
- `alunos/<id>/desconto_autorizado` — descontos concedidos por período/turma
- `alunos/<id>/cobranca` — títulos financeiros do aluno (mesma engine de Contas a Receber, escopada ao aluno)
- `alunosporturma` — relatório/consulta de alunos por turma (resumo de turmas, enturmações, alunos por disciplina, reordenação)
- `aluno_segunda_via` — emissão de 2ª via (boleto/documento)
- `consulta_aluno` — consulta rápida
- `ocorrencias_alunos` — lista global de ocorrências com registro e **operações em lote** (inclui envio de mensagens em lote)
- `ranking_de_alunos` — ranking dos melhores alunos por período/curso/série/turma/fase de nota

**Entidades:** Aluno, AlunoTurma (enturmação), FichaMedica, Observação/Ocorrência, Histórico, Desconto, Título, Responsável (vínculo), CampoDinamico.

**Regras de negócio observadas:**
- Cadastro distingue **Nome Social vs Nome Civil** (toggle "Possui diferença entre Nome Social e Nome Civil?").
- Campos: sexo, cor/raça, estado civil, CPF/RG (com órgão emissor/UF/data), celular, e-mail, flag **Falecido**, dados censitários (deficiências p/ Educacenso), Google for Education, **campos dinâmicos** (extensão de schema por escola), **gestão de presença** (catraca).
- **Filiação 1 / Filiação 2** com grau de parentesco + designação separada de **Responsável financeiro** e **Responsável pedagógico** (papéis distintos sobre o mesmo aluno).
- Flags de alerta no cadastro (parametrizáveis no Financeiro): **OBS** (observação com impedimento), **FNC** (inadimplente), **COB** (título em cobrança registrada em aberto), **BIB** (pendência na biblioteca) — a listagem traz `status_obs`, `status_fnc`, `status_cob`, `status_bib`, `status_pdm` (procedimentos de matrícula pendentes).
- **Situação do aluno na turma** é entidade configurável (21 registros no tenant): Lista de espera, Seleção, Sem Situação, Fichamento, Pré-matrícula online, Pré-Matrícula, Cursando (Ativo), Evadido, Transferido, Cancelado, Aprovado, Reprovado. Cada situação mapeia para: **situação sistema** (Ativo/Inativo), **situação acadêmica**, **situação Educacenso** e flag **digitação de nota** (Permitido/Não permitido).
- **Motivos de inativação** configuráveis (5 no tenant): Cancelado, Desistência, Matrícula Suspensa, Transferido, Troca de turma.
- Release note: ao mudar a situação do aluno para ATIVO, a matrícula muda automaticamente de **TEMPORÁRIA para NORMAL**.
- Desconto concedido é por **aluno × turma × período de vigência** (ex.: "2º FILHO, de 01/01/2023 até 31/12/2023, 10,00%").
- Matrícula numérica gerada por **sequenciais nomeados** (ver Configurações).
- Endereço é entidade compartilhada: alterar endereço propaga para as pessoas vinculadas (aviso explícito na UI).
- Ocorrências têm **tipo, situação e data de liberação** (controle de quando ficam visíveis ao responsável).

**Configurabilidade:** situações de aluno, motivos de inativação, campos dinâmicos, tipos de ocorrência, avisos OBS/FNC/COB/BIB lig/desl, parentescos.

---

## 2. Módulo Responsáveis

**Propósito:** cadastro do responsável (pessoa física ou jurídica — campo "CPF ou CNPJ"), vínculos com alunos, papel financeiro e acesso ao portal/app.

**Telas:** `responsaveis/<id>/detalhe`, `responsaveis/<id>/alunos_vinculados`, `responsaveis/<id>/observacoes`, `controle_de_acesso/portal/resps`, `responsavel_financeiro`, `responsavel_verifica_situacao`.

**Entidades:** Responsável, VínculoAlunoResponsável, Observação, Profissão (cadastro auxiliar), Religião (cadastro auxiliar).

**Regras de negócio observadas:**
- Dados: login/senha próprios, profissão, local/telefone de trabalho, falecido, forma de entrega de documento, flag **Funcionário**, indicador de inscrição estadual (não contribuinte/contribuinte — para NFS-e), mãe/pai do responsável.
- **Dados financeiros por responsável:** forma de recebimento preferida, **dia preferencial de vencimento**, texto livre adicionado às instruções do boleto.
- **Controle de acesso físico:** identificador de catraca, regra "Sempre permitir entrar e sair", impressão digital.
- **Mobile:** flag "Pode acessar o aplicativo".
- Acesso ao portal é **derivado, não direto**: a UI explica que o responsável só acessa o produto se houver aluno vinculado **ativo em turma cujo período e situação atendam aos requisitos de acesso** configurados (tela mostra passo a passo de correção). Endpoint público `acesso/vinculo_aluno_responsavel_liberado/`.
- Parentesco é cadastro configurável (17 registros, incluindo lixo de dados reais: "maãe", "mae" — o blueprint novo deve normalizar isso).

---

## 3. Módulo Colaboradores

**Propósito:** cadastro de funcionários/professores, formação, alocação didática e disponibilidade de horários.

**Telas:**
- `colaboradores` — listagem paginada
- `colaboradores/<id>/detalhe` — cadastro (dados pessoais, endereço, complementares, censitários, registro de nascimento, catraca, Google for Education)
- `colaboradores/<id>/escolaridade` — formação acadêmica
- `colaboradores/<id>/disciplinas` — disciplinas que o professor está habilitado a lecionar
- `colaboradores/<id>/disciplinas_alocadas` — turmas × disciplinas efetivamente alocadas
- `colaboradores/<id>/obs` — observações
- `colaboradores/<id>/quadro_horarios` — grade de disponibilidade
- `professores_turmas` — alocação de professor por turma (com filtro por período)

**Regras de negócio observadas:**
- **A alocação ativa o acesso:** banner literal — "Caso esta pessoa seja professor, aloque-o em uma turma na aba Turmas e disciplinas alocadas **para ativar seu acesso no produto**". Ou seja, o perfil professor no portal é derivado da alocação didática.
- Quadro de horários: matriz **slot × dia da semana** com slots nomeados M1–M8 (manhã), T1–T8 (tarde), N1–N8 (noite), por período letivo e turno; clique na célula alterna disponibilidade.
- Dados p/ compliance: identificação no INEP, flag "Considerar como **gestor escolar do EducaCenso**?", dados censitários de deficiência, matrícula funcional, datas de admissão/inativação, nº da pasta (arquivo físico).
- Catraca "GPA": identificador, senha de catraca, impressão digital.
- Cargo, apelido, login/senha, flag "Acesso ativo".

---

## 4. Estrutura Acadêmica (Períodos, Cursos, Séries, Turmas, Disciplinas, Grade)

**Propósito:** espinha dorsal da modelagem acadêmica que tudo referencia.

**Telas:** `periodo`/`anos_letivos`, `cursos`, `series`, `turmas` + `turmas/cadastrar`, `disciplina` (+detalhe/atualizar), `grade_curricular`, `itinerarios_formativos`, `tipo_horario`, `consulta_turma`.

**Entidades e hierarquia:** Período (ano letivo) → Curso → Série → Turma → Disciplina (via Grade Curricular); Aluno↔Turma (enturmação); Professor↔Turma↔Disciplina (alocação).

**Regras de negócio observadas:**
- **Período:** sigla + nome + data inicial/final + situação (tenant tem anos letivos de 2016 a 2025, 18 registros).
- **Curso:** nome, portaria de autorização, flag **Oficial** (cursos não-oficiais existem — ex.: extracurriculares; só turmas oficiais aparecem na API pública `lista_turmas`).
- **Série:** pertence a curso, tem **serviço financeiro padrão** vinculado (ex.: série "2º ANO" → serviço "MENSALIDADE - 2º ANO") e **próxima série** cadastrada (alimenta a Promoção entre séries).
- **Turma (cadastro):** nome, sigla/código, turno, período, curso/série, tipo da turma, flag "utiliza grade curricular", **quantidade máxima de alunos** (alimenta dashboard de ocupação), **coordenador responsável**, **serviço relativo à mensalidade**, código de agrupamento, datas início/fim, sala, tipo de horário, **código INEP**, situações do aluno na disciplina (Cursando/Aprovado/Reprovado), código para integração externa, flag "permitir integração com aplicativos de comunicação" (ClassApp), e **dados restringentes**: data de nascimento mín/máx, sexo permitido, "permitir acesso ao portal educacional", "exigir que o aluno já esteja vinculado em outra turma" (pré-requisito de matrícula — útil p/ turmas extracurriculares).
- Nomenclatura real de turma: "2º ANO - M1 - AMETISTA" (série + turno/slot + nome fantasia).
- **Disciplina:** sigla (ex.: "DC - 03"), tipo base (Normal / composta / etc.), nº de ordem no histórico, **agrupamento** (áreas de conhecimento BNCC), código MEC. Existem **disciplinas compostas** (com componentes "NC") e a pseudo-disciplina **"FREQUÊNCIA DIÁRIA"** (diário só de chamada, sem nota).
- **Grade curricular:** disciplina × série com ordem no boletim, "Requer Nota" (Sim/Não), tipo, carga horária exibida como "Hora/aula - semanal"; modos padrão/relatório, impressão com análise descritiva por série e conteúdo programático.
- **Tipo de horário:** cadastro nomeado por segmento/turno (ex.: "Ensino Fundamental II - Tipo A", "Educação Infantil - Turno Manhã") — define a malha de slots usada por turmas e quadro de horários.
- **Itinerários formativos** (Novo Ensino Médio) — rota dedicada + tabela de itinerário no boletim; constante `ACESSO_NOVO_ENSINO_MEDIO`.

---

## 5. Diários de Classe & Frequência

**Propósito:** diário pedagógico digital por turma × disciplina × fase de nota: aulas, conteúdo, tarefas, chamada e integração com a planilha de notas.

**Telas:** `diarios` (listagem com auditoria e criação em massa "CRIAR DIÁRIOS"), `diario/<id>` (abas: Alunos, Registro de aulas, Frequência, Notas e faltas), `diarios/frequencia_em_lote`, `diarios/configuracoes`, `conteudos`, `tarefas`, `relatorio_frequencia`.

**Regras de negócio observadas (tela de detalhe):**
- Diário = Período × Fase de nota × Curso × Série/Turma × Disciplina, com situação ("Disponível", "Concluído", "Confirmado").
- Parâmetros por diário: **mín e máx de aulas** (ex.: 1–100; 10–80 para Fund II), **limite para digitação de notas** (ex.: 30/09/2025), **prazo para criar aulas** (31/03/2025), **prazo para registrar conteúdo** (04/04/2025), "aulas devem estar no período da fase de nota?".
- Roster com matrícula, **data de entrada e saída do aluno no diário** e situação (suporta transferências no meio da fase).
- API pública permite **marcação de frequência por catraca/terceiros**: `identificar_marcacao/{chave_pessoa}`, `marcar_frequencia_aluno`, `listar_frequencia_aluno`, `diario_frequencia`.

**Configurações do diário (tela `diarios/configuracoes` — tudo parametrizável Sim/Não):**
- Registro de aulas: permitir aulas em datas passadas/futuras; **permitir criar aulas em feriados** (Não no tenant); copiar conteúdo entre disciplinas; **professor só lança aulas dentro do quadro de horário**; **criar aulas automaticamente a partir do quadro de horário** ao criar o diário (respeitando a quantidade máxima); exigir preenchimento de conteúdo e/ou tarefas pelo professor; comportamento quando há registros anteriores pendentes.
- Frequência: registrar em datas passadas/futuras; importar apuração de frequência (catraca); **chamada online para professores**; permitir frequência **"Dispensada"** e **"Falta justificada"**; registrar frequência em turma/disciplina alternativa; permitir registrar frequência e **reprocessar nota com fase já confirmada**; **como contabilizar "Falta justificada" para aprovação por frequência** (ex.: "Presença (Falta abonada)").
- Conclusão do diário: exigir (ou não) conteúdo digitado e tarefas digitadas.
- Exibição: renomear a funcionalidade no menu ("Diário de classe"), títulos customizados de conteúdo/tarefas, **situações de aluno que veem o diário** (ex.: "Cursando (Ativo)"), mensagem na lista de diários do professor.
- **Integração diário ↔ planilha de notas:** "Integração automática" + permissão na planilha quando o diário estiver concluído e confirmado ("Permite digitar e confirmar" / bloquear).

---

## 6. Avaliação, Notas & Boletim (núcleo pedagógico)

**Propósito:** motor configurável de avaliação: sistemas de avaliação por série, fases de nota, fórmulas, conceitos, competências, relatórios descritivos, boletim e regras de aprovação/reprovação.

**Telas:** `sistema_avaliacao` (+detalhe/CRUD), `definicoes_boletim_serie` (+detalhes por série), `formula_composicao`, `nota_conceito` (Conceitos), `avaliacoes`, `planilha_notas_faltas`, `alterar_notas_lote`, `reprocessamento_notas`, `fila_processamento`, `reprovacao_por_faltas` (em lote), `reprovar_aluno_recuperacao`, `avaliacao_relatorio`, `configuracoes_avaliacoes_competencia`, `planilha_avaliacao_competencia`, `boletim`, `boletim--regras-publicacao`, `fase_nota`, `portal_web_parametros/envio_mensagem` (parâmetros de digitação/confirmação de notas).

**Modelo central — Sistema de Avaliação** (observado: "Fund II", vinculado a 4 séries, com status Ativo/Desbloqueado e validação "Pendente" + ação "Validar o sistema"):
- Sistema = lista ordenada de **fases de nota** com "número de exibição". No tenant: 1º BIMESTRE → RECUPERAÇÃO 1º BIMESTRE → MÉDIA 1º BIMESTRE (×4 bimestres) → TOTAL ANUAL → MÉDIA FINAL → fase auxiliar "Precisa de" (nota necessária).
- Cada fase tem: nome, sigla (ex.: "1º BIM"), número de exibição, **tipo de fase** ("Informada por fórmula de composição", "Calculada", "Fórmula virtual" — estas duas últimas não podem sofrer alteração em lote), **valor máximo na fase**, composição de nota, "calcular disciplinas compostas", **dispensa de nota automática** (se a fórmula tiver uma única nota de composição, alunos que chegam à fase são dispensados automaticamente; com "tipo de dispensa" e "confirmar nota").
- **Datas e prazos por fase:** início–fim das aulas (ex.: 1º BIM = 13/01/2025–04/04/2025), **semanas e dias letivos** (12 semanas | 60 dias letivos), **limite para digitação de notas** (25/04/2025), flags de exibição no Portal Web.
- **Fórmulas de composição** com sintaxe própria sobre notas de composição: ex. observado `([CF01/01] [CF01/02]) >= 4.5` — fórmula nomeada "Fund II - 6º ao 9º Ano (1º, 2º e 3º Bimestre)".
- **Definições de aprovação/reprovação por fase:** *verificação por nota* (fórmula booleana; se aprovado → vai para fase X com situação Y; se reprovado → fase/situação Z) e *verificação por frequência* — texto literal da UI: "caso o aluno tenha obtido a nota para aprovação, mas não atingiu a fórmula de frequência, **será reprovado**. Será aprovado apenas quando atender aos dois critérios, nota e frequência", com "situação caso reprovado por frequência" própria. Ou seja: **aprovação = nota E frequência**, com roteamento de fase (recuperação) dirigido por fórmula.
- As definições do sistema podem ser **sobrescritas por série** em "Definições de boletim por série" (matriz Série × Sistema de avaliação × Modelo de boletim; ex.: 2º–4º ano → "Fund I (2º ao 4º Ano)", 5º ano → sistema próprio, 6º–9º → "Fund II"; modelo "Padrão").
- **Reprovação por faltas em lote** e **reprovar aluno em recuperação** são processamentos dedicados; também existe "alterar situação de aprovação/reprovação **por exceção** na enturmação" (release note) — override manual.

**Conceitos (`nota_conceito`):** escalas de conceito por série; regra explícita na UI: "Os conceitos devem ser criados usando **uma única unidade de medida (nota OU percentual)**. Ao criar o primeiro conceito em uma série, todos os demais devem seguir o mesmo padrão."

**Avaliações (`avaliacoes`):** instrumentos avaliativos por turma × disciplina × fase com **valor máximo** e totalizadores ("Total avaliações", "Total do valor máximo"); criação direto da planilha de notas; médias podem ser **calculadas por soma** quando tarefas/avaliações (release note); "cálculo por bloco" com permissão para professor alterar o bloco.

**Planilha de notas e faltas:** lançamento por Período × Curso × Série × Turma × Disciplina × Fase × Situação do aluno (default "Todas as situações Ativo"); colunas configuráveis NC01–NC10, Aulas, Faltas, Situação do aluno na turma; importação de padrão de avaliação; "aplicar nota na turma" (preencher coluna inteira); botão de **reprocessamento de notas**; sugestões nos inputs de digitação.

**Fluxo de confirmação de notas (parâmetros do Portal Web):** Messenger (fila assíncrona) para processamento de notas; **confirmar notas automaticamente** (opcional); professor pode confirmar/desconfirmar notas da turma pelo portal; professor pode **solicitar confirmação** mesmo sem todas as notas digitadas ou sem faltas/aulas dadas (parametrizável); **bloquear boletim com impedimento** (financeiro/documental); **permitir professor alterar fórmula** (sim/não). `fila_processamento` mostra a fila por turma/disciplina/aluno com log; `reprocessamento_notas` reprocessa cálculos.

**Cálculo (parâmetros globais):** quantidade de casas decimais; método **"Truncar"** (vs arredondar); texto a exibir quando dispensado, quando em 2ª chamada ("2CH"), texto no lugar da nota de fase; como a dispensa interfere na fase ("A nota de composição deve ser interpretada como zero"); tratamento da situação final de disciplinas compostas ("A disciplina componente deverá ser igual à disciplina composta").

**Modelo de boletim (configuração rica por sistema/série):** orientação de impressão (paisagem), origem do cabeçalho (unidade), dados do aluno (foto, nascimento, nacionalidade, filiação, endereço — cada um exibir/não), zero por extenso, notas de fase em negrito, aulas dadas e faltas por fase, total de faltas, situação na disciplina, separadores por fase, disciplinas dispensadas, médias das disciplinas componentes, textos personalizados (NCs dispensadas; disciplina que não requer nota), tabela de itinerário formativo, frequência em %, modo de agrupamento das disciplinas (BNCC), **cores para notas** com "média para definição de cores", tamanho da fonte, **ranking no boletim**, mensagens personalizadas da unidade e da série, observações do aluno na turma, **progressão parcial**, assinaturas configuráveis (Direção/Secretaria/Coordenação com imagem de assinatura), **protocolo de recebimento do boletim pelo responsável**. Há também `boletim--regras-publicacao` (regras de publicação/visibilidade do boletim no portal).

**Métodos avaliativos alternativos:**
- **Avaliação por relatório** (descritiva — educação infantil/Montessori): por turma/fase/disciplina, com confirmação/remoção pelo professor parametrizável.
- **Avaliação por competência:** estrutura própria — *Índices de aprendizagem*, *Tipos de competência*, *Fases de competência*, *Competências* (por disciplina × fase), planilha de lançamento com vínculo de alunos em lote e relatório configurável (foto, assinaturas professor/responsável, agrupar por tipo, título personalizado).
- **Avaliação institucional** (pesquisa de satisfação família→escola, com "minhas avaliações").

**Processamentos acadêmicos em lote:** `promocao_aluno_turma` (promoção entre séries com origem/destino, filtro de **alunos impedidos/não impedidos**, situação de destino), `finalizar_turmas` (irreversível — "Após o processamento, não será possível reverter", define nova situação dos alunos), `inativar_enturmacoes_lote` (ex.: limpar "Fichamento (Seleção)" com prazos vencidos), `alterar_notas_lote` (com etapa de **Simulação** antes do Processamento), `reprovacao_por_faltas` em lote.

**Histórico escolar (configuração própria):** regras de geração aplicadas automaticamente **ao concluir a turma**; cálculo da carga horária total (Semanal | Anual | Aulas dadas; ou "Semanas letivas × CH semanal" vs "Semanas letivas × CH-aula × aulas semanais"); incluir/excluir disciplinas dispensadas e inaptas (CH zerada para inaptos); **cursos que emitem histórico** (Infantil, Fundamental, Médio, Técnico); formatação (dados do aluno, escola, disciplinas, totais de rendimento por ano, dias letivos, faltas, média global, frequência); edição manual de notas no histórico (release note); códigos BNCC de agrupamento no histórico do EM.

---

## 7. Módulo Comunicação

**Propósito:** hub multicanal escola↔família — 18 canais mapeados (ver doc 04).

**Telas/funcionalidades:**
- `comunicados` + `comunicados/cadastrar` + detalhe/visualizar/listagem — broadcast por turma
- `mensagens` ("Conversas do App") — chat 1:1/grupo com responsáveis/alunos
- `caixa_saida` — **audit trail central de tudo que foi enviado**
- `central_novidades` — release notes do produto para o usuário
- `envio_de_senhas` — disparo em massa de e-mail com link de recuperação, por tipo de acesso (Alunos/Responsáveis/...)
- `mensagem_config_smtp` (configuração de remetente), `mobile_tipo_mensagem` (tipos de mensagem do app), `texto_personalizado` (templates HTML de e-mail com variáveis `[NOMEALUNO]`, `[CPFALUNO]`, `[MATRICULA]`, `[PARENTESCO_FILIACAO_1]`)
- Aviso de tela inicial (`/api/v1/aviso_tela_inicial/` — banner global, presente em todas as telas)

**Regras de negócio observadas:**
- **Comunicado:** título, toggle "Visível na área de Comunicados?", data de publicação, corpo rich-text (CKEditor com imagem, tabela, vídeo YouTube), **público-alvo = multi-select de turmas + filtro por situação do aluno na turma**. Dispara e-mail e aparece no portal/app.
- **Conversas do App:** filtros por unidade/período/curso/série/turma, tipo de mensagem, pessoa relacionada, "apenas não lidas" (default `min_mensagens_nao_lidas=1`); botão "Nova mensagem".
- **Caixa de saída:** cada envio registra inserção, tipo, destinatário, **tipo de destinatário (Responsável/Aluno)**, canal (Email/Celular/Login mobile), data de envio e **log de envio** ("Mensagem e-mail enviada para X") — operações em lote (reenvio). É a prova de entrega da escola.
- E-mail/SMS sem UI de disparo manual dedicada — são **subprodutos de eventos** (comunicado, cobrança, boletim, senha); campo `celular_para_envio_sms` no schema público de Aluno.
- Push: via **ClassApp** (integração com token, tags, sincronização forçada, log dedicado) e via **app próprio SIGA** ("API de Envio de Push para o APP SIGA").
- Ocorrências têm "envio de mensagens em lote" embutido.
- Variáveis de procedimentos de matrícula disponíveis no envio de mensagens (release note).

**Configurabilidade:** templates HTML por escola (`texto_personalizado` com cabeçalho/rodapé), remetente SMTP, tipos de mensagem mobile; provedores (SMTP/SMS) são geridos no nível do produto, não da escola.

---

## 8. Módulo Financeiro

**Propósito:** ciclo completo de receita escolar (títulos, boletos, PIX, negociação, régua de cobrança, caixa físico) + despesas, contabilidade e fiscal.

**Telas/funcionalidades:**
- **Receita:** `contas_receber` ("Títulos e operações"), `lancamentos_de_cobranca_em_aberto`, `baixa_pagamento`, `mensalidades`, `parcelas`, `recibos`, `aluno_segunda_via`
- **Cobrança bancária:** `cobranca_registrada` (lotes de remessa/retorno), `agente_cobranca`, `forma_recebimento`, `regua_cobranca_automatica`, `cobranca_ativa` (telecobrança), `calculo_multa_juros`, `titulos_pendencia_registro_boleto`, filas ISAAC (`titulos_isaac_fila_registro`, `cancelar_titulos_isaac_lote`, `operadores_isaac`, `vincular_servicos_produtos_plataforma_isaac`)
- **Caixa físico:** `caixa` (abertura/movimentação/fechamento por operador), `tipo_recebimento_caixa`, `cheques`, `cheque_custodia`, `recebimento_por_cartao` (+operadoras), `recebimentos_por_pix`
- **Despesa/contábil:** `contas_pagar`, `favorecidos`, `plano_contas`, `centro_resultado`, `historico_padrao`, `conciliacao_bancaria`, `fluxo_caixa_previsto`/`realizado` (DFC), `exportacao_contabil`, `relatorios_consolidacao_contabil`
- **Fiscal:** `nota_fiscal_servico` (NFS-e/RPS em lote), `notas_fiscais`
- **Comercial:** `descontos`, `plano_pagamento`, `situacao_contratos`, `contratos_financeiros`, `assinatura_eletronica`, `filantropia` (bolsas/ficha socioeconômica)
- **Relatórios:** faturamento, recebimento, cancelamentos, descontos, ticket médio, inadimplência (resumos por mês/serviço)

**Entidades:** Título, Parcela, Serviço, Desconto, PlanoPagamento, FormaRecebimento, AgenteCobrança, LoteCobrança, Caixa/Movimentação, Cheque, PlanoContas, CentroResultado, Despesa/Favorecido, NFS-e/RPS, Contrato, Empresa (multi-empresa dentro do tenant).

**Regras de negócio observadas (concretas):**
- **Título:** nº sequencial ("Gerar título a partir do número" configurável), serviço, competência, vencimento, valor documento vs valor a receber (com multa/juros), parcela "09/12", responsável pagador, situação (Em aberto/Liquidado/Cancelado), vínculo a turma, status de registro bancário ("Cobrança registrada — Registro confirmado"), pendências. Operações em lote: gerar taxa, **negociações**, imprimir boletos/carnê, gerar recibo, **declaração de imposto de renda**.
- **Multa e juros** é cadastro nomeado reutilizável: percentual de multa (ex.: **2,00%**), valor fixo de multa, percentual de juros (**1,00%**), valor diário de juros; existe "Sem Multa e Juros" (0%). Exposto também na API pública (`/api/v0/financeiro/calculo_multa_juros/`).
- **Descontos:** nome, classificação contábil (ex.: Comercial), tipo, **ordem de cálculo**, situação, **regra de concessão**: "Até o vencimento" (desconto condicionado à pontualidade) ou "Não condicionado (Nunca expira)". Exemplos reais: 2º FILHO (10%), 3º FILHO, BOLSISTA, DESC.DIVERSOS 10/15/20/25/30/65%. Release notes: termo de consentimento para **solicitação de desconto pelo Portal**, operação de apagar solicitações, manter descontos condicionais na negociação.
- **Planos de pagamento:** nome, período, serviço, ativo, **qtde de parcelas** e valores (ex.: "MENSALIDADE FUND I": 11 parcelas, R$ 26.158,00/ano; "MATRÍCULA FUND I EM 1/2/3 PARCELAS": R$ 2.853,00; Fund II R$ 2.950,00; "DIF DE MATRÍCULA" R$ 169/202). Modo de funcionamento "Simultâneo" (parâmetro).
- **Régua de cobrança automática:** lista de regras ("Adicionar regra"); release notes detalham: envio em lote imediato, **parâmetro mobile via ClassApp**, opção de **só enviar mensagem se o título estiver registrado**, aba com histórico de envios da régua no detalhe do título.
- **Agente de cobrança:** vincula forma de recebimento + **formato do arquivo de remessa (CNAB 400)** + flags "gerar impedimentos", "apenas títulos em aberto", "liquidar diretamente na escola" (4 agentes Santander no tenant).
- **Formas de recebimento:** tipos observados — "Direto" (caixa da escola), "Boleto bancário" (Santander), "**Boleto integrado / Híbrido - Itaú API**" (registro online); credenciamento BB API e CAIXA API; PIX imediato (`ACESSO_PIX_IMEDIATO`), bolecode Itaú (`ACESSO_BOLECODE_ITAU`).
- **Tipos de recebimento do caixa:** código + nome (CAC Cartão de Crédito, CAD Débito, CHQ Cheque, CHP Cheque Pré-Datado, DEP Depósito, DIN Dinheiro, MAN Liquidação Manual, PIX), flags vinculado a cartão / a transferência-PIX, **qtde dias de repasse** e intervalo (D+n por operadora).
- **Parâmetros financeiros (tela riquíssima — régua fina do comportamento):**
  - **Pagamento a menor na liquidação:** conceder desconto automático "apenas em multa e juros" até um **valor limite**; acima do limite, "**cobrar diferença em novo título**" (com forma de recebimento configurável); travas: bloquear liquidação com **mais de 40% de pagamentos a menor** (com data de carência) e liquidação de títulos **vencidos há mais de 180 dias**.
  - Valor pago a maior pode ser considerado **juros**.
  - Permitir **reabrir caixa**; permitir cancelar título faturado em exercício anterior; **data de bloqueio de operações financeiras** (fechamento contábil) com "operadores com permissão" para além da data.
  - **Inadimplência → impedimentos:** "Gerar impedimentos para títulos em atraso: Para o responsável" (impedimento bloqueia boletim/matrícula conforme outros parâmetros).
  - **Inativação do aluno na turma:** solicitar (ou não) **cancelamento dos títulos relacionados**; política de faturamento de títulos já liquidados ("Não antecipar"); com ISAAC, a cobrança "é garantida pelo isaac e deve ser cancelada pela operação de inativar o aluno na turma".
  - **Troca de turma:** opção de migrar todos os títulos para a turma atual; verificação de **impedimento de matrícula** ao alterar turma; verificação de procedimentos de matrícula sem vínculo com a série.
  - **Pré-matrícula:** "Mudança da situação do aluno na turma **mediante pagamento do título de pré-matrícula**: identificar pagamento de qualquer parcela" — confirmação de matrícula dirigida por pagamento.
  - **Automação de matrícula:** regra "apenas cobranças com o mesmo serviço definidos na série/turma".
  - **Dia preferencial** de vencimento por responsável, com regra para mês posterior ao vencimento.
  - Caixa: numeração **sequencial global**, retirada automática de todas as formas ao fechar, conta de destino para retiradas, custódia para cheques, cartões repassados à conta da operadora; multi-empresa no caixa (permitir ou não recebimentos de empresas diferentes; aglutinar títulos de empresas diferentes).
  - Avisos OBS/FNC/COB/BIB ligáveis; observação automática no aluno ao conceder desconto (com tipo de ocorrência).
  - Bloco final: "Estes parâmetros podem ser alterados **apenas pela equipe da Activesoft**" (Messenger do boleto integrado, fila de remoção automática) — separação entre parâmetros da escola e do fornecedor.
- **Caixa físico:** abertura por operador, movimentações, fechamento, saldo, usuário de abertura; cheques pré-datados com "bom-para" influenciando o valor a receber; **custódia de cheques** com destinos (Depósito, Devolvido ao emissor, Tesouraria) e flag de impedimento.
- **Contas a pagar:** fornecedor, empresa, plano de contas (multi-select), centro de resultado, datas (emissão/vencimento/baixa), situação, "pago pelo caixa?"; módulo "núcleo financeiro" ativável (implantação progressiva: "Permite apenas parametrização").
- **Plano de contas:** hierárquico (1. RECEITAS / 2. DESPESAS, 99 contas), conta-pai, tipo Sintética/Analítica, natureza Entrada/Saída, nº de conta contábil para exportação.
- **NFS-e:** envio em lote de RPS, recriação de lotes com RPS rejeitados, variáveis CPFALUNO/MATRICULA/NOMEALUNO na discriminação do serviço, "nova rotina de envio da NFSe" parametrizável.
- **Negociação de títulos:** com regras de desconto, mensagens de erro específicas, possibilidade (parametrizável) de negociar título dentro de lote de cobrança registrada; "Multa / Juros Negociação" vira serviço próprio no título.
- **API pública financeira para parceiros:** `gerar_cobranca` (POST), `informacoes_boleto`, `calculo_multa_juros`, `forma_recebimento`, `servicos`, `empresas`.

---

## 9. Módulo Captação & Matrícula

**Propósito:** funil do lead à matrícula assinada: captação pública → ficha de inscrição → reserva de vaga → pré-matrícula → contrato/assinatura → enturmação.

**Telas:** `captacao`, `captacao_alunos` (formulário **público** — exige `?codigo da instituição` na URL; multi-tenant por link), `ficha_inscricao_novatos` (pública, idem), `matricula_online` (parâmetros), `confirmacao_matricula`, `procedimento_matricula`, `sequencias`, `assinatura_eletronica`, `situacao_contratos`, `status_matricula_plano_pagamento_isaac`, `status_rematricula_isaac`, `consulta_turma`.

**Regras de negócio observadas:**
- **Funil de captação** com "oportunidades ganhas / perdidas" (cards no dashboard) e **"WhatsApp do consultor"** (botão na captação) — existe o papel `USUARIO_VENDEDOR_CAPTACAO`.
- **Procedimentos de matrícula** = checklist documental por série × período (ex.: "6 FOTOS 3X4"), com **data limite de entrega**, obrigatoriedade (Obrigatório/Opcional), "permite envio no portal" e "exibir no Matrícula online" (Ambos/...). Pendência gera flag `status_pdm` no aluno; auditoria de procedimentos; exclusão em lote; impedimento de excluir matrícula/alterar turma com **solicitação de matrícula pendente de análise**; cobrança comunicada no detalhamento do aluno.
- **Sequenciais de matrícula** nomeados por contexto (MATRICULA_CEMP=100, MATRICULA_PETELECO=151, SOLICITACAO_2025=3075...) — numeração independente por unidade/tipo/ano.
- **Pré-matrícula dirigida por pagamento** (ver Financeiro): pagar qualquer parcela do título de pré-matrícula muda a situação do aluno na turma automaticamente.
- **Confirmação de matrícula** permite "emitir contrato manual"; contratos com **assinatura eletrônica via Clicksign** — dashboard monitora "Contratos sem assinatura há mais de 14 dias (Clicksign)".
- **Rematrícula/renovação:** solicitação de renovação com verificação de campos ocultos; fórmulas de matrícula ("cadastro de fórmulas de matrícula a selecionar", "fórmula de matrícula normal para RVA"); status de matrícula/rematrícula sincronizado com planos de pagamento **ISAAC**.
- Matrícula online exibe **termo de consentimento LGPD e termo de uso de imagem** (ver módulo LGPD).
- Restrições de elegibilidade ficam na turma (idade mín/máx, sexo, pré-requisito de vínculo em outra turma).

---

## 10. Módulo LGPD / Termos

**Propósito:** consentimento digital e privacidade.

**Telas:** `termo_consentimento`, `termo_uso_de_imagem`, `termo_uso_privacidade` (aceite da plataforma), páginas institucionais (Política de Privacidade, Política para crianças e adolescentes, FAQ Privacidade — links do Grupo Arco no shell de todas as telas).

**Regras observadas:**
- **Termo de consentimento:** flag Ativo + texto editável; o texto real cita a Lei 13.709, finalidades (cadastros, prospecção, bolsas filantrópicas, descontos comerciais) e canal do titular (e-mail do **DPO** + telefone). Exibido nos módulos: *Matrícula online - Ficha de inscrição, Reserva de vaga, Captação de alunos, Inscrição em eventos, Filantropia (ficha socioeconômica e novos cadastros) e Solicitações de alteração de dados cadastrais de alunos, responsáveis e professores*.
- **Termo de uso de imagem:** flag Ativo + rich-text; exibido em ficha de inscrição, reserva de vaga, inscrição em eventos e alteração cadastral de alunos.
- Consentimento também acionado em **solicitação de desconto pelo portal** (release note).
- API pública separa endpoints "dados sensíveis" (`lista_alunos_dados_sensiveis`, `lista_responsaveis_dados_sensiveis`) — classificação explícita de PII.
- Anti-padrão a corrigir no novo sistema: PII real no swagger público, 3 analytics simultâneos (GA + Clarity + PostHog/UserGuiding) rastreando menores, cookies sem flag Secure.

---

## 11. Módulo Calendário & Agendamentos

**Telas:** `calendario`/`calendario_escolar`/`calendario_eventos`, `feriado`/`feriados`, `agendamentos`, `eventos` (inscrição em eventos referenciada pelos termos LGPD).

**Regras observadas:**
- **Feriado:** data, nome, **efeito no financeiro** (Sim/Não — desloca vencimentos), **efeito na biblioteca** (prazos de devolução), escopo por **unidade** ("Todas" ou específica); operação "**Copiar entre períodos**" (reaproveitar calendário ano a ano). 17 feriados no tenant (nacionais + estaduais do Pará).
- Feriados interagem com o diário ("permitir criar aulas em feriados" = parametrizável).
- **Agendamentos:** a escola cadastra **disponibilidades** (sala, horário) e o responsável **marca reunião pelo portal** (permissão "Visualização de horários disponíveis para marcar uma reunião com a escola — Apenas responsáveis"); campos: data/hora, situação, **motivo do cancelamento**, responsável; relatório "Agendamento por período" com quadro resumo e dados da turma do aluno.
- Calendário de eventos integra fases de nota (datas de início/fim das aulas por fase ficam no sistema de avaliação).

---

## 12. Portais (Aluno / Responsável / Professor) & Controle de Acesso

**Propósito:** o mesmo sistema serve 4 personas via SSO contextual — `/portal/aluno/`, `/portal/responsavel/`, `/portal/professor/` rodam **no mesmo domínio**, roteados client-side por claim do JWT/sessão. O que cada perfil vê é minuciosamente parametrizado pela escola.

**Tela-chave:** `controle_de_acesso/portal/resps` (com abas Alunos / Responsáveis / Professores) — catálogo de permissões observado:
- **Critérios de acesso ao produto:** situação do aluno na turma (multi-select — 6 selecionadas no tenant) × período (7 selecionados) — define quem loga; URL de redirecionamento no logout.
- **Carteirinha estudantil** digital no app com data de validade configurável.
- **Lista de alunos da turma** visível (ou não) a alunos/responsáveis, com período de referência.
- **Permissões por situação do aluno** (matriz situação × recurso): impressão de relatórios (carteirinha, avaliação por competência, por relatório, boletim) para "alunos e responsáveis"; **marcar reunião** (apenas responsáveis); **exibir títulos de cobrança**; exibição na planilha de notas (apenas professores).
- **Toggles de funcionalidades** para aluno/responsável: frequência (menu "Entradas e Saídas" — dados de catraca), **boletim detalhado**, porta-arquivos (upload de arquivos pela família/professores), lista de professores da turma, comunicados, avaliação comportamental, notas (boletim), atividades avaliativas, quadro de horário, **leitor de gabarito de provas** e correção, **resultado de simulado ENEM**, **atualizar dados cadastrais** (responsável edita o próprio cadastro; pode até alterar o próprio login).
- **Contato com a instituição:** habilita envio de e-mail in-app com endereços por setor (secretaria, tesouraria, direção, biblioteca) + tipo "Sugestão ou crítica".
- **Ocorrências visíveis ao responsável por categoria:** Pedagógicas / Disciplinares / Financeiras / Psicológicas (cada uma lig/desl).
- **Ficha médica:** responsável pode visualizar e **editar** (toggle).
- **Menu Financeiro** no acesso do responsável (consulta a títulos, segunda via).
- **Portal do professor:** digitação/confirmação de notas, diário, chamada online, avaliação por relatório/competência — regido pelos parâmetros do módulo 6.

---

## 13. Configurações & Administração

**Telas/funcionalidades:**
- `unidade` — multi-unidade por tenant (nome, número, ativa) + campos dinâmicos por unidade
- `empresa`/`instituicao` — multi-empresa (CNPJs distintos para faturamento)
- `usuarios`, `perfis`, `meu_perfil` (e-mail, senha, celular, telefone comercial, **setor** — ex.: Administrativo)
- `auditoria`/`audit`/`logs` — botão "Auditoria" presente em praticamente toda tela de configuração e operação financeira (trilha por entidade); modal de auditoria de alunos
- `gerador_consultas` — query builder/SQL (Monaco editor) com consultas salvas por categoria (Sistema, Financeiro...), importação/exportação de consultas — usado inclusive para extrações ad-hoc (ex.: "ClassApp - Exportação de EID")
- `relatorios` — central de relatórios: modelos prontos sugeridos (Relação de alunos, por turma com assinatura, Contas a receber, Boletim escolar, Matrículas efetivadas), **relatórios favoritos**, relatórios personalizáveis, `relatorios_exportados` (fila de geração)
- `exportacao_educacenso` — censo escolar MEC por blocos oficiais: 00 Identificação da escola, 10 Caracterização/infraestrutura, 20 Turmas, 30 Pessoas físicas, 40 Gestores escolares, 50 Profissionais, 60 Vínculo dos alunos; valida campos obrigatórios por unidade antes de exportar
- `gerar_emails_gfe` — provisionamento de e-mails Google for Education
- `indicadores_dashboard` — KPIs: **ocupação de turmas** (alunos ativos × capacidade, até 3 séries), **inadimplência em R$** (link para títulos em atraso), títulos com pendência de registro de boleto, oportunidades ganhas/perdidas (captação), contratos sem assinatura >14 dias
- `fila_processamento` — fila assíncrona (notas) com log por item
- `assistente_virtual` — help center contextual: artigos por rota (`?url=/tela/`) + Zendesk embutido; `central_novidades` (changelog do produto)
- `campo_dinamico` — campos customizados por escola (aparecem no cadastro do aluno)
- Cadastros auxiliares: `profissao`, `religiao`, `tipo_responsavel` (parentesco), `forma_entrega_documento`, `situacao_aluno`, `motivo_inativacao`, `tipo_horario`, `sequencias`
- `secretaria`/`secretaria_digital`, `biblioteca`, `livraria`/`livros` (módulos satélites; biblioteca gera pendência BIB e sofre efeito de feriado)
- `whitelist`, `feature` — gestão de feature flags
- `roteiro_treinamento/rva_ria` — trilhas de treinamento de usuários (onboarding com contagem de treinamentos pendentes/realizados por usuário)

---

## 14. Integrações (15 terceiros mapeados)

| Integração | Função |
|---|---|
| **ISAAC Pagamentos** | Garantidora de mensalidade: filas de registro/cancelamento de títulos, operadores, vínculo serviço↔produto, status matrícula/rematrícula; cobrança "garantida pelo isaac" só cancelável inativando o aluno na turma |
| **PJ Bank** | Importação de extrato/conciliação |
| **ClassApp** | App de comunicação com famílias: push, chat, tags, sync forçada, log de integração, régua de cobrança mobile |
| **Microsoft Teams** | Sync de turmas/disciplinas |
| **Edebê** | Editora — SSO/sync de usuários e colaboradores via API pública |
| **Banco do Brasil API / CAIXA API / Itaú API (bolecode)** | Registro de boleto online / boleto híbrido |
| **PIX** | Recebimento imediato (`ACESSO_PIX_IMEDIATO`) |
| **ReCB** | Recebimento eletrônico (feature-flagged por cliente) |
| **Clicksign** | Assinatura eletrônica de contratos |
| **ViaCEP** | Autocomplete de endereço |
| **Zendesk** | Suporte embutido |
| **Metabase** | Dashboards embedded (JWT) |
| **Google for Education** | Provisionamento de contas |
| **Catraca GPA** | Controle de acesso físico + frequência (API `identificar_marcacao`/`marcar_frequencia_aluno`) |
| **API pública SigaWeb** | 40 endpoints `/api/v0/` (Bearer por instituição): listas de alunos/responsáveis/colaboradores (com variantes "dados sensíveis" e "endereço"), enturmação, diários, frequência (GET/POST), **correção de prova em lote (POST)**, boletim, financeiro (gerar cobrança, boleto, multa/juros), porta-arquivos (signed URL) |

---

## 15. Perfis de Usuário / Personas

Constantes do bundle: `USUARIO_TIPO`, `USUARIO_CARGO`, `USUARIO_GRUPO`, `USUARIO_OPERADOR`, `USUARIO_SUPORTE`, `USUARIO_VENDEDOR_CAPTACAO` — modelo de usuário com tipo + cargo + grupo de permissões.

1. **Admin/Secretaria (operador interno)** — persona capturada no pilot. Acesso total ao backoffice: cadastros, diários, financeiro, configurações. Opera **logado em uma unidade** (parâmetro "exibir dados apenas da unidade em que o operador está logado"); permissões granulares por operador (ex.: lista de "operadores com permissão" para operar após a data de bloqueio financeiro). Setor declarado no perfil (ex.: "Administrativo").
2. **Operador de caixa/tesouraria** — abre/fecha caixa nominal, faz liquidações, retiradas; reabertura de caixa é permissão parametrizada.
3. **Coordenador** — campo "coordenador responsável" na turma; API pública `lista_coordenadores`; assinaturas de coordenação no boletim; recebe fluxo de confirmação de notas via Messenger.
4. **Professor** (`/portal/professor/`) — acesso **ativado pela alocação em turma×disciplina**; lança aulas/conteúdo/tarefas/chamada (restringível ao quadro de horário), digita notas, **confirma/remove confirmação de notas** (parametrizável), pode solicitar confirmação, pode (ou não) alterar fórmula, confirma avaliação por relatório/competência; chamada online.
5. **Responsável** (`/portal/responsavel/` + app) — vê boletim, frequência/entradas-saídas, ocorrências (por categoria liberada), comunicados, chat, títulos/2ª via, agenda reuniões, atualiza cadastro e ficha médica, assina termos LGPD e contratos; acesso condicionado a aluno ativo em período/situação elegível; papéis distintos: responsável **financeiro** vs **pedagógico** vs filiação.
6. **Aluno** (`/portal/aluno/`) — boletim/notas, atividades avaliativas, quadro de horário, carteirinha estudantil digital, comunicados, porta-arquivos, resultado de simulado ENEM, avaliação institucional.
7. **Vendedor de captação** (`USUARIO_VENDEDOR_CAPTACAO`) — opera o funil de captação/oportunidades, com WhatsApp do consultor; ciclo B2B consultivo.
8. **Suporte Activesoft** (`USUARIO_OPERADOR`/`USUARIO_SUPORTE`) — usuários do fornecedor dentro do ambiente do cliente; certos parâmetros "podem ser alterados apenas pela equipe da Activesoft".
9. **Parceiro de API** — não-humano; Bearer token por instituição, consome/escreve via `/api/v0/` (editoras, catracas, plataformas de matrícula).

---

## 16. Configurabilidade transversal (o que cada escola parametriza)

- **Vocabulário e taxonomias:** situações de aluno, motivos de inativação, parentescos, profissões, religiões, formas de entrega de documento, tipos de ocorrência, tipos de horário, campos dinâmicos.
- **Motor pedagógico:** sistemas de avaliação por série, fases de nota, fórmulas de composição (DSL própria), conceitos (nota vs percentual), regras de aprovação por nota E frequência com roteamento de fases, casas decimais e truncamento, textos de dispensa/2ª chamada, definições de boletim por série e modelo de boletim (dezenas de toggles), histórico escolar (cálculo de CH, cursos emissores, formatação).
- **Diário:** ~25 parâmetros (datas, feriados, quadro de horário, frequência justificada/dispensada e seu peso na aprovação, exigências de conteúdo, integração com planilha, renomear menu).
- **Financeiro:** multa/juros nomeados, descontos com regra de concessão e ordem de cálculo, planos de pagamento, formas de recebimento, agentes de cobrança CNAB, régua automática, ~40 parâmetros globais (pagamento a menor, impedimentos, cancelamento na inativação, automação de matrícula, caixa, NFS-e).
- **Portal/app:** matriz de permissões por persona × situação do aluno × recurso; e-mails por setor; termos LGPD com texto próprio.
- **Comunicação:** templates HTML com variáveis, remetente, tipos de mensagem.
- **Plataforma:** multi-unidade, multi-empresa, feature flags por instituição, tiers comerciais (Light/Basic/...), sequenciais de numeração.

---

## 17. Apontamentos para o SDD do novo sistema

1. O **par "situação do aluno na turma" (configurável, com mapeamento para situação-sistema/acadêmica/Educacenso/digitação-de-nota)** é a peça de estado central — quase todo módulo filtra por ela.
2. O motor de notas é **fórmula-dirigido com fila assíncrona** (digitar → confirmar → processar → boletim), com aprovação dupla (nota E frequência) e fases de recuperação roteadas por fórmula — não hardcodar bimestres.
3. Financeiro e acadêmico são acoplados por desenho: pagamento confirma pré-matrícula; inadimplência gera impedimento que pode bloquear boletim; inativação na turma cancela títulos; série carrega o serviço de mensalidade.
4. A **caixa de saída** (audit trail de comunicação) e a **auditoria onipresente** são requisitos de confiança não negociáveis para o público escola.
5. Corrigir as dívidas observadas: PII em docs públicas, analytics sobre menores, dados sujos em cadastros auxiliares, acessibilidade.
