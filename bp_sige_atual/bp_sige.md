# BP — SIGE (Sistema Integrado de Gestão Escolar)

> **Blueprint funcional do sistema.** Este documento descreve O QUE o SIGE faz — domínio, regras de negócio, fluxos, integrações e requisitos — de forma independente de qualquer tecnologia de implementação. Ele serve de base para a elaboração do Documento de Especificação de Software (SSD) de uma nova construção do sistema.

---

## 1. Visão Geral

O SIGE é um sistema de gestão para instituição de ensino infantil (multiunidade), atualmente operado pela escola **Aldeia Montessori**. Ele cobre o ciclo completo de vida do aluno e da operação escolar:

- **Captação**: gestão de interessados (prospects), visitas, interações comerciais e conversão em matrícula.
- **Acadêmico**: matrícula, turmas, frequência, diário do aluno (rotina diária: refeições, sono, medicações, higiene).
- **Financeiro**: cobrança por boletos bancários, remessa/retorno bancário, juros/multa/mora, notas fiscais de serviço (NFS-e) e plano de contas.
- **Contratual**: requerimentos formais com fluxo de aprovação e assinatura digital de contratos.
- **Comunicação**: circulares, comunicados, ocorrências, fotos, notificações internas, e-mails e notificações push.
- **Administrativo**: colaboradores, unidades, serviços e preços, anos letivos, calendário escolar, documentos institucionais.

O sistema possui **duas frentes de uso**:

1. **Aplicação web administrativa** — usada pela secretaria, coordenação, direção e colaboradores.
2. **API para aplicativo móvel** — usada por responsáveis (pais/parentes) e colaboradores em campo (professores/cuidadores).

---

## 2. Atores e Perfis de Acesso

### 2.1 Tipos de usuário

| Ator | Descrição | Canal principal |
|---|---|---|
| **Administrador** | Acesso completo a todos os módulos e unidades. | Web |
| **Colaborador** | Professor, auxiliar, coordenador, diretor, administrativo. Acesso restrito por unidade e por módulo. | Web + App |
| **Responsável legal** | Pai/mãe/tutor responsável pelo aluno. Acesso irrestrito aos dados dos seus alunos. | App |
| **Parente** | Outros familiares/contatos do aluno, com permissões granulares por aluno. | App |
| **Prospect** | Interessado externo (sem login), interage por formulário público de agendamento de visita. | Web pública |

### 2.2 Modelo de autorização

- Toda pessoa do sistema possui um cadastro único de **Pessoa** (dados pessoais) e, opcionalmente, uma credencial de **Login** (e-mail + senha).
- **Colaboradores** têm dois vetores de permissão: *unidades de acesso* (em quais unidades atuam) e *módulos de acesso* (quais módulos podem usar). Flags adicionais: pode assinar contratos como representante; pode assinar como testemunha; pode alterar vencimento de boletos; é visível para parentes.
- **Parentes** têm **atribuições por aluno** — um conjunto de permissões concedido individualmente para cada aluno vinculado:

| Código | Atribuição |
|---|---|
| 0 | Diário (leitura) |
| 1 | Comunicados |
| 2 | Diário completo (edição) |
| 3 | Financeiro |
| 4 | Dados do aluno |
| 5 | Renovação de matrícula (sazonal) |
| 6 | Fotos |
| 7 | Fotos (permissão especial para não responsáveis) |
| 8 | Receber notificações push |

- O **responsável legal** de um aluno tem acesso irrestrito a ele, independentemente das atribuições.
- Um colaborador que também é parente pode **alternar de perfil** dentro do aplicativo.

### 2.3 Autenticação

- **Web**: e-mail (ou CPF) + senha; recuperação de senha por token temporal enviado por e-mail (validade curta, ~10 minutos).
- **App/API**: token assinado (com expiração) emitido após login; enviado em cabeçalho de autorização e validado em todas as requisições, exceto endpoints públicos (login, recuperação de senha, verificação de versão).
- **Login social**: Google, Apple e Facebook — após validação no provedor, convergem para o mesmo fluxo de emissão de token.
- **Recuperação de senha no app**: código numérico de 6 dígitos com validade de 1 hora, enviado por e-mail **ou SMS** (integração com provedor de SMS).
- Na criação de login de parente, o sistema gera senha padrão derivada de dados pessoais e a envia por e-mail (regra a ser revisada na nova construção — ver §12).
- O sistema atual contém uma **senha-mestre global** que autoriza login com qualquer e-mail; é um risco de segurança e **não deve ser replicada** (ver §12).

---

## 3. Modelo de Domínio

O domínio se organiza em 8 subdomínios, com ~67 entidades. Abaixo, as entidades por subdomínio com seus papéis e relacionamentos essenciais. Valores monetários são armazenados como **inteiros em centavos** em todo o sistema.

### 3.1 Pessoas

