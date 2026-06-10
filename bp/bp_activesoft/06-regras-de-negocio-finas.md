# 06 — Regras de Negócio Finas e Detalhes Operacionais

> **Fontes:** `_janus_spec/ui/SDD.md`, `_janus_spec/ui/glossario.md` (1.973 termos / 310 entidades / 484 relações),
> `.janus/inferences/*.json`, `.janus/analyses/*.md` e texto visível + respostas de API extraídos de
> `.janus/captures/<slug>/default/{dom.html,har.json}` (318 telas).
> Instância observada: `siga04.activesoft.com.br`, cliente CEMP / Mundo Montessori (3 unidades, 2 empresas).
> **Os valores concretos citados são parametrizações do tenant observado** — servem como exemplo, não como default.

---

## 1. Validações e máscaras

### 1.1 Mensagens de validação/aviso literais capturadas
- **Alterar notas em lote** (`alterar_notas_lote`): *"As fases de nota dos tipos 'Calculada' e 'Fórmula virtual' não podem sofrer alterações em lote. Portanto, planilhas com esse tipo de fase não são listadas abaixo."*
- **Finalizar turmas** (`finalizar_turmas`): *"Atenção! Após o processamento, não será possível reverter a finalização das turmas."* (operação irreversível).
- **Conceitos** (`nota_conceito`): *"Atenção à unidade de medida — Os conceitos devem ser criados usando uma única unidade de medida (nota OU percentual). Ao criar o primeiro conceito em uma série, todos os demais devem seguir o mesmo padrão."* (o primeiro conceito da série trava a unidade de medida).
- **Exportação Educacenso** (`exportacao_educacenso`): *"Antes de exportar o arquivo, é necessário que todos os campos de 'preenchimento obrigatório' de todas as suas unidades sejam preenchidos."*
- **Contas a pagar** (`contas_pagar`): *"Atenção — O módulo núcleo financeiro não está habilitado para integração. Caso já esteja com a parametrização concluída, habilite a utilização integrada do módulo."*
- **Fila de boleto integrado** (`integracao_agente`): *"Apenas registros com o status 'Aguardando' podem ser excluídos"*; *"O prazo para confirmação de registro no PJBank é de até 10 minutos quando o banco emissor é Banco do Brasil e de até 3 horas quando o banco emissor é BPP"*; *"Títulos garantidos pelo isaac são registrados em tempo real. Caso haja alguma instabilidade na integração, os títulos serão registrados em até 10 minutos após a criação."*
- **Envio de senhas** (`envio_de_senhas`): *"Será enviado um e-mail, para os usuários selecionados, com um link para recuperação da senha. Apenas usuários com e-mail cadastrado estão aptos a seleção."*
- **Parâmetros financeiros**: *"Esta configuração será considerada apenas nas próximas liquidações que forem efetuadas"* (alterações não retroagem); seção final: *"Estes parâmetros podem ser alterados apenas pela equipe da Activesoft."* (parâmetros bloqueados: Messenger do boleto integrado com execução automática; inserção automática de títulos na fila de remoção de boleto integrado).
- **Relatório de ticket médio**: label de erro **"Campo obrigatório"** sob o filtro Turmas — turma é obrigatória para gerar o relatório (que também exige Data de faturamento e Data de corte).
- **Agendamentos**: *"Os relatórios gerados não informam dados sobre horários disponíveis."*
- **API parceiros** (`docs`): toda API exige parâmetro `version` (=0); autorização via `Bearer <token>`; **cada token libera acesso a uma única instituição**.

### 1.2 Máscaras e formatos observados
- Datas: placeholder `DD/MM/AAAA` (65 ocorrências em telas); competência/faturamento: `MM/YYYY` (relatórios de descontos e consolidação contábil).
- CPF exibido mascarado para LGPD no cadastro do aluno: `633.XXX.XXX-20` (tela `alunos/{id}/detalhe`).
- Educacenso (`/api/v1/exportacao_educacenso/campos/?unidade=N`): cada campo carrega `tamanhoMinimo`, `tamanhoMaximo`, `stObrigatorio`, `stPreenchidoPeloUsuario`, `valorPadrao` e `instrucoesPreenchimento` com domínios enumerados — ex.: "Tipo de registro" valor fixo `00` (2 chars); "Código de escola - Inep" 8 chars; "Situação de funcionamento" 1 char, aceita somente `1 – Em atividade, 2 – Paralisada, 3 – Extinta`; "Nome da escola" 1–100 chars. Layouts de registro: 00 (identificação da escola), 10 (caracterização/infraestrutura), 20 (turmas), 30 (pessoas físicas), 40 (gestores), 50 (profissionais), 60 (vínculo dos alunos).
- E-mail GFE: domínio com placeholder `@colegioactivesoft.com.br`.

