# 02 — Domínio Acadêmico

> Documento 02 de 10. O mais extenso: pessoas e acesso, estrutura acadêmica configurável, captação e matrícula, rotina diária infantil, frequência, motor de avaliação, boletim e histórico. IDs: `PES-xx` (pessoas), `ACA-xx` (acadêmico), `MAT-xx` (matrícula), `ROT-xx` (rotina/diário), `AVA-xx` (avaliação).
>
> Regra-mestra: tudo que a base faz aqui é MANTER; o que o repertório agrega entra como ACRESCENTAR/APRIMORAR; os conflitos C1, C3, C5, C6 aplicam a estratégia validada (a base é o caso default/seed do modelo configurável).

---

## 1. Pessoas e acesso

### PES-01 — Pessoa (registro único)
Cadastro único de qualquer indivíduo ou organização (PF/PJ), base de alunos, parentes, colaboradores e fornecedores (MANTER, base §3.1). Campos: nome, **nome social** (toggle "possui diferença entre nome social e civil" — ACRESCENTAR), apelido, sexo, **cor/raça** (censitário — ACRESCENTAR), nascimento, CPF/RG (PF) ou CNPJ/razão social/inscrições (PJ), e-mails, telefones (lista), naturalidade, nacionalidade, estado civil, avatar. Validação algorítmica de CPF/CNPJ; CPF/RG únicos por tenant. Endereço como entidade própria 1:N (CEP, logradouro, número, complemento, bairro, cidade, UF), com lookup de CEP (ViaCEP/BrasilAPI — ACRESCENTAR leve).

### PES-02 — Parente e atribuições
Parente vincula uma Pessoa a um ou mais alunos/prospects, com **parentesco por vínculo** (cadastro configurável) e **atribuições por aluno** (MANTER literal — base §2.2). Códigos de atribuição (seeds): diário (leitura), comunicados, diário completo (edição), financeiro, dados do aluno, renovação de matrícula (sazonal), fotos, fotos para não responsáveis, receber push. Adiciona-se o **papel do vínculo** (APRIMORAR, C4): responsável **financeiro** (pagador) × **pedagógico** (destinatário pedagógico) × responsável **legal** (acesso pleno, regra fixa).

### PES-03 — Responsável legal (regra fixa inviolável)
O responsável legal tem acesso pleno aos **dados do próprio filho** (diário, saúde, comunicados, financeiro do aluno), independentemente de atribuições, situação do aluno ou configuração de impedimento da escola (PIL-06). Esta regra **não é configurável**.

### PES-04 — Colaborador
Vincula Pessoa à instituição (MANTER, base §3.1): função (cadastro), unidade(s), admissão/demissão, jornada, salário base e vale-transporte (centavos), dados bancários para depósito, foto. Permissões em dois vetores: **unidades de acesso** e **módulos de acesso**; flags: assina contrato como representante, assina como testemunha, altera vencimento de boleto, visível para parentes. Acréscimos por segmento seriado: **formação acadêmica** (escolaridade), **disciplinas habilitadas**, **dados INEP/gestor Educacenso**, **quadro de disponibilidade** (slots por turno/dia — alimenta a criação automática de aulas, AVA). Importação em lote (PLT-11).

### PES-05 — Acesso derivado de vínculo
Dois vetores derivados, somados às permissões explícitas:
- **Responsável/parente**: acessa o portal/app porque tem vínculo com aluno elegível; o que vê é filtrado pelas atribuições (PES-02) — comportamento da base.
- **Professor** (segmentos seriados): o acesso ao portal do professor é **ativado pela alocação didática** turma×disciplina (ACRESCENTAR). Sem alocação, sem portal de professor.

### PES-06 — Autenticação
- **Web/Portais**: e-mail (ou CPF) + senha; recuperação por token temporal curto.
- **App/API**: token assinado com expiração, validado **antes** da execução de qualquer ação (RQ-SEG-01), em todas as requisições exceto endpoints explicitamente públicos (doc 06). Recuperação por código de 6 dígitos (validade 1h) por e-mail **ou SMS**.
- **Login social**: Google, Apple, Facebook convergindo ao mesmo fluxo de emissão de token (MANTER).
- **Provisionamento**: convite com definição de senha pelo usuário (RQ-SEG-03) — substitui a senha derivada de CPF; envio de convites/redefinições em massa (PLT-11).
- Troca de perfil (colaborador↔parente, e entre perfis acumulados) pelo seletor do AppHeader (doc 05).