| Entidade | Papel |
|---|---|
| **Pessoa** | Registro único de qualquer indivíduo ou empresa: nome, apelido, sexo, nascimento, CPF/RG (PF) ou CNPJ/razão social/inscrições (PJ), e-mails, telefones (lista), naturalidade, avatar. Validação algorítmica de CPF/CNPJ; CPF/RG únicos. |
| **Parente** | Vincula uma Pessoa a um ou mais alunos e prospects, com o tipo de parentesco por vínculo e as atribuições (permissões) por aluno. |
| **Parentesco** | Tipo do vínculo (pai, mãe, avó, tutor…). |
| **Colaborador** | Vincula Pessoa à instituição: função, unidade, datas de admissão/demissão, jornada (entrada/saída/intervalo), salário base e vale-transporte (centavos), flags de assinatura de contrato e de permissões. |
| **Função de Colaborador** | Cargo (professor, auxiliar, coordenador, diretor…). |
| **Endereço** | Endereços de uma Pessoa (CEP, logradouro, número, complemento, bairro, cidade, UF). |
| **Estado / Nacionalidade / Estado Civil** | Tabelas de referência. |
| **Login** | Credencial de acesso (e-mail único, hash de senha, token de API, atalhos personalizados de menu, flag ativo). |
| **Token de Recuperação** | Código de recuperação de senha com expiração e marcação de uso. |

Relações: Pessoa 1—0..1 Aluno; Pessoa 1—0..1 Colaborador; Pessoa 1—0..1 Parente; Pessoa 1—N Endereço; Pessoa 1—0..1 Login. Parente N—N Aluno e N—N Prospect (com parentesco e atribuições por vínculo).

### 3.2 Acadêmico

| Entidade | Papel |
|---|---|
| **Aluno** | Matrícula de uma Pessoa em um ano letivo: número de matrícula, datas de matrícula e início, unidade, curso, agrupamento, nível, turno, permanência, horário, turmas (uma por tipo de serviço), serviços contratados, dados financeiros (responsável financeiro, forma de pagamento, dia de vencimento 1–28), tipos de refeição, parentes vinculados, responsável legal. |
| **Turma** | Grupo de alunos por serviço/unidade/ano letivo: nome, vagas (regulares e inclusivas), horários inicial/final, dias da semana, colaboradores responsáveis, tipos de refeição servidos. |
| **Curso** | Programa educacional (ex.: Educação Infantil), com agrupamentos associados. |
| **Agrupamento** | Faixa etária em meses (ex.: Berçário 0–12m), com ordenação; usado para classificar automaticamente aluno/prospect pela data de nascimento. |
| **Nível** | Série/classe dentro de um agrupamento (ex.: Pré I, Pré II). |
| **Turno** | Período (matutino, vespertino, integral). |
| **Permanência** | Carga de permanência diária (integral, parcial; horas/dia). |
| **Horário** | Janela padrão de entrada/saída do aluno. |
| **Presença** | Registro de frequência diária: presença/ausência, horário previsto, entrada e saída reais. Saída anterior ao fim do horário caracteriza **presença parcial**. |
| **Diário** | Agregador do dia de um aluno: reúne presença, comunicados, refeições, medicações, trocas de fralda e repousos da data. |
| **Cancelamento de Matrícula** | Registro de cancelamento com motivo, data, flags ativo/finalizado (permite reativação enquanto não finalizado). |
| **Registro de Atendimento** | Histórico de configuração de atendimento (horário) do aluno ao longo do tempo. |

Regras estruturais: aluno deve pertencer a pelo menos uma turma; turma exige ao menos um colaborador e horário inicial < final.

### 3.3 Rotina diária e cuidados (creche)

| Entidade | Papel |
|---|---|
| **Refeição** | Consumo de uma refeição no dia (tipo + escala de consumo: recusou / pouco / metade / bem / muito bem + observações). |
| **Tipo de Refeição** | Café, almoço, lanche, janta… |
| **Medicação** | Prescrição do dia: medicamento, lista de horários, registro de quem ministrou em cada horário, anexo de receita médica. Derivados: ministração completa/parcial. |
| **Troca de Fralda** | Horário + observações (inclui registros de evacuação). |
| **Repouso** | Sono/descanso com horário inicial/final + observações. |
| **Agendamento** | Agendamento de visita/reunião com data, hora, pessoa e colaborador responsável. |

Todos os registros do diário identificam se ocorreram durante **presença parcial**.

### 3.4 Financeiro

| Entidade | Papel |
|---|---|
| **Boleto** | Cobrança emitida contra o responsável financeiro: números interno e bancário, vencimento, valor sem desconto, valor com desconto, valor líquido recebido, data de liquidação, banco emissor, referência (mês/ano), tipo de cota, data da última notificação de vencimento, flag de exclusão. |
| **Serviço** | Oferta comercializável: combinação de unidade + tipo de serviço + curso + agrupamento + nível + turno + permanência + horário; obrigatório ou opcional; ordenável. |
| **Tipo de Serviço (auxiliar)** | Catálogo dos tipos (escolaridade, hotelaria/extensão, atividades extras etc.). |
| **Valor** | Preço de um serviço com vigência (data início/fim); múltiplas vigências por serviço; o sistema resolve o **valor vigente** pela data. |
| **Movimento** | Lançamento contábil de receita/despesa (plano de contas): tipo, descrição, valor, data, classificação de custo, pessoa vinculada, vínculo opcional com boleto e data de pagamento. |
| **Classificação de Custo** | Centro/classificação contábil (nome + código). |
| **Fornecedor** | Pessoa (geralmente PJ) fornecedora de produtos/serviços. |
| **Detalhe de Remessa** | Item de arquivo de remessa bancária vinculando boleto à remessa. |
| **RPS / Lote de RPS** | Recibo Provisório de Serviços e seu lote de envio à prefeitura (conteúdo do protocolo fiscal, status do processamento, numeração sequencial). |
| **Nota Fiscal / Registro de Notas Fiscais** | NFS-e emitida (número, data de emissão, vínculo com RPS e aluno) e o controle mensal de emissão por unidade. |
| **Registro de Pagamento** | Pagamento efetivado de um boleto (data, valor). |
| **Controle de Contratos** | Vigência de contratos de trabalho dos colaboradores. |