### 1.3 Restrições de cadastro de turma (`turmas--cadastrar`)
- "Dados restringentes" da turma: **data de nascimento mínima e máxima** do aluno, **sexo** restringível, e pré-requisito *"Exigir que o aluno já esteja vinculado em outra turma"* com turma e situação do aluno na turma exigidas.
- Demais campos com efeito de regra: qtde máx de alunos (vagas), serviço relativo à mensalidade, tipo de horário, código de agrupamento, código INEP, situações default do aluno na disciplina (Cursando/Aprovado/Reprovado), "Utiliza grade curricular", "Permitir acesso ao portal educacional", "Permitir integração com aplicativos de comunicação".
- Da entidade `Serie` (glossário): `permite_matricula_repetida`, `prefixo_matricula`, `proxima_serie` (promoção), `quantidade_dependencias` e `serie_dependencia` (progressão parcial), `situacao_aprovado/cursando/pparcial/reprovado` default por série, `utiliza_avaliacao_meta/nota/relatorio`, `utiliza_rotina_educacao_infantil`, `recalcula_matricula_identificadora_enturmacao`, `inscrturma_qtdediasuteis_solicitacaovalida` (validade da solicitação de inscrição em dias úteis).

### 1.4 Ficha médica do aluno (44 atributos)
Campos booleanos condicionais com campo-texto dependente: `alergico`→`nome_alergia`, `doenca_congenita`→`nome_doenca_congenita`, `em_tratamento_medico`→`nome_tratamento_medico`, `medicacao_especifica`→`nome_medicacao_especifica`, `plano_saude`→`nome_plano_saude`; doenças (catapora, caxumba, coqueluche, escarlatina, rubéola, sarampo, `outras_doenca_contagiosa`); deficiências (auditiva, física, mental, visual + `deficiencia_descricao` e `possui_laudo_medico`); hospital de remoção (nome, endereço, telefone), médico (nome, telefone, tipo), `nome_parente_aavisar`, `numero_sus`, `tipo_sangue`, `restricoes_alimentares`, `nome_remedio_febre`.

---

## 2. Cálculos e fórmulas

### 2.1 Multa e juros (`calculo_multa_juros`)
- Modelo: cada "cálculo" tem **Percentual da multa** OU **Valor da multa** (fixo) e **Percentual de juros ao mês** (`percentual_juros_ao_mes`) OU **Valor diário de juros** (`valor_juros_ao_dia`) — percentual e valor são alternativos.
- Dados reais: "Multa e Juros - Padrão" = multa 2,00% + juros 1,00% a.m.; "Sem Multa e Juros" = 0,00%/0,00%. A unidade tem `tipo_calculo_multa_juros_padrao` (default por unidade).
- No título: serviço próprio de negociação "Multa / Juros Negociação - CEMP" aparece como item de serviço do título (multa/juros viram componente do título negociado).

### 2.2 Composição de notas, aprovação e arredondamento
- **Fase de nota** (API `/api/v1/fase_nota/crud/{id}`): tipo de fase (`tipo_informada`) pode ser digitada, **"Informada por fórmula de composição"**, "Calculada" ou "Fórmula virtual"; campos: `media_minima_aprovacao` = 7.00, `valor_nota_maxima` = 10.00, `valor_arredondamento_media`, `numero_semanas_letivas` = 12, `numero_dias_letivos` = 60, `data_limite_digitacao_nota`, `data_inicio_exibicao` (data em que a nota aparece no Portal Web), `data_inicial/final_periodo_aula`.
- **Fórmula de verificação de aprovação literal**: `([CF01/01]+[CF01/02])>=4.5` — sintaxe `[CF<fase>/<componente>]`. Roteamento condicional: *fase de nota caso aprovado* = MÉDIA 1º BIMESTRE, *caso reprovado* = RECUPERAÇÃO 1º BIMESTRE — ou, alternativamente, **situação** caso aprovado/reprovado (a fórmula decide o fluxo do aluno entre fases).
- **Verificação por frequência** (texto literal): *"serão analisadas as duas verificações para que o aluno seja aprovado. Ou seja, caso o aluno tenha obtido a nota para aprovação, mas não atingiu a fórmula de frequência, será reprovado. Será aprovado apenas quando atender aos dois critérios"* — aprovação = nota **E** frequência; existe `situacao_reprovado_frequencia` específica.
- **Estrutura de fases observada (Fund II)**: 15 fases — 1º BIM → RECUPERAÇÃO 1º BIM → MÉDIA 1º BIM (× 4 bimestres) + "Precisa de" + TOTAL ANUAL + MÉDIA FINAL; `numero_fase` (ordem interna) ≠ `numero_exibicao` (ordem no boletim).
- **Dispensa automática**: *"Durante a confirmação das notas, caso os alunos cheguem a esta fase, eles devem ser automaticamente dispensados — desde que a fórmula desta fase de nota contenha apenas uma única nota de composição"* (`dispensar_nota_automatica`, `tipo_dispensa_automatica`, `confirmar_dispensa_automatica`).
- **Parâmetros de cálculo por unidade**: `nota_casas_decimais` = 1; modo **Truncar** ou Arredondar (`nota_truncar_arredondar`); texto exibido quando dispensado = `F`, quando vai a 2ª chamada = `2CH`, no lugar de nota de fase = `*`; *"Como a dispensa deve interferir no cálculo de nota da fase?"* → **"A nota de composição deve ser interpretada como zero"**; *"Como tratar a situação final das disciplinas?"* → **"A disciplina componente deverá ser igual à disciplina composta"**; cores na tabela de notas com média de corte = 7 (`nota_exibir_cor_media`).
- **Disciplina composta**: `calcular_disciplina_composta`, `calc_disc_comp_utiliza_formula_fn`, `tipo_composicao` e `formula_composicao` na grade curricular; exibição opcional das médias das componentes no boletim.