---

## 2. Captação (comercial)

### ACA-01 — Prospect e funil
Cadastro de interessado (MANTER, base §3.6): pessoa, unidade/agrupamento pretendidos, ano pretendido, permanência/turno/horário desejados, primeiro atendimento, necessidades especiais, **"por nascer"** (gestação, diferencial infantil), meio de conhecimento/atendimento, colaborador responsável. Agrupamento calculado pela idade. Funil por status (novo, em contato, proposta, matriculado, desistência) com inativação motivada. KPIs de funil ganhas/perdidas no dashboard (APRIMORAR).

### ACA-02 — Interações e agendamento de visita
Interações com tipo, data/hora, anexos, próximo contato e status derivado (pendente/atrasada/concluída) (MANTER). Formulário público **"agende uma visita"** com e-mails automáticos de confirmação, alerta de duplicidade e de conversão (MANTER). Papel opcional de **vendedor de captação** com botão de WhatsApp do consultor (ACRESCENTAR leve — a interação por WhatsApp deve ser registrada no CRM). Indicações por famílias e acompanhamento sistemático preservados.

---

## 3. Estrutura acadêmica configurável (C1)

### ACA-03 — Princípio: estrutura montada pela escola
Cada escola configura sua estrutura acadêmica a partir de **templates** (PLT-04). Não há um modelo único imposto: o sistema oferece um modelo de domínio unificado cujas dimensões são ativadas conforme o segmento. **Nada da base é descontinuado** — o modelo infantil é um template de partida intocado.

### ACA-04 — Espinha dorsal unificada
```
Período letivo
└── Curso  (com flag oficial; modalidade; portaria)
    └── Nível estrutural  ←  configurável por escola, pode ser:
         • Agrupamento etário (meses) + Nível  → template Infantil
         • Série (ano escolar)                  → template Seriado
    └── Turma  (vínculo aluno via Enturmação)
        └── Disciplina via Grade Curricular  ← opcional, ativada no template seriado
```

Dimensões preservadas da base e disponíveis a qualquer template: **turno**, **permanência** (carga diária — integral/parcial), **horário** (janela de entrada/saída). Estas três são âncoras do modelo comercial de creche e **não existem no repertório seriado** — permanecem como dimensões próprias do SIGE.

### ACA-05 — Período letivo
Ano/período escolar (MANTER + APRIMORAR): sigla, nome, datas inicial/final, dias e semanas letivas, **somente um corrente por vez**, janela de renovação de matrícula. Encadeamento explícito `próximo período` (PIL-11). Dias letivos (com atividades especiais) e feriados parametrizados, com cálculo de dias úteis descontando fins de semana e feriados; feriados com **efeitos tipados** (financeiro etc.) e **cópia entre períodos** (APRIMORAR — resolve §12.7 da base).

### ACA-06 — Curso
Programa/segmento (MANTER + APRIMORAR): nome, código, modalidade, flag **oficial** (cursos livres/extracurriculares existem), portaria de autorização, coordenador responsável, **serviço financeiro** associado, código de integração. Agrupamentos/séries associados conforme o template.

### ACA-07 — Agrupamento etário e Nível (template infantil)
Agrupamento = faixa etária em meses (ex.: *Berçário 0–12m (exemplo)*) com ordenação, usado para **classificar automaticamente** aluno/prospect pela data de nascimento e unidade (MANTER literal, base §7.2). Nível = série/classe dentro do agrupamento. A progressão derivada da idade permanece como regra automática (PIL-11).

### ACA-08 — Série (template seriado)
Série pertence a curso (ACRESCENTAR por segmento): nome, código, **prefixo de matrícula**, **serviço financeiro padrão** (mensalidade da série), **próxima série** (promoção), código Educacenso, `permite matrícula repetida`, dependências/`quantidade de dependências` (progressão parcial — por segmento), situações default de resultado (FK para situações — §5), flags `utiliza avaliação por nota/meta/relatório`, `utiliza rotina de educação infantil`, `permitir saída sozinho`.