### 3.5 Comunicação

| Entidade | Papel |
|---|---|
| **Comunicado** | Recado/anotação do diário entre escola e família: mensagem, autor (colaborador ou parente), menções a pessoas, destinatários, controle de leitura por destinatário, status (ativo / pendente de aprovação / excluído), flag observação. |
| **Circular** | Comunicado formal individualizado por aluno/destinatário, derivado de um lote, com controle de leitura (lido / lido em). |
| **Lote de Circular** | Emissão em massa: título, mensagem, arquivo anexo, turmas destinatárias, data de envio. |
| **Ocorrência** | Registro de incidente/observação institucional: autor, texto com menções e tags, controle de visualização, comentários encadeados (respostas). |
| **Notificação** | Notificação interna ("sino"): destinatário, tipo, conteúdo, metadados de vínculo (ex.: aluno + boleto), lida/removida. Tipos: 3=circular, 4=financeiro, 5=comunicado, 6=foto, 7=push customizada. |
| **Notificação Push / Lote de Push** | Campanhas push customizadas pelo administrador, individualizadas por destinatário. |
| **Foto / Compartilhamento de Foto** | Foto de evento/rotina com descrição, alunos e colaboradores envolvidos, fluxo de **dupla aprovação** e registro de quem pode visualizar/compartilhamentos. |
| **Documento / Capítulo** | Biblioteca institucional de documentos organizados em capítulos/seções ordenáveis. |

### 3.6 Comercial (captação)

| Entidade | Papel |
|---|---|
| **Prospect** | Interessado/pré-matrícula: pessoa, unidade pretendida, permanência/turno/horário desejados, data do primeiro atendimento, necessidades especiais, "por nascer" (gestação), agrupamento calculado pela idade. |
| **Interação** | Contato realizado/agendado com o prospect: tipo, título, data/hora, colaborador responsável, conclusão e resultado. Status derivado: concluída / atrasada / pendente. |
| **Tipo de Interação** | Ligação, e-mail, visita, reunião… |
| **Inativação de Prospect** | Desistência/inativação com motivo, data e flags ativo/finalizado. |
| **Meio de Atendimento / Meio de Conhecimento** | Canal do primeiro contato e como conheceu a escola. |
| **Acompanhamento Sistemático** | Acompanhamento pedagógico (ex.: necessidades especiais) com data e observações. |
| **Indicação** | Indicações de novos alunos por famílias. |

### 3.7 Estrutura administrativa

| Entidade | Papel |
|---|---|
| **Unidade** | Local físico da instituição; nome único; agrupamentos disponíveis; pode **estender** outra unidade (compartilhando estrutura). |
| **Ano Letivo** | Ano escolar; somente um marcado como corrente; janela de renovação de matrícula (início/fim). |
| **Dia Letivo** | Dias com aula no calendário do ano, incluindo dias de atividades especiais. |
| **Feriado** | Feriados e pontos facultativos (impactam cálculo de vencimentos e dias letivos). |
| **Requerimento** | Solicitação formal sobre um aluno (ver §6.3): tipo, dados específicos, data de implementação, status, identificador do documento de assinatura. |
| **Log** | Auditoria: usuário, ação, entidade afetada, registro, descrição. |
| **Cor** | Categorização visual (nome + código hexadecimal). |

---

## 4. Módulos Funcionais — Aplicação Web

### 4.1 Dashboard
- Indicadores gerais: totais de prospects, matrículas e cancelamentos no período.
- Gráficos por unidade e período, com filtro de data inicial/final.

### 4.2 Alunos
- Listagem com filtros (ativos/cancelados), ordenação e busca.
- Cadastro completo: dados pessoais, atendimento (unidade, curso, agrupamento, nível, turno, permanência, horário), turmas, serviços obrigatórios e opcionais, responsáveis/parentes com parentescos e atribuições, dados financeiros (responsável financeiro, dia de vencimento).
- Busca de parente por CPF (com/sem máscara) para reaproveitar cadastro; validação de CPF duplicado.
- Criação automática de login para parentes (com envio de senha por e-mail).
- Edição de atendimento com detecção de mudanças críticas (confirmação ao alterar serviços), atualização dos valores financeiros do período e registro em log de auditoria.
- Cancelamento de matrícula com 3 motivos (fim de ciclo / solicitação do responsável / não renovação), efeitos: cancela requerimentos pendentes, interrompe geração de boletos futuros, mantém acesso do parente até a data efetiva; reativável enquanto não finalizado.
- Relatórios: alunos por unidade com nascimento; pais/mães por faixa etária e agrupamento; exportação CSV.