### 2.3 Carga horária do histórico escolar
- Fórmula da CH total (parametrizável): **"Semanas letivas × Carga horária semanal da disciplina"** ou **"Semanas letivas × Carga horária aula × Quantidade de aulas semanais"**; alternativa por **soma de aulas dadas** (`historico_carga_horaria_soma_aulas_dadas`); CH em disciplinas onde o aluno não está apto = **"Zerada"** (`historico_ch_zerada_disciplina_nao_apta`). Inclusão de disciplinas dispensadas/inaptas é opcional (default Não).
- Grade curricular guarda `carga_anual`, `carga_semanal`, `carga_horaria_aula_hora_minuto`, `qtde_aula_semanal_hora_minuto` (suporte a hora-relógio e hora-aula).

### 2.4 Frequência
- *"Contabilizar 'Falta justificada' para aprovação do aluno por frequência como:"* → **"Presença (Falta abonada)"** (config do diário).
- Tipos especiais de frequência habilitáveis: "Dispensada" (`diario_permite_dispensa_frequencia`, off no cliente) e "Falta justificada" (on).
- Boletim: cálculo de faltas configurável por fase (`tipo_calculo_faltas`, `tipo_calculo_faltas_fase_nota`; exibição "Número de faltas" vs frequência em %).

### 2.5 Liquidação a menor / a maior (parâmetros globais financeiros — valores reais)
- Recebido **a menor**: *"Conceder desconto quando recebido a menor..."* — modalidade **"Apenas em multa e juros"** (`tipo_concessao_desconto_recebido_menor`=0); **valor-limite do desconto automático = R$ 1,00** (`valor_liquid_conceder_desconto`:1.0); acima do limite → **"Cobrar diferença em novo título"** (`cobrar_diferenca_liquidado_menor`:1), com forma de recebimento configurável para o novo título.
- Recebido **a maior**: *"Considerar o valor a maior como juros"* até R$ 1,00 (`valor_liquid_acrescentar_juros`:1.0); título guarda `valor_recebido_a_maior`, `data_devolucao_recebido_a_maior`, `liq_nome_conta_devolucao_recebido_a_maior`, `liq_valor_devolvido` (devolução rastreada).
- Trava de sanidade em lote: *"Permitir processamento de liquidação com mais de 40% de pagamentos a menor?"* e *"Permitir processamento de liquidação de títulos vencidos a mais de 180 dias?"* — cada uma com data "Não realizar a verificação até o dia..." (janela de carência da validação).
- Caixa pode liquidar a menor/a maior e conceder desconto maior que multa+juros conforme flags por unidade (`caixa_pode_liquidar_menor_maior`, `caixa_pode_conceder_desconto_maior_valor_multa_juros`); caixa com limite de desconto concedido por percentual e por valor (`caixa_limitedescontoconcedidopercentual/valor`).

### 2.6 Descontos e bolsas
- Desconto (entidade): **`ordem_calculo`** (sequência de aplicação quando há múltiplos descontos), `percentual_desconto`, `tipo_abatimento`/`tipo_desconto`, `dia_desconto_condicional`, **regra de concessão** com valores observados: *"Até o vencimento"* (condicional — expira) e *"Não condicionado (Nunca expira)"*; classificação contábil (ex.: Comercial).
- Bolsa por aluno-turma (`Aluno Turma Bolsa`): `percentual_abatimento` OU `valor_abatimento`, vigência `data_inicial`/`data_final`, `usuario_autorizacao` e `solicitacao_desconto_origem` (workflow de autorização). Exemplo real: desconto "2º FILHO" 10,00% de 01/01/2024 a 01/01/2025.
- Título expõe `valor_desconto_hoje` (desconto vigente calculado por data) e `mensagem_desconto`.
- Parâmetro: *"Inserir observação no aluno caso haja concessão de descontos?"* com tipo de ocorrência configurável.
- Relatório de faturamento tem modo *"Considerando apenas descontos não-condicionados"* (descontos condicionais não reduzem faturamento contábil).
- Período controla `disponivel_solicitacao_desconto` (janela para solicitar desconto por ano letivo).

### 2.7 Indicadores
- Dashboard: **Inadimplência em R$** (títulos em atraso), títulos com pendência no registro de boleto, ocupação de turmas = *"Quantidade de alunos por turma × Capacidade da turma"* (máx. 3 séries por gráfico), oportunidades ganhas/perdidas (captação) e **"Contratos sem assinatura há mais de 14 dias (Clicksign)"** (SLA de assinatura de contrato = 14 dias).
- Ranking de alunos: calculado por fase de nota definida em `fase_nota_calculo_ranking`; exibição no boletim opcional (`exibir_ranking`).

---

## 3. Máquinas de estado