### ACA-09 — Turma
Grupo de alunos (MANTER + APRIMORAR): nome, sigla/código, sala, **vagas regulares e inclusivas separadas** (MANTER — diferencial de inclusão), turno, período, curso, agrupamento/série, tipo de horário, datas início/fim, colaboradores responsáveis/coordenador, serviço de mensalidade, modelo de contrato, tipos de refeição servidos (infantil). **Restrições de elegibilidade** (APRIMORAR): nascimento mín/máx, sexo, pré-requisito "exigir vínculo em outra turma" (extracurriculares), código INEP. **Modo de frequência** da turma (ACA-13). Regras estruturais da base preservadas: turma exige ≥1 colaborador e horário inicial < final; aluno deve pertencer a ≥1 turma. Ocupação projetada considera **trocas de turma aprovadas e ainda não implementadas** (MANTER) e separa vagas regulares de inclusivas.

### ACA-10 — Disciplina e Grade Curricular (template seriado)
Disciplina (ACRESCENTAR por segmento): nome, sigla, tipo (normal/composta), agrupamento de área (BNCC), nº de ordem, código MEC/Educacenso; disciplinas **compostas** (mãe + componentes) e a pseudo-disciplina de **frequência diária** (diário só de chamada). Grade Curricular = disciplina × série × período: carga anual/semanal (hora-relógio e hora-aula), `requer nota`, `usa conceito`, ordem no boletim, exibe em boletim/ata/histórico, conteúdo programático, disciplina alternativa. Itinerários formativos do Novo Ensino Médio por área de conhecimento (ACRESCENTAR por segmento).

### ACA-11 — Tipo de horário
Cadastro nomeado por segmento/turno definindo a malha de slots de aula (até 8 por turno: manhã/tarde/noite) e tolerância de atraso. Usado por turmas e pelo quadro de disponibilidade do professor (PES-04).

---

## 4. Matrícula, contrato e renovação

### MAT-01 — Enturmação (entidade central)
Vínculo aluno×turma×período (MANTER + APRIMORAR), reúne: matrícula (número por sequencial nomeado — PLT-21), datas de matrícula/efetivação/início, **situação do aluno** (§5), responsáveis do vínculo (principal e secundário, cada um com papel), dados financeiros (responsável financeiro, plano/forma, dia de vencimento), serviços contratados (obrigatórios/opcionais), tipos de refeição, turmas (uma por tipo de serviço). Acréscimos do repertório: planos de pagamento distintos para **pré-matrícula** e matrícula efetivada, **matrícula autorizada apesar de impedimento** com justificativa + autor (override auditado, PIL-07), situação de exceção auditada, progressão parcial (por segmento). Aluno deve pertencer a ≥1 turma (MANTER).

### MAT-02 — Atendimento e suas mudanças
"Atendimento" = conjunto acadêmico do aluno (unidade, curso, agrupamento/série, nível, turno, permanência, horário, turmas, serviços) (MANTER, glossário). Edição com **detecção de mudanças críticas** (confirmação ao alterar serviços), atualização dos valores financeiros do período e registro em auditoria. Histórico de atendimento ao longo do tempo preservado.

### MAT-03 — Captação → matrícula (jornada, wizard)
1. Prospect entra pelo formulário público ou é cadastrado pela equipe (ACA-02).
2. **Matrícula online self-service** (ACRESCENTAR): ficha de inscrição pública (form-builder por escola) e/ou RM online com etapas configuráveis (dados do aluno, responsáveis, ficha médica, seleção de plano, serviços adicionais), exibindo **termos LGPD** (consentimento + uso de imagem) com aceite datado (doc 04).
3. **Procedimentos de matrícula** (ACRESCENTAR): checklist documental por série/agrupamento × período (ex.: *carteira de vacinação, laudos, fotos (exemplo)*), com obrigatoriedade, prazo de entrega, upload pelo portal e flag de pendência (PDM) que alimenta impedimentos (doc 03).
4. Conversão: criação do aluno com atendimento completo, responsáveis e financeiro; logins de parentes criados por convite (PES-06).
5. **Contrato e efetivação** — MAT-04.

### MAT-04 — Efetivação por gatilho configurável (C3)
O **Requerimento** (MAT-06) é o mecanismo formal único de contrato. A condição de **efetivação da matrícula** é configurável por escola:
- **Assinatura** (default, comportamento da base): efetiva quando todos os signatários assinam o contrato.
- **Pagamento da pré-matrícula**: pagar parcela(s) do título de pré-matrícula muda a situação automaticamente (qualquer parcela ou todas — parâmetro).
- **Ambos**.