### 4.3 Turmas
- Listagem por ano letivo com ocupação (matriculados × vagas), separando vagas regulares e inclusivas; considera requerimentos de troca de turma aprovados ao projetar ocupação.
- Criação/edição de turma (nome, serviço, vagas, horários, dias da semana, colaboradores).
- Histórico de turmas por período; reordenação manual.
- Lista de alunos por turma; importação de alunos em lote.
- Detecção de **alunos soltos** (sem atendimento no ano corrente).
- Painel de **contratos não assinados** com consulta de status na plataforma de assinatura.
- Registro de frequência integrado ao calendário escolar.

### 4.4 Serviços e Preços
- Catálogo de serviços por combinação de atributos (unidade/curso/agrupamento/nível/turno/permanência/horário), com obrigatoriedade e ordenação.
- Tabela de valores com vigência por período; múltiplos preços históricos por serviço; edição em lote.

### 4.5 Financeiro
- **Geração de boletos** por aluno/serviço com três tipos de cota: Cota Mensal de Anuidade Escolar (CMAE), Cota de Composição (CC) e Cota de Reserva de Vaga (CRV); cálculo de valores com/sem desconto.
- **Remessa bancária**: geração de arquivo no padrão de intercâmbio bancário (CNAB) para dois bancos (Santander e Sicoob), com operações de entrada de título e pedido de baixa; sequenciamento de lotes; notificação push aos responsáveis após emissão.
- **Retorno bancário**: importação de arquivos de retorno, identificação de pagamentos e baixa automática (data de liquidação + valor recebido). Baixa manual individual por código de barras (validação de 44 ou 47 dígitos).
- **Controle de cobrança**: painel de boletos por período/unidade/aluno; disparo de comunicado de cobrança por e-mail aos responsáveis (lista de boletos vencidos com valores atualizados).
- **Visualização/impressão de boleto** com valores finais, instruções e código de barras (HTML e PDF).
- **Notas fiscais (NFS-e)**: geração de RPS por período, simulação prévia, envio de lotes à prefeitura, consulta de status, lista de notas emitidas, controle mensal por unidade e vínculo de notas a alunos.
- **Plano de contas**: lançamentos de receitas/despesas com classificação de custo, vínculo a pessoas/fornecedores e conciliação com boletos.
- Cadastro de fornecedores.

### 4.6 Requerimentos (solicitações formais)
- Tipos: contrato de serviços; alteração de permanência; alteração de turno/horário; cancelamento de serviços; contratação de serviços; solicitação de documentos; alteração de turma; renovação de matrícula.
- Fluxo de status: **0 Em aberto → 1 Aprovado → 3 Aguardando assinatura → 4 Assinado → implementado** (com 2 = Negado e 5 = Cancelado/Finalizado).
- Formulário dinâmico por tipo, com dados específicos (nova turma, novo horário, novos serviços) e **data de implementação** ("a partir de").
- Aprovação/reprovação pela coordenação com registro de motivo.
- Para tipos contratuais: geração do contrato em PDF, envio à plataforma de assinatura digital, acompanhamento dos signatários (responsável, representante da escola, testemunhas) e marcação automática como assinado quando todos concluírem.
- **Implementação automática** na data prevista: atualização do atendimento do aluno (turma/horário/serviços/permanência) e recálculo financeiro quando aplicável.
- Painéis: em aberto, aguardando assinatura, aguardando implementação, aprovados, reprovados; relatório geral.

### 4.7 Prospects (comercial)
- Cadastro de interessado com dados pessoais, unidade/agrupamento pretendidos, ano pretendido, meio de conhecimento/atendimento e colaborador responsável.
- Formulário público de **agendamento de visita** ("agende uma visita") com e-mails automáticos de confirmação, alerta de duplicidade e de conversão.
- Funil por status (novo, em contato, proposta, matriculado, desistência); inativação com motivo.
- Interações: registro de contatos com data/hora, anexos e próximo contato; status derivado (pendente/atrasada/concluída).
- Gráficos de prospects por unidade/período.

### 4.8 Anos Letivos e Calendário
- Cadastro de ano letivo; marcação única de ano corrente; criação do ano seguinte.
- **Renovação de matrículas em lote**: seleção dos alunos aptos (ativos, não cancelados), projeção do atendimento para o ano seguinte (mesmo curso/agrupamento, novos serviços e valores), seleção de turmas futuras e geração de requerimento de renovação.
- Dias letivos por ano, com atividades especiais; cálculo de dias úteis descontando fins de semana e feriados.