### 3.1 Situação do aluno na turma (21 situações configuráveis; API `/api/v1/situacao_aluno/`)
Cada situação nomeada mapeia para uma **situação de sistema** fixa: `S`=Seleção, `P`=Pré-Matrícula, `A`=Ativo, `I`=Inativo; e carrega flags de comportamento: `st_permitir_digitacao_nota_falta`, `permite_assinatura_eletronica`, `situacao_academica` (A=Aprovado / R=Reprovado) e `situacao_educacenso`. Valores reais:

| Situação | Sistema | Acadêmica | Digita nota | Assin. eletrônica |
|---|---|---|---|---|
| Lista de espera | Seleção (S) | — | Não | Não |
| Fichamento | Seleção (S) | — | Não | Não |
| Pré-matrícula online | Pré-Matrícula (P) | — | Não | Não |
| Pré-matrícula | Pré-Matrícula (P) | — | Não | Não |
| Cursando | Ativo (A) | — | **Sim** | **Sim** (única) |
| Evadido / Transferido / Cancelado | Inativo (I) | — | Não | Não |
| Aprovado | Inativo (I) | A | **Sim** | Não |
| Reprovado | Inativo (I) | R | **Sim** | Não |

- Transições suportadas: Lista de espera/Fichamento → Pré-matrícula (online) → Cursando → {Aprovado, Reprovado, Evadido, Transferido, Cancelado}; "Finalizar turmas" e "Promover entre séries" fazem a transição em massa; "Inativar alunos na turma (em lote)" leva a inativo com **motivo de inativação** (cadastro próprio: Cancelado, Desistência, Matrícula Suspensa, Transferido, Troca de turma) e `estab_ensino_inativacao` (escola de destino na transferência).
- **Gatilho de pré-matrícula → matrícula por pagamento** (parâmetro): *"Mudança da situação do aluno na turma mediante pagamento do título de pré-matrícula"* — modo observado: **"Identificar pagamento de qualquer parcela de pré-matrícula"** (alternativa: todas as parcelas).
- Enturmação (`alunoturma`) carrega ainda: `data_efetivacao_matricula`, `data_situacao_ativo`, `data_situacao_inativo`, `comentario_inativacao`, "situação exceção" (`situacao_excecao_*`: situação + usuário + comentário + data — override auditado), `problema_autorizado_matricula` + `justificativa_autorizacao_matricula` + `usuario_autorizacao_matricula` (matrícula autorizada apesar de impedimento, com justificativa e autor), planos de pagamento distintos para pré-matrícula, matrícula e solicitação online, progressão parcial (`turma_progressao_parcial`, `qtd_turma_progressao_parcial`).

### 3.2 Título de cobrança
- `situacao_titulo_cobranca` em código de 3 letras: observado `ABE` → "Em aberto"; filtros da tela indicam demais estados (liquidado/baixado — `data_baixa`, `data_pagamento` —, cancelado, negociado).
- Estados paralelos independentes no mesmo título: **cobrança registrada** (`situacao_cobranca_registrada` — "Registro confirmado"; `cob_registrada_aviso_pendente`), **situação no agente** (`situacao_no_agente`), **pendência isaac** (`possui_pendencia_isaac`), pendência de registro de boleto (`descricao_pendencia`), tipo de liquidação (`tipo_liquidacao`), `st_anuidade`.
- `tipo_recebimento` do título: `DIR` (direto/caixa da escola) vs boleto bancário vs boleto integrado/híbrido (tipos vistos: "Boleto integrado / Híbrido - Itaú API", "Direto", "Boleto bancário").
- **Fila de boleto integrado** (`integracao_agente`): operação (inclusão/remoção) × situação; só status "Aguardando" é excluível; SLAs: PJBank-BB 10 min, BPP 3 h, isaac tempo real (fallback 10 min).
- **Lote de cobrança registrada** (`cobranca_registrada`): lote tem agente, tipo, situação (filtro default "Abertos") e data de fechamento; arquivo de remessa por agente no formato **CNAB 400** (Santander observado). Agente de cobrança configura: `gera_impedimento`, `apenas_titulo_aberto`, `liquidar diretamente na escola`, `permite_cobranca_registrada`.

### 3.3 Mensagens (caixa de saída)
- Ciclo: **inserção → (agendamento `data_hora_agendamento` / `data_limite_para_envio`) → envio (`data_hora_envio`) → log** (*"Mensagem e-mail enviada para X"*). Tipos: Email / SMS / Mobile; tipo de destinatário: Aluno / Responsável; flags `permite_remocao_auto_boleto_integrado` e `processar_boleto_integrado_messenger` ligam mensagem ao fluxo de boleto.

### 3.4 Diário, sistema de avaliação e caixa
- Situação do diário: "Disponível" (entre outras); limites por diário: **mín 1 / máx 100 aulas**, prazo para criar aulas, prazo para registrar conteúdo, limite para digitação de notas, e regra *"Aulas devem estar no período da fase de nota? Sim"*.
- Conclusão do diário exige (configurável) conteúdo digitado e tarefas digitadas.
- Aluno no diário tem entrada/saída (`entrada no diário`, `saída do diário`) e situação própria.
- Sistema de avaliação: estados **Ativo/Inativo × Bloqueado/Desbloqueado × Validação Pendente/Validado** (ação "Validar o sistema"; `sistema_avaliacao_bloqueado` reaparece em `Configuracoes Serie Periodo`).
- Caixa (tesouraria): aberto → fechado (com `stcaixaaberto`, `stdatahoraaberturavencida` — abertura "vencida"), reabertura só se `permite_reabrir_caixa`.