A **reserva de vaga paga** é um estado intermediário opcional, compatível com a cota CRV (doc 03). SLA de assinatura com alertas e cancelamento automático por prazo (parâmetro; *14 dias (exemplo)*) — APRIMORAR sobre o painel de contratos não assinados da base.

### MAT-05 — Contrato e assinatura eletrônica
Geração do contrato em PDF com dados do aluno, responsáveis, serviços, valores e signatários (MANTER). Assinatura via **conector abstrato** com Autentique **e** Clicksign homologados na v1 (doc 06, C8): múltiplos signatários (responsável, representante da escola, testemunhas), consulta de status/eventos, webhook, credencial por colaborador signatário. Verificação periódica e marcação automática de assinado.

### MAT-06 — Requerimentos (solicitações formais) — MANTER literal
Tipos (base §4.6): contrato de serviços, alteração de permanência, alteração de turno/horário, cancelamento de serviços, contratação de serviços, solicitação de documentos, alteração de turma, renovação de matrícula. Workflow de status canônico: **0 em aberto → 1 aprovado → 3 aguardando assinatura → 4 assinado → implementado**, com 2 negado e 5 cancelado/finalizado. Formulário dinâmico por tipo com dados específicos e **data de implementação** ("a partir de"). Aprovação/reprovação pela coordenação com motivo. **Implementação automática na data prevista** (rotina): aplica a mudança no atendimento e recalcula o financeiro (DF-2). Painéis: em aberto, aguardando assinatura, aguardando implementação, aprovados, reprovados.

### MAT-07 — Cancelamento de matrícula — MANTER literal
Motivos (seeds, configuráveis — PLT-19): fim de ciclo, solicitação do responsável, não renovação. Efeitos: cancela requerimentos pendentes, interrompe geração de títulos futuros, **mantém acesso do parente até a data efetiva**, reativável enquanto não finalizado. Estado modelado como propriedade da situação do aluno (§5, C6).

### MAT-08 — Renovação anual — MANTER + APRIMORAR
Abertura da janela de renovação do período. Responsáveis legais confirmam/recusam pelo app (atribuição sazonal) ou a equipe processa em lote. Rotina de renovação projeta o novo atendimento (curso/agrupamento/série seguintes via encadeamento PIL-11, serviços e valores do novo ano), alertando serviços inexistentes na nova turma; gera requerimento de renovação; matrículas não renovadas são canceladas por rotina. O processamento em lote usa o framework com simulação (PLT-11).

### MAT-09 — Promoção e finalização (segmentos seriados) — ACRESCENTAR
Promoção entre séries em lote (origem→destino, filtro por situação e por impedidos/não impedidos, situação de destino) e finalização de turmas (**irreversível**, com aviso explícito) via framework de lote. Inativação de enturmações em lote (ex.: limpar fichamentos vencidos).

---

## 5. Situação do aluno — meta-modelo (C6, PIL-04)

### MAT-10 — Cadastro de situações com seeds protegidos
"Situação do aluno na turma" é cadastro configurável por escola; cada situação mapeia para: **situação de sistema** (domínio fixo: seleção/pré-matrícula/ativo/inativo), **situação acadêmica** (aprovado/reprovado/cursando — usada em fórmulas), **situação Educacenso**, e flags de comportamento: digita nota/falta, permite assinatura eletrônica, **mantém acesso do parente**, **corta geração de cobrança**, **reativável**. Os comportamentos finos da base (cancelamento reativável, acesso até a data efetiva, interrupção de boletos futuros) são propriedades declarativas de situações **seed de sistema, não removíveis** (PIL-03). A escola pode criar situações adicionais; não pode quebrar as invariantes das seeds.

Seeds de partida (mapeamento à base): *Ativo/Cursando* (sistema=ativo, digita nota=sim), *Cancelado* (sistema=inativo, reativável até finalizar, corta cobrança), *Finalizado* (sistema=inativo, não reativável). Seeds adicionais para segmentos seriados: pré-matrícula, lista de espera, transferido, aprovado, reprovado, evadido.

---

## 6. Rotina diária infantil — MANTER literal (DF-1)

### ROT-01 — Diário de rotina
Agregador do dia do aluno (base §3.3), distinto do diário de classe (ACA/AVA). Reúne, por data: presença, refeições, sono, medicações, trocas de fralda, comunicados. Todos os registros identificam se ocorreram em **presença parcial**.