### 4.9 Dia a Dia (diário institucional)
- Linha do tempo de comunicados e fotos por unidade/aluno/colaborador, com filtros por data, texto e hashtag; abas geral/pendente/excluído com contagens; paginação.
- Comunicado com menções (@pessoa, #tag), anexos, vínculo a aluno (ou observação geral) e marcação de presença/ausência.
- Fotos com **dupla aprovação** (duas aprovações independentes ativam a publicação) e compartilhamento com parentes.
- Visibilidade restrita às unidades de acesso do usuário; parentes veem apenas seus filhos.

### 4.10 Comunicados e Ocorrências
- Comunicados com título, mensagem, anexo, turmas/alunos destinatários, menções, fluxo de aprovação e lixeira; confirmação de leitura por parente; e-mail automático aos destinatários.
- Ocorrências com comentários encadeados, menções, controle de visualização e pesquisa avançada.

### 4.11 Circulares
- Criação de circular (título, mensagem, arquivo) com seleção de turmas destinatárias e remetentes.
- Envio individualizado por aluno/responsável, com rastreamento de entrega e **confirmação de leitura**; reenvio aos que não confirmaram; e-mail automático.

### 4.12 Fotos
- Upload com descrição, alunos e colaboradores envolvidos; fluxo de dupla aprovação; status ativo/pendente/excluído; busca, contagens e paginação; compartilhamento seletivo com parentes.

### 4.13 Notificações
- Centro de notificações internas com tipos, metadados de vínculo e status de leitura.
- **Campanhas push** criadas pelo administrador: seleção de alunos/turmas, título e texto, filtro opcional por atribuição do parente; envio em lote, registro individual por destinatário e espelhamento como notificação interna.

### 4.14 Colaboradores
- Cadastro vinculado a Pessoa: função, unidade(s), admissão/demissão, jornada, salário, banco para depósito, foto.
- Permissões: módulos e unidades de acesso, assinatura como representante/testemunha, alteração de vencimento de boletos, visibilidade para parentes, credencial na plataforma de assinatura digital.
- Importação em lote via arquivo CSV.

### 4.15 Documentos
- Biblioteca de documentos institucionais organizados em capítulos ordenáveis, com visualização autenticada e exclusão lógica.

### 4.16 Configuração
- Parametrização administrativa por módulo (feriados, datas especiais, valores padrão).

---

## 5. API / Aplicativo Móvel

### 5.1 Capacidades para responsáveis (pais/parentes)
- Login (tradicional e social), recuperação de senha por e-mail/SMS, troca de perfil, atalhos personalizados de menu.
- Lista de alunos vinculados, contextualizada por permissão (diário, circulares, financeiro, dados, fotos).
- **Diário do filho**: presença/frequência por período, refeições, sono, medicações, trocas de fralda, comunicados do dia; inserção de anotações pelo responsável.
- **Financeiro**: boletos do aluno, detalhe, linha digitável e PDF do boleto; parcelas de cotas.
- **Circulares**: lista por aluno, última circular, leitura do arquivo e confirmação de leitura.
- **Fotos**: galeria por aluno (conforme permissão), registro de visualização/compartilhamento.
- **Renovação de matrícula** (na janela sazonal): solicitar, recusar e consultar status — restrito ao responsável legal ou parente com atribuição específica.
- Dados pessoais: consulta/edição de cadastro, endereço, telefones e avatar.
- Notificações: listagem, contagem de não lidas, marcação de leitura/remoção.
- Calendário: feriados, dias letivos, próximas aulas.

### 5.2 Capacidades para colaboradores (operação em sala)
- Turmas do colaborador; alunos por turma e por contexto (chamada, refeição, sono, evacuação, medicação).
- Registro da rotina: presença/ausência, entrada e saída, refeições com escala de consumo, repouso, medicações (prescrever e ministrar por horário), trocas de fralda, saída do aluno com pessoa autorizada.
- Comunicados: criar, editar status, excluir; menções a alunos e colaboradores; destinatários por aluno.
- Fotos: upload, fila de aprovação, lixeira, alteração de status.
- Permissões/módulos acessíveis ao colaborador; dados pessoais próprios.

### 5.3 Contratos da API
- Token de acesso com expiração transportado em cabeçalho de autorização; validação centralizada com resposta padronizada de erro (código + mensagem) para token inválido.
- Resposta padrão: `{ status, message, data }`.
- **Verificação de versão mínima do aplicativo** por plataforma (bloqueio de versões antigas).
- Duas gerações de API convivem (legada e atual); a nova construção deve unificar os contratos preservando as capacidades.
- **Deep links** configurados para Android e iOS (arquivos de associação de domínio publicados em endpoint conhecido), usados nos retornos de login social.

---

## 6. Fluxos de Negócio Principais

### 6.1 Captação → Matrícula
1. Interessado entra pelo formulário público de visita ou é cadastrado pela equipe (prospect).
2. E-mails automáticos: confirmação ao interessado; alerta interno em caso de duplicidade.
3. Equipe registra interações (ligações, visitas, propostas) com agendamento de próximos contatos.
4. Conversão: criação do cadastro de aluno com atendimento completo, serviços, responsáveis e financeiro.
5. Logins de parentes são criados automaticamente; senha enviada por e-mail.
6. Contrato de serviços é gerado (requerimento contratual) e enviado para assinatura digital.
7. Após assinatura de todos os signatários, a matrícula se efetiva; turmas são atribuídas e a cobrança inicia.

### 6.2 Cobrança (ciclo mensal)
1. Boletos são gerados para os alunos ativos conforme serviços contratados, dia de vencimento do aluno e valores vigentes.
2. Arquivo de remessa é gerado por banco e enviado; responsáveis recebem push de novo boleto.
3. Banco devolve arquivo de retorno → importação → baixa automática dos pagamentos.
4. Rotina diária notifica responsáveis (push + notificação interna): boleto vencendo hoje; boleto vencido (reaviso a cada 3 dias, controlado por data da última notificação).
5. Painel de cobrança permite disparo de e-mail consolidado de débitos com valores atualizados (juros/multa/mora).
6. Rotinas de conciliação vinculam pagamentos a lançamentos contábeis e fazem prova real entre boletos pagos e movimentos.

### 6.3 Alteração contratual via Requerimento
1. Solicitação registrada (alteração de turma/horário/permanência/serviços, cancelamento, documentos).
2. Coordenação aprova ou nega (com motivo).
3. Se contratual: contrato em PDF → assinatura digital pelo responsável, representante da escola e testemunhas → verificação periódica automática do status → marcado como assinado.
4. Na data de implementação, rotina automática aplica a mudança no atendimento do aluno e recalcula o financeiro.

### 6.4 Renovação anual de matrícula
1. Administração abre a janela de renovação do ano letivo.
2. Responsáveis legais confirmam ou recusam a renovação pelo aplicativo (ou a equipe processa em lote).
3. Rotina de renovação projeta o novo atendimento (curso/agrupamento seguintes, serviços e valores do novo ano), alertando quando um serviço não existe para a nova turma.
4. Novo contrato é gerado e segue o fluxo de assinatura; matrículas não renovadas são canceladas por rotina própria.

### 6.5 Rotina diária do aluno (operação de creche)
1. Colaborador registra entrada (chamada) dos alunos da turma.
2. Ao longo do dia: refeições (com escala de consumo), sono, medicações ministradas por horário, trocas de fralda, comunicados e fotos.
3. Saída registrada com horário (e responsável pela retirada); rotina noturna fecha presenças sem saída usando o horário padrão da turma.
4. Responsáveis acompanham tudo em tempo quase real pelo aplicativo, com notificações.
5. Saída antes do fim do horário marca os registros do dia como ocorridos em "presença parcial".

### 6.6 Comunicação institucional
1. Conteúdo criado (comunicado/foto/circular) → fluxo de aprovação (dupla aprovação para fotos).
2. Publicação dispara: notificação interna + push (conforme atribuições) + e-mail (circulares e comunicados).
3. Leitura é rastreada por destinatário; circulares não confirmadas podem ser reenviadas.

### 6.7 Fiscal (NFS-e)
1. Mensalmente, sistema consolida os serviços faturados por unidade (controle mensal de notas: contagem de alunos e valor total, excluindo cancelados e não emissores de nota).
2. Geração dos RPS (com simulação prévia) → envio do lote à administração fazendária municipal usando certificado digital da instituição.
3. Consulta de processamento do lote → registro das notas emitidas → vínculo às cobranças/alunos.
4. Suporte a retenções configuráveis (impostos sobre serviço).

---

## 7. Regras de Negócio Críticas

### 7.1 Financeiro
- **Moeda**: todos os valores em centavos (inteiros); exibição com conversão.
- **Boleto vencido**:
  - **Mora**: 2% sobre o valor original.
  - **Juros**: valor diário (mínimo de R$ 0,01/dia) × dias de atraso.
  - **Valor atualizado** = valor sem desconto + multa + mora.
  - **Desconto**: válido somente até a data-limite (pagamento pontual).
- **Vencimento**: nunca em sábado, domingo ou feriado — desloca para dia útil; calendário de feriados nacionais/municipais parametrizado.
- Dia de vencimento por aluno restrito a 1–28.
- Boleto liquidado é imutável.
- Bolsas/descontos têm vigência monitorada (rotina alerta expiração) e auditoria periódica de descontos aplicados.

### 7.2 Acadêmico
- Apenas um ano letivo corrente por vez.
- Agrupamento do aluno/prospect deriva da data de nascimento (faixas em meses) e da unidade.
- Ocupação de turma considera vagas regulares e inclusivas separadamente, além de trocas de turma já aprovadas e ainda não implementadas.
- Unidade pode estender outra (herda agrupamentos/estrutura).

### 7.3 Conteúdo e privacidade
- Fotos exigem **duas aprovações independentes** antes de ficarem visíveis às famílias.
- Acesso de parente é sempre filtrado pelas atribuições por aluno; responsável legal tem acesso pleno.
- Toda alteração relevante de cadastro gera registro de auditoria (quem, o quê, quando).

### 7.4 Estados canônicos
- **Requerimento**: 0 em aberto, 1 aprovado, 2 negado, 3 aguardando assinatura, 4 assinado, 5 cancelado/finalizado.
- **Conteúdo (comunicado/foto)**: ativo, pendente de aprovação, excluído (lixeira).
- **Interação comercial**: pendente, atrasada, concluída (derivado de data/hora × conclusão).
- **Cancelamento de matrícula**: ativo/finalizado (reativável enquanto não finalizado).
- **Consumo de refeição**: 0 recusou, 1 pouco, 2 metade, 3 bem, 4 muito bem.

---

## 8. Integrações Externas

| Integração | Finalidade | Pontos funcionais |
|---|---|---|
| **Autentique** (assinatura eletrônica) | Assinatura digital de contratos e requerimentos. | Criação de documentos, múltiplos signatários (responsável, representante, testemunhas), consulta de status/eventos (enviado, aberto, assinado, recusado), webhook de eventos, credencial por colaborador signatário. |
| **Bancos – Santander e Sicoob** | Cobrança registrada por boleto. | Geração de arquivos de remessa (entrada de título e pedido de baixa) e processamento de arquivos de retorno no padrão CNAB; layouts específicos por banco. |
| **Prefeitura (NFS-e municipal)** | Emissão de notas fiscais de serviço. | Envio e consulta de lotes de RPS via webservice oficial, autenticado por certificado digital da instituição (certificados renovados anualmente por CNPJ). |
| **Google** | Login social e envio de e-mails transacionais pela conta institucional. | Autenticação federada; remetente único do sistema. |
| **Apple / Facebook** | Login social no aplicativo. | Validação no provedor e emissão do token interno; deep links de retorno. |
| **OneSignal** (push) | Notificações push no aplicativo. | Destinatário endereçado por identificador externo do login; lotes de até 50 destinatários por chamada; campanhas e avisos automáticos. |
| **Provedor de SMS** | Código de recuperação de senha por SMS. | Envio do código de 6 dígitos ao telefone do responsável. |
| **Exportação publicitária** | Público customizado para mídia (ex.: pixel de anúncios). | Exportação periódica de CSV com contatos de alunos/responsáveis formatados. |

---

## 9. Processos Automáticos (rotinas agendadas)

### 9.1 Financeiro
| Rotina | Função | Periodicidade típica |
|---|---|---|
| Baixa de boletos | Processa arquivos de retorno bancário e liquida boletos; move arquivos processados. | Diária |
| Notificações de vencimento | Push + notificação interna para boletos vencendo hoje e vencidos (reaviso a cada 3 dias). | Diária |
| Conciliação movimentos × boletos | Vincula lançamentos de entrada aos boletos pagos. | Semanal |
| Prova real | Confere consistência entre boletos pagos e movimentos registrados. | Mensal |
| Vencimento de bolsas | Alerta bolsas/descontos expirando. | Diária |
| Verificação de descontos | Audita descontos aplicados em boletos. | Semanal |
| Relatório de boletos | Consolida pendentes/pagos. | Mensal |

### 9.2 Fiscal
| Rotina | Função |
|---|---|
| Envio de RPS | Transmite lotes de RPS à prefeitura (certificado digital). |
| Consulta de RPS | Verifica o processamento dos lotes enviados. |
| Verificação de RPS | Valida totais e valores por unidade antes do envio. |
| Controle mensal de notas | Consolida emissão mensal por unidade (alunos ativos, sem cancelamento, com flag de emissão), evitando duplicidade. |

### 9.3 Matrículas e contratos
| Rotina | Função |
|---|---|
| Renovações | Processa requerimentos de renovação: novo atendimento, turmas, serviços e valores do ano seguinte; gera log/relatório; alerta serviços inexistentes na nova turma. |
| Requerimentos | Implementa requerimentos assinados cuja data de vigência chegou (turno, permanência, turma, contratos). |
| Cancelamentos de matrícula | Efetiva cancelamentos programados. |
| Verificação de assinaturas | Consulta status na plataforma de assinatura e marca contratos totalmente assinados. |
| Testemunhas de assinatura | Gera/encaminha contratos para assinatura das testemunhas. |

### 9.4 Operação diária
| Rotina | Função |
|---|---|
| Fechamento de presenças | Preenche saída dos registros abertos com o horário padrão da turma. |
| Comunicado de presença | Gera comunicados de presença aos responsáveis. |

### 9.5 Dados e relatórios
| Rotina | Função |
|---|---|
| Exportação de contatos (CSV) | Alunos e responsáveis formatados para públicos de mídia. |
| Relatórios de apoio | Atendimentos, renovações inconsistentes, cotas de reserva incorretas, indicações. |

> Observação: o sistema atual possui ainda diversas rotinas pontuais de correção de dados (circulares, prospects, serviços, datas de desconto, presenças, fotos). Elas indicam pontos de fragilidade do modelo atual e não precisam ser reconstruídas como funcionalidades — ver §12.

---

## 10. Comunicações Geradas pelo Sistema

### 10.1 E-mails transacionais
| Gatilho | Destinatário | Conteúdo |
|---|---|---|
| Boletos vencidos (painel de cobrança) | Responsável financeiro | Tabela de débitos com dias de atraso e valores atualizados; link do boleto; assinatura da direção. |
| Circular publicada | Responsáveis das turmas | Conteúdo da circular + anexo. |
| Comunicado/interação | Responsável | Mensagem da equipe sobre o aluno. |
| Ocorrência | Responsável | Relato do evento. |
| Criação de usuário / redefinição | Usuário | Senha inicial ou nova senha. |
| Recuperação de senha (web e app) | Usuário | Token/código de recuperação. |
| Prospect inscrito | Interessado | Confirmação de interesse/visita. |
| Prospect duplicado | Equipe interna | Alerta de lead repetido. |
| Prospect convertido | Interessado | Boas-vindas. |

### 10.2 Push e notificações internas
- Novo boleto / boleto vencendo / vencido (com janela de reaviso de 3 dias).
- Comunicado publicado; foto aprovada; circular publicada.
- Campanhas customizadas do administrador (com filtro por atribuição).
- Toda push relevante é espelhada como notificação interna consultável no aplicativo.

### 10.3 Documentos gerados (PDF)
- **Contrato de serviços** (matrícula, renovação, alterações): dados do aluno, responsáveis, serviços, valores, signatários.
- **Boleto bancário**: beneficiário, aluno, valores, vencimento, instruções e código de barras/linha digitável.
- Circulares e documentos institucionais anexados (armazenados e servidos pelo sistema).

---

## 11. Requisitos Não Funcionais

- **Idioma/localização**: português do Brasil; fuso horário de Brasília; formatos DD/MM/AAAA e HH:MM; moeda Real (centavos como unidade interna).
- **Multiunidade**: dados e permissões segregados por unidade; unidades podem se estender.
- **Auditoria**: trilha de quem fez o quê em entidades sensíveis (cadastro, atendimento, financeiro).
- **Exclusão lógica** como padrão (lixeira/reativação) para conteúdo e cadastros sensíveis; nada crítico é apagado fisicamente.
- **Controle de versão do aplicativo** com versão mínima por plataforma.
- **Anexos e mídia**: armazenamento de fotos, receitas médicas, circulares, documentos e arquivos bancários, com controle de acesso por permissão.
- **Disponibilidade da rotina diária**: registros de sala (chamada, refeições, medicação) acontecem ao longo do dia letivo e exigem resposta rápida no aplicativo.
- **Concorrência**: aprovação dupla de fotos e fluxos de assinatura implicam estados intermediários consistentes.
- **Sigilo**: dados de menores, saúde (medicações/alergias) e finanças familiares — exigem proteção rigorosa de acesso, transporte e armazenamento, em conformidade com a legislação de proteção de dados (LGPD).
- **Certificados digitais** institucionais com renovação anual (integração fiscal).

---

## 12. Pontos de Atenção para a Nova Construção

Achados do sistema atual que **não devem ser replicados** ou merecem decisão explícita de projeto:

1. **Senha-mestre global** hard-coded que permite login como qualquer usuário — eliminar; substituir por impersonação auditada com permissão de administrador.
2. **Segredos no código/configuração versionada** (chaves de push, SMS, assinatura, segredo de token) — mover para gestão segura de segredos.
3. **Senha inicial derivada de dados pessoais** (CPF + ano) — substituir por convite com definição de senha pelo próprio usuário.
4. **Relações armazenadas como listas JSON desnormalizadas** (aluno↔turmas, parente↔alunos, turma↔colaboradores, atribuições) — modelar como relações normalizadas com integridade referencial, preservando a semântica descrita no §3.
5. **Grande volume de rotinas de correção de dados** — sintoma de invariantes não garantidas; o novo modelo deve impor essas invariantes na escrita (transações e validações), tornando as correções desnecessárias.
6. **Duas gerações de API convivendo** — unificar contratos, mantendo todas as capacidades do §5.
7. **Feriados parcialmente fixos em código** — parametrizar integralmente por calendário administrável.
8. **Token de API sem prefixo padrão e sem revogação granular** — adotar padrão de autorização com expiração, renovação e revogação.
9. **Limites operacionais herdados** (ex.: lotes de 50 destinatários de push, paginação de 50/500 registros) são restrições da integração/UX atuais — reavaliar conforme os provedores.
10. **Código morto/duplicado** no sistema atual (controladores de backup e variantes) não representa requisito — este blueprint já consolida apenas o comportamento vigente.

---

## 13. Glossário

| Termo | Significado |
|---|---|
| **Atendimento** | Conjunto de parâmetros acadêmicos do aluno: unidade, curso, agrupamento, nível, turno, permanência, horário, turmas e serviços. |
| **Agrupamento** | Faixa etária (em meses) que classifica o aluno/prospect. |
| **Atribuição** | Permissão granular concedida a um parente sobre um aluno específico. |
| **CMAE / CC / CRV** | Tipos de cota de cobrança: Cota Mensal de Anuidade Escolar / Cota de Composição / Cota de Reserva de Vaga. |
| **CNAB** | Padrão brasileiro de intercâmbio de arquivos de cobrança com bancos (remessa/retorno). |
| **Diário** | Agregado dos registros do dia de um aluno (presença, refeições, sono, medicação, higiene, comunicados). |
| **NFS-e / RPS** | Nota Fiscal de Serviço eletrônica municipal / Recibo Provisório de Serviços que a precede. |
| **Presença parcial** | Permanência do aluno por período menor que o horário contratado no dia. |
| **Prospect** | Interessado em matrícula ainda não convertido em aluno. |
| **Requerimento** | Solicitação formal sobre a matrícula/atendimento de um aluno, com fluxo de aprovação e, quando contratual, assinatura digital. |
| **Responsável legal** | Parente designado como responsável principal pelo aluno, com acesso e poderes plenos. |
| **Unidade** | Sede física da instituição; o sistema é multiunidade. |