---

## 4. Processos em lote e processamento assíncrono

- **Alterar notas em lote** — wizard de 4 passos: *Filtrar → Editar → Simulação → Processamento* (há etapa de **simulação** antes de efetivar); exclui fases Calculada/Fórmula virtual; filtros incluem "Alunos com notas / Em qualquer situação" e professor.
- **Frequência em lote** (`diarios/frequencia_em_lote`): por período/curso/série/turma/fase de nota/**intervalo de aulas**/disciplinas/situação do aluno.
- **Finalizar turmas** — 3 passos: *Selecionar turmas e alunos → Selecionar nova situação → Processamento*; irreversível.
- **Inativar alunos na turma em lote** — 3 passos; filtro especial *"Exibir apenas alunos com prazos vencidos"* (prazo de fichamento/seleção vencido); situação de origem default "Fichamento (Seleção)".
- **Promover entre séries** (`promocao_aluno_turma`) — 2 passos (origem/destino → confirmação); filtros por situação na turma de origem e por **alunos impedidos / não impedidos** (impedimento financeiro filtra a promoção); define situação do aluno na turma de destino.
- **Reprocessamento de notas**: botão na planilha de notas e em configurações; **Fila de processamento** (`fila_processamento`): registros com Data/Hora inserção, Turma, Disciplina, Aluno, Data/Hora processamento e **Log de processamento** (campos `atencao` e `erro` na API) — processamento de notas é assíncrono via fila; parâmetro "O sistema deve ativar o uso do Messenger para o processamento de notas".
- **Permitido reprocessar nota ao registrar frequência mesmo com fase confirmada** (config diário: Sim).
- **Enviar nova mensagem** — 3 passos: *Filtrar → Selecionar → Nova mensagem*; regras: *"Caso o responsável esteja vinculado a mais de um aluno, enviar mensagem uma única vez?"* (dedupe) e *"Listar pessoas atreladas ao aluno mesmo que ele não possua vínculo à turma(s)?"*; aviso: *"Caso deseje fazer o envio de cobranças em lote, usar a tela de títulos e operações"*.
- **Régua de cobrança automática**: *"O sistema realiza o envio das mensagens diariamente às 14h00"*. Regras = offset em dias do vencimento (`quantidade_dias_para_vencimento`: **-15, -5, 0** observados) × canal (e-mail/SMS/mobile) × lista de serviços (`array_id_servico`) × formas de recebimento (`array_id_forma_recebimento`) × texto personalizado por regra × conta SMTP; flag `enviar_para_titulos_registrados`. Assuntos reais: "...Boleto da mensalidade escolar disponível (VENCE EM 5 DIAS)" / "(VENCE HOJE)".
- **Operações em lote em Contas a Receber**: Imprimir boletos/carnê, Gerar Recibo, **Gerar taxa**, Negociações, Imposto de renda; modo padrão vs modo relatório.
- **Cancelar títulos Isaac (em lote)** (`cancelar_titulos_isaac_lote`) e fila de registro isaac (`titulos_isaac_fila_registro`); endpoint `titulos_isaac_pre_matricula_pendentes/?aluno_id=` consultado ao abrir a aba de cobrança do aluno (`{"possuiPendencia":false}`).
- **Gerar e-mails para Google for Education** — 3 passos; 3 regras de geração: `primeironome + inicial do 2º + . + último sobrenome @dominio`, `primeironome.últimosobrenome@dominio`, `matricula@dominio`; opção "criar apenas para alunos sem e-mail".
- **Exportação Educacenso** — 2 passos (dados de referência → processamento), bloqueada até preencher obrigatórios de todas as unidades.
- **Envio de senhas em lote** — wizard por tipo de acesso, restrito a usuários com e-mail.

---

## 5. Documentos emitidos