### ROT-02 — Registros da rotina
- **Refeição**: tipo + escala de consumo (0 recusou, 1 pouco, 2 metade, 3 bem, 4 muito bem) + observações. Tipos de refeição são cadastro (café, almoço, lanche, janta…). Conecta-se a restrições alimentares da ficha médica (PES — ROT-07).
- **Medicação**: prescrição do dia (medicamento, lista de horários, anexo de receita), registro de quem ministrou em cada horário; derivados ministração completa/parcial.
- **Troca de fralda**: horário + observações (inclui evacuação).
- **Repouso**: sono com horário inicial/final + observações.
- **Chamada/presença**: entrada e saída reais; saída com pessoa autorizada à retirada.

### ROT-03 — Operação de sala (fluxo, App)
Colaborador registra entrada da turma; ao longo do dia lança refeições, sono, medicações por horário, trocas de fralda, comunicados e fotos; registra saída. **Rotina noturna** fecha presenças sem saída com o horário padrão da turma. Responsáveis acompanham em tempo quase real, com notificações. Saída antes do fim do horário marca os registros do dia como **presença parcial** (regra crítica MANTER).

### ROT-07 — Ficha médica estruturada — ACRESCENTAR/APRIMORAR
Entidade 1:1 com o aluno (não existia consolidada na base): tipo sanguíneo, SUS, plano de saúde, médico e hospital de remoção, **alergias**, **restrições alimentares**, doenças, deficiências (com laudo), medicação específica, contato de emergência, remédio para febre — com campos-texto dependentes de flags. Editável pelo responsável (toggle de portal, doc 04). Alimenta a rotina de refeições e medicações (ROT-02) e o Educacenso (dados de deficiência).

---

## 7. Frequência (C5)

### ACA-13 — Dois modos, definidos pela escola
O **modo de frequência é propriedade da turma/segmento**, escolhido pela escola:
- **Presença diária** (template infantil — MANTER): entrada/saída reais, presença parcial, fechamento noturno (ROT-03).
- **Frequência por aula/disciplina** (template seriado — ACRESCENTAR): chamada por aula no diário de classe, com **falta justificada** e **dispensada** (parametrizáveis) e peso configurável da falta justificada na aprovação por frequência.

Ambos alimentam uma **apuração única de frequência** para relatórios e para a regra de aprovação por frequência (AVA-04). Uma escola pode operar os dois modos em turmas diferentes; a regra que vale para aprovação é a do modo da turma.

---

## 8. Motor de avaliação (segmentos seriados — ACRESCENTAR) (AVA)

> Núcleo pedagógico do repertório. **Não hardcodar bimestres**; valores citados são exemplos.

### AVA-01 — Sistema de avaliação
Conjunto de **fases de nota** aplicado a séries em um período, com estados Ativo/Bloqueado e validação (ação "validar o sistema"). Pode ser sobrescrito por série em "definições de boletim por série".

### AVA-02 — Fase de nota
Etapa avaliativa (bimestre, recuperação, média, total, exame): nome, sigla, nº de ordem de exibição (≠ ordem interna), tipo (digitada / informada por fórmula de composição / calculada / fórmula virtual — as duas últimas não editáveis em lote), valor máximo, datas (início/fim das aulas, **limite de digitação**, início de exibição no portal), semanas e dias letivos, **dispensa automática** (quando a fórmula tem uma única nota de composição), flags de boletim.

### AVA-03 — Fórmulas de composição
Expressões configuráveis (DSL própria) sobre notas de composição calculam médias, faltas e aprovação, com **roteamento condicional de fase** (se aprovado → fase X / situação Y; se reprovado → recuperação / situação Z). Parâmetros de cálculo por unidade: casas decimais, **truncar vs arredondar**, textos para dispensado/2ª chamada/sem nota, tratamento de dispensa e de disciplinas compostas. Fórmulas usadas no cálculo são **versionadas/snapshot** para reprodutibilidade.

### AVA-04 — Aprovação por nota E frequência (invariante)
Quando a verificação por frequência está ativa, o aluno só é aprovado se atender **nota E frequência**; reprovação por frequência tem situação própria (FK a MAT-10).