- **Boletim escolar**: modelo "Padrão" ou "Personalizável" **por série/período** (`definicoes_boletim_serie` → `Configuracoes Serie Periodo` com ~60 flags): orientação de impressão, origem do cabeçalho, foto/data de nascimento/nacionalidade/filiação/endereço do aluno, zero por extenso, nota de fase em negrito, aulas dadas por fase, faltas por fase vs total, situação na disciplina, separador por fases, disciplinas dispensadas, médias das componentes, frequência em %, agrupamento de disciplinas, cores por nota, tamanho de fonte, ranking, progressão parcial, mensagens personalizadas (unidade/série), observações do aluno na turma, **assinaturas configuráveis** (Direção/Secretaria/Coordenação com imagens cadastradas) e **"Protocolo de recebimento do boletim pelo responsável"**; textos personalizados quando "todas as NCs forem dispensadas" e quando "a disciplina não requer nota". Publicação controlada por fase (`data_inicio_exibicao`, `imprimir_boletim_nota`, `imprimir_boletim_nota_parcial`, `imprimir_boletim_np_nao_confirmada`, `imprimir_boletim_faltas`) e por "Bloquear boletim com impedimento" (`bloquear_boletim_impedimento`).
- **Histórico escolar** (`configuracao--historico_escolar`): *"As definições configuradas serão aplicadas a todos os históricos emitidos pela escola"*; *"Ao concluir a turma, as regras para geração serão aplicadas automaticamente no histórico para todos os cursos"*; cálculo de CH (semanal/anual/aulas dadas), inclusão de dispensadas/inaptas, **cursos com impressão liberada** por unidade (flags `st_imprime_historico_*`); totais por ano: dias letivos, faltas, média global, frequência; preview com "dados fictícios"; histórico guarda `nota_minima_para_aprovacao` e `resultado_final` por ano cursado; período tem `permite_alterar_nota_historico`.
- **Boletos/carnê** (impressão em lote), **2ª via** (`aluno_segunda_via`), **Recibos**, **Declaração de imposto de renda** (no acesso responsável, com períodos de referência configuráveis), **Declaração acadêmica** (`st_exibir_declaracao_academica`), **Carteirinha estudantil** (app, com validade configurada), **Contratos** com assinatura eletrônica **Clicksign** (modelo de contrato por turma: `modelo_contrato`, `modelo_contrato_rmonline_confirmacao`; alerta de não assinados >14 dias), **NFSe** (rotina nova via `utilizar_NFSe_novo`), **Ata de resultados finais** (`st_exibe_ata_resultados_finais` por turma), **grade curricular impressa** (análise descritiva por série, conteúdo programático, 1 disciplina por página).
- **Relatórios financeiros** com regras embutidas: Faturamento (por aluno; filtros por competência/emissão/vencimento; modo só não-condicionados), Recebimento (por data de baixa vs data de pagamento — distinção explícita; conta financeira; tipo de liquidação; participação em negociação), Cancelamentos (*"Repercussão na contabilidade: Apenas cancelamentos após o mês do faturamento"*), **Consolidação contábil** (*"Não são considerados serviços do tipo 'negociação'"*; data de corte = último dia do mês), Ticket médio (turmas obrigatórias + data de corte), Relação de títulos.
- Exportações: **Educacenso** (layout posicional INEP), **exportação contábil**, "Exportações salvas".

---

## 6. Parametrização por escola/unidade

### 6.1 Parâmetros globais financeiros (`/api/v1/parametro_global/financeiro/` — valores reais)
- **Data de bloqueio de operações financeiras** (`bloqueio_operacoes_financeiras_data`; trava retroativa) com lista de "operadores com permissão" para ignorar o bloqueio.
- Avisos em cadastro de aluno/responsável: **OBS** (observação com impedimento), **FNC** (inadimplente), **COB** (título em aberto em cobrança registrada), **BIB** (pendência de biblioteca) — flags `frm_aluno_verificar_*`. Estes códigos batem com os campos `status_obs/status_fnc/status_cob/status_bib/status_pdm` da listagem de alunos.
- Matrícula/turma: `verificar_pendencia_mudanca_turma`=true; `alterar_turma_todos_titulos`=true (títulos migram para a turma atual); `tipo_impedimento_matricula_titulo_atraso`="R"; impedimentos gerados "Para o responsável"; `cancelar_titulo_inativar_aluno` + `cancelar_titulo_inativar_aluno_antecipar_faturamento` ("Não antecipar").
- **Automação de matrícula**: `regra_gatilho_servicos`=1 → *"Apenas cobranças com o mesmo serviço definidos na série/turma vinculadas à turma"*; flag por plano (`possui_automacao_matricula`); `atualiza_matricula_automaticamente` por unidade; fórmulas de matrícula por unidade: `formula_matricula_normal`, `formula_matricula_temporaria`.
- Numeração: `id_titulo_cobranca_iniciar`=600000 ("Gerar título a partir do número"); numeração do caixa "Sequencial global para todas as aberturas e movimentações"; **sequenciais de matrícula nomeados** por unidade/ano (tela `sequencias`: MATRICULA_CEMP=100, MATRÍCULA_MM_2026=121, SOLICITACAO_2025=3075...; prefixo de matrícula por série).
- Multi-empresa: `financeiro_filtro_por_unidade`=true (operador só vê a própria unidade), `caixa_permite_receber_multi_empresa`=true, `aglutinacao_titulos_empresas_diferentes`=true (aglutinar títulos com serviços de empresas diferentes), título aglutinado indica data de faturamento.
- Outros: `permite_cancelar_titulo_faturado_em_exercicio_anterior`=true; `permite_negociar_titulo_cobranca_registrada`=true; `impedir_liquidacao_data_validade`=false; `considerar_dia_preferencial_proximo_mes` (dia preferencial de pagamento do responsável vale para o mês seguinte ao vencimento); `situacao_plano_pagamento`="S" (modo "Simultâneo"); `habilita_liquidacao_pix_imediato`; `permite_enviar_boleto_PDF`.
- Caixa: retirada automática ao fechar (todas as formas), conta para retiradas (exceto cartão/cheque/pix), custódia para retirada de cheques, *"Recebimentos em cartão serão repassados para a conta já definida no cadastro de operadora de cartão"*, cheque pré-datado valorizado pela data do "bom-para" (`caixa_calcular_valor_recebimento_cheque_pre`), "pagamento rápido" com conta tesouraria.

### 6.2 Parâmetros do portal/app (`/api/v1/parametro_internet/portal_web_geral/` — ~100 chaves)
- **Acesso**: situações do aluno que dão acesso ao portal (`acesso_portal_aluno_situacoes`) × períodos liberados; professor só acessa com situação ativa.
- **Permissões de edição pelo responsável**: alterar dados cadastrais, alterar login (apenas o próprio), alterar CPF/RG/endereço/registro de nascimento/nome do aluno/filiação (flags individuais `permite_alterar_*`); editar ficha médica.
- **Exibições** (por perfil aluno/responsável/professor): financeiro, boletim detalhado, frequência ("Entradas e Saídas"), comunicados, avaliação comportamental/por relatório/por competência, quadro de horário, atividades avaliativas, gabarito/correção de provas, resultado de simulado ENEM, porta-arquivos, lista de professores, lista de alunos da turma (com período de referência), ocorrências por categoria (pedagógicas/disciplinares/financeiras/psicológicas), fotograma, carteirinha.
- **Permissões por situação do aluno**: relatórios (carteirinha, avaliações, boletim), agendamento de reunião, exibição de títulos de cobrança, exibição na planilha de notas — todos condicionados à situação do aluno na turma.
- **Financeiro no portal**: faturamento inicial/final exibido, **descontos ocultáveis** (`array_id_desconto_boleto_web`), serviços exibíveis selecionáveis, declaração IR com períodos.
- **Professor**: confirmar/remover confirmação de notas pelo portal, solicitar confirmação mesmo sem todas as notas digitadas / sem faltas e aulas digitadas, alterar fórmula (`professor_alterar_formula`=false), lançar aulas só dentro do quadro de horário (=true), inserir ocorrências, ver só alunos aguardando nota da fase, importar apuração de frequência, chamada online.
- **Matrícula/reserva online** (`rmonline_*`): mensagens por etapa (inicial, dados do aluno, dados do responsável, ficha médica, seleção de plano de mensalidade, serviço adicional, confirmação, comentário adicional), flags de exibição de cada etapa, comentário adicional obrigatório ou não, situações elegíveis para reserva (`array_IdSituacao_aluno_turma_reserva_matricula`), período de referência da reserva, datas-limite (`reserva_matricula_data_limite`, `inscricao_turmas_data_limite`), boleto exibido na reserva, HTML do formulário (`matricula_online_html_formulario`).
- **Contato com a instituição**: e-mails por setor (secretaria/tesouraria/direção/biblioteca) e tipo de solicitação para críticas e sugestões.

### 6.3 Configurações do diário (tela `diarios--configuracoes`)
Registro de aulas: permitir aulas passadas (Sim) / futuras (Sim) / **em feriados (Não)**; copiar conteúdo entre disciplinas (Sim); criar aulas automaticamente a partir do quadro de horário, respeitando a quantidade máxima do diário; exigir conteúdo/tarefas (Não); quando houver registros anteriores pendentes → permitir "a digitação de conteúdos e frequências". Frequência: passada/futura permitidas; "Dispensada" (Não), "Falta justificada" (Sim), frequência em turma/disciplina alternativa (Sim), reprocessar com fase confirmada (Sim). Integração com planilha de notas: **"Integração automática"**; quando diário concluído e confirmado → planilha **"Permite digitar e confirmar"**. Exibição: nome do menu customizável ("Diário de classe"), títulos de seções customizáveis ("Conteúdo ministrado", "Tarefas"), diários exibidos para alunos "Cursando (Ativo)".