### AVA-05 — Conceitos e métodos alternativos
- **Conceitos**: escala qualitativa por série, com **unidade única** (nota OU percentual) fixada pelo primeiro conceito (invariante).
- **Avaliação por relatório (descritiva)**: por turma/fase/disciplina, com confirmação do professor — **aderente ao perfil infantil/Montessori**, primeiro degrau pedagógico antes de notas.
- **Avaliação por competência**: índices, tipos, fases, planilha e relatório configurável.
- **Avaliação institucional**: pesquisa família→escola (canal família→escola, doc 04).

### AVA-06 — Diário de classe
Diário = turma × disciplina × fase (ACRESCENTAR por segmento): aulas, conteúdo, tarefas, chamada (ACA-13). Roster com data de entrada/saída do aluno no diário (suporta transferências no meio da fase). ~25 parâmetros por escola (datas, aulas em feriados, criação automática de aulas pelo quadro de horário, exigência de conteúdo/tarefas, integração com a planilha de notas). Situação do diário: disponível → concluído → confirmado.

### AVA-07 — Lançamento, confirmação e fila
Planilha de notas e faltas por período×curso×série×turma×disciplina×fase×situação; colunas configuráveis, aplicação em lote, importação de padrão de avaliação. **Fluxo de confirmação** professor → coordenação, com processamento **assíncrono em fila** (PLT-12), reprocessamento controlado, gate "bloquear boletim com impedimento" e permissões do professor (confirmar/desconfirmar, solicitar confirmação, alterar fórmula — parametrizável). Alteração de notas em lote com **simulação** (PLT-11).

### AVA-08 — Boletim
Configurável por sistema/série (ACRESCENTAR por segmento): dezenas de opções de exibição (notas por fase, faltas, frequência %, foto, filiação, disciplinas dispensadas, médias de componentes, agrupamento BNCC, cores por nota, ranking — *ranking fora da v1*, progressão parcial), assinaturas digitalizadas, textos personalizados, **protocolo de recebimento pelo responsável**. **Regras de publicação** explícitas (data de início de exibição, bloqueio por impedimento) — gate de qualidade (PIL-05).

### AVA-09 — Histórico escolar
Geração automática **ao concluir a turma** (ACRESCENTAR por segmento): cálculo de carga horária parametrizado, inclusão/exclusão de disciplinas dispensadas/inaptas, cursos com impressão liberada por unidade, totais por ano (dias letivos, faltas, média global, frequência), suporte a histórico de escola de origem. Edição controlada de notas no histórico conforme parâmetro do período.

---

## 9. Hub 360° do aluno (padrão de UI — detalhe no doc 05)

A secretaria opera **centrada no aluno**: um registro, múltiplas abas — cadastro, ficha médica, turmas/enturmação, ocorrências, histórico, descontos, cobranças, diário. Padrão de navegação preservado e elevado a requisito (era o único fluxo consistente da referência). Vínculo N:N com responsáveis com papel financeiro destacado.

---

## 10. Invariantes do domínio acadêmico

1. Apenas um período letivo corrente por vez (MANTER).
2. No template infantil, o agrupamento do aluno/prospect deriva da data de nascimento e da unidade (MANTER).
3. Aluno pertence a ≥1 turma; turma tem ≥1 colaborador e horário inicial < final (MANTER).
4. Ocupação de turma separa vagas regulares de inclusivas e considera trocas aprovadas não implementadas (MANTER).
5. Saída antes do fim do horário marca os registros do dia como presença parcial (MANTER).
6. O modo de frequência é definido pela turma; a aprovação por frequência usa o modo da turma (C5).
7. Aprovação = nota E frequência quando a verificação por frequência está ativa (AVA-04).
8. Conceitos de uma série usam unidade única, fixada pelo primeiro conceito (AVA-05).
9. Fases calculadas/fórmula virtual não são editáveis em lote (AVA-02).
10. Finalização de turmas é irreversível (MAT-09).
11. Situações seed de sistema não são removíveis; só elas carregam as semânticas críticas da base (C6).
12. Implementação de requerimento ocorre automaticamente na data prevista, com recálculo financeiro (DF-2).
13. O responsável legal nunca perde acesso aos dados do próprio filho (PES-03, regra fixa).
14. Boletim só publica conforme as regras de publicação e o gate de impedimento configurados (AVA-08).
15. Promoção/renovação em lote passam por simulação antes de efetivar (PLT-11).