### 6.4 Outros cadastros parametrizadores
- **Feriado**: flags de efeito — `st_efeito_biblioteca`, `st_efeito_financeiro` (feriado desloca prazos de biblioteca/financeiro), por unidade.
- **Tipo de horário**: grade de até 8 slots por turno (M1–M8, T1–T8, N1–N8 com hora inicial/final) + `qtd_minutos_atraso` (tolerância de atraso).
- **Tipos de recebimento do caixa**: código de 3 letras (CAC, CAD, CHQ, CHP, DEP, DIN, MAN=Liquidação Manual, PIX), vínculo a cartão e a transferência/PIX, **qtde dias repasse** e **qtde dias intervalo** (prazo de liquidação por meio de pagamento).
- **Operadora de cartão**: conta de repasse + empresa + tipo de integração; `percentual_tarifa`/`valor_tarifa`, `qtd_parcelas`, `qtd_dias_repasse`.
- **Conta financeira**: banco/agência/dígitos, `permite_saldo_negativo`, `usa_conciliacao_bancaria` + `data_inicio_conciliacao`, `usa_ordem_pagamento`, `disponivel_retirada_caixa`, nº conta contábil para integração.
- **Forma de recebimento**: tipos Direto / Boleto bancário / Boleto integrado-Híbrido (Itaú API); flags de credenciamento (PIX, Bolecode Itaú, Itaú API, Kobana), `permite_pix`, carteira homologada; forma de recebimento prioritária do responsável (`st_forma_recebimento_responsavel_prioritario`, campo `forma_recebimento` no cadastro do responsável e `instrucao_boleto` individual).
- **SMTP**: contas remetentes por unidade (nome, e-mail, responder-para, com-cópia) — usadas pela régua de cobrança.
- **LGPD**: termo de consentimento e termo de uso de imagem com texto rico, flag Ativo e escopo fixo declarado: *"Matrícula online - Ficha de inscrição, Reserva de vaga, Captação de alunos, Inscrição em eventos, Filantropia (ficha socioeconômica e novos cadastros) e Solicitações de alteração de dados cadastrais"*; responsável guarda `data_hora_aceite_termo_uso_privacidade`.
- **Impedimentos por canal** (entidade `Unidade`): tipos de impedimento independentes para matrícula, agendamento, impressão de boleto web, recebimento no caixa, RM online e web (`tipo_impedimento_agendamento`, `tipo_impedimento_imprimir_boleto_web`, `tipo_impedimento_matricula`, `tipo_impedimento_recebimento_caixa`, `tipo_impedimento_rm_online`, `tipo_impedimento_web`) + `mensagem_impedimento` e `st_nao_exibir_relacao_impedimento` no portal.
- **isaac (Arco)**: `POSSUI_FORMA_RECEBIMENTO_ISAAC`, `REGISTRO_BOLETO_ISAAC_LAMBDA_AWS`, `st_permite_acesso_usuario_isaac`, `usuario_isaac`, `vinculo_isaac` em centro de resultado, `servicos_vinculados_isaac` no plano de pagamento, `possui_cobranca_isaac_ano_atual` no responsável; telas dedicadas: vincular serviços a produtos da Plataforma isaac, operadores Isaac, status de matrícula/plano de pagamento isaac, status de rematrícula isaac, fila de registro, pendências de registro de boleto.

---

## 7. Restrições e invariantes do domínio (20)

1. **Finalização de turmas é irreversível** (confirmado por aviso explícito).
2. **Fases calculadas/fórmula virtual nunca são editáveis em lote** — só fases digitadas.
3. **Aprovação = nota E frequência** quando a verificação por frequência está ativa; reprovação por frequência tem situação própria.
4. **Só situação "Cursando" permite assinatura eletrônica**; digitação de nota/falta só em Cursando, Aprovado e Reprovado.
5. **Aulas do diário devem cair dentro do período da fase de nota** (flag por diário) e respeitar mín/máx de aulas.
6. **Conceitos de uma série usam uma única unidade de medida** (nota OU percentual), fixada pelo primeiro conceito.
7. **Títulos em cobrança registrada** só podem ser negociados se parâmetro permitir; cancelamento de título faturado em exercício anterior idem.
8. **Liquidações com mais de 40% de pagamentos a menor ou títulos vencidos >180 dias são bloqueadas** salvo carência configurada.
9. **Data de bloqueio financeiro** impede operações retroativas exceto para operadores autorizados.
10. **Troca de turma propaga títulos** (parametrizado) e exige verificação de impedimento de matrícula; **inativação do aluno dispara solicitação de cancelamento dos títulos vinculados** (parametrizado, com opção de antecipar/não antecipar faturamento).
11. **Pagamento de pré-matrícula muda a situação do aluno automaticamente** (qualquer parcela ou todas, conforme parâmetro).
12. **Educacenso**: exportação bloqueada até preenchimento dos obrigatórios em todas as unidades; campos com tamanho/domínio fixados pelo layout INEP.
13. **Avisos OBS/FNC/COB/BIB** funcionam como badges de impedimento transversais nas telas de aluno/responsável.
14. **Dispensa de nota** zera a nota de composição (configuração do cliente) e usa textos fixos (F, 2CH, `*`) no boletim.
15. **O cálculo de nota usa truncamento com 1 casa decimal** (por unidade), e a situação final da disciplina componente segue a composta.
16. **Régua de cobrança roda 1×/dia às 14h** — não é tempo real; já o registro de boleto isaac é em tempo real com fallback de 10 min.
17. **Caixa**: numeração sequencial global; retiradas de cartão obrigatoriamente para a conta da operadora; cheques pré-datados valorizados pelo "bom-para".
18. **Relatórios contábeis ignoram serviços do tipo "negociação"** e cancelamentos só repercutem se posteriores ao mês de faturamento.
19. **Token de API de parceiro é por instituição** e exige `version=0`.
20. **CPF é exibido mascarado** nas telas de detalhe (LGPD); consentimento e uso de imagem têm escopo de módulos fixo declarado no próprio termo.

---

## Lacunas (telas capturadas como shell SPA, sem conteúdo renderizado)

`baixa_pagamento`, `matricula`, `matricula_online`, `confirmacao_matricula`, `aluno_segunda_via`, `nfse`/`nota_fiscal_servico`, `conciliacao_bancaria`, `exportacao_contabil`, `contratos_financeiros`, `situacao_contratos`, `reprovacao_por_faltas`, `reprovar_aluno_recuperacao`, telas isaac (`status_*_isaac`, `vincular_servicos_*`, `cancelar_titulos_isaac_lote`, `operadores_isaac`) — para essas, as regras acima vêm dos atributos de API no glossário e dos endpoints listados em `.janus/analyses/*.md`; o detalhe de UI exigiria recaptura com interação (login + clique).
