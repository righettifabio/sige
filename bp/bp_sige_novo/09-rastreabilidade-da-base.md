# 09 — Rastreabilidade da Base (garantia auditável G1)

> Documento 09 de 10. **Auditoria de cobertura**: cada capacidade, entidade, regra, fluxo, integração, rotina e comunicação do sistema atual (`bp/bp_sige_atual/bp_sige.md`) é mapeada para onde vive no novo blueprint, com a **natureza da preservação**:
>
> - **Literal** — MANTER sem mudança de comportamento.
> - **Seed** — comportamento atual preservado como registro default de um mecanismo generalizado (comportamento idêntico no dia 1).
> - **Generalizada** — capacidade absorvida por um modelo mais amplo, preservando a semântica.
> - **Corrigida** — item do §12 da base (defeito) substituído pela solução indicada.
>
> **Critério de aceite: zero linhas sem destino.** Resultado: atendido (ver §14).

---

## 1. Visão geral (base §1)

| Item da base | Destino | Natureza |
|---|---|---|
| Gestão de instituição de ensino infantil multiunidade | doc 00 §1; doc 01 PLT-01 | Generalizada (agora SaaS creche→médio) |
| Ciclo de vida do aluno e da operação | doc 00 §1; docs 02–04 | Literal |
| Frente web administrativa | doc 05; docs 01–04 | Literal |
| Frente API para app móvel | doc 06 API-01..23 | Generalizada (contrato unificado) |

## 2. Atores e acesso (base §2)

| Item da base | Destino | Natureza |
|---|---|---|
| Administrador, Colaborador, Responsável legal, Parente, Prospect | doc 00 §3 (personas) | Literal |
| Pessoa única + Login | doc 02 PES-01, PES-06 | Literal |
| Colaborador: unidades × módulos de acesso + flags | doc 02 PES-04 | Literal |
| Parente: atribuições por aluno (9 códigos) | doc 02 PES-02 | Literal (+ papéis, generalizada) |
| Responsável legal com acesso pleno | doc 02 PES-03 (regra fixa) | Literal |
| Alternância de perfil colaborador↔parente | doc 02 PES-06; doc 05 UI-05 | Literal |
| Autenticação web (e-mail/CPF + senha, token de recuperação) | doc 02 PES-06 | Literal |
| Autenticação app (token, expiração, header) | doc 02 PES-06; doc 06 API-10/02 | Corrigida (token assinado, valida antes) |
| Login social Google/Apple/Facebook | doc 02 PES-06; doc 06 INT-09 | Literal |
| Recuperação por código 6 dígitos e-mail/SMS | doc 02 PES-06 | Literal |
| Senha padrão derivada de dados pessoais (§12.3) | doc 02 PES-06; RQ-SEG-03 | Corrigida (convite) |
| Senha-mestre global (§12.1) | doc 01 PLT-06; RQ-SEG-02 | Corrigida (impersonação auditada) |

## 3. Modelo de domínio — entidades (base §3)

| Subdomínio / entidades da base | Destino | Natureza |
|---|---|---|
| **Pessoas**: Pessoa, Parente, Parentesco, Colaborador, Função, Endereço, refs, Login, Token recuperação | doc 02 PES-01..06; doc 01 PLT-19 (cadastros) | Literal |
| **Acadêmico**: Aluno, Turma, Curso, Agrupamento, Nível, Turno, Permanência, Horário, Presença, Diário, Cancelamento, Registro de Atendimento | doc 02 ACA-04..11, MAT-01/02/07, ACA-13 | Literal (Generalizada em estrutura configurável C1) |
| **Rotina/cuidados**: Refeição, Tipo de Refeição, Medicação, Troca de Fralda, Repouso, Agendamento | doc 02 ROT-01..03 | Literal |
| **Financeiro**: Boleto, Serviço, Tipo de Serviço, Valor, Movimento, Classificação de Custo, Fornecedor, Detalhe de Remessa, RPS/Lote, Nota Fiscal, Registro de Pagamento, Controle de Contratos | doc 03 FIN-01..21 | Literal (Boleto→Título generalizada C2) |
| **Comunicação**: Comunicado, Circular, Lote de Circular, Ocorrência, Notificação, Push/Lote, Foto/Compartilhamento, Documento/Capítulo | doc 04 COM-01..08 | Literal |
| **Comercial**: Prospect, Interação, Tipo de Interação, Inativação, Meio de Atendimento/Conhecimento, Acompanhamento Sistemático, Indicação | doc 02 ACA-01/02 | Literal |
| **Administrativo**: Unidade, Ano Letivo, Dia Letivo, Feriado, Requerimento, Log, Cor | doc 01 PLT-01/14; doc 02 ACA-05, MAT-06 | Literal |
| Valores em centavos | doc 00 PIL-13 | Literal |
| Extensão de unidade | doc 01 PLT-01 | Literal |
| Relações em JSON desnormalizado (§12.4) | doc 07 TEC-20; PIL-09 | Corrigida (modelo normalizado) |

## 4. Módulos web (base §4)

| Módulo da base | Destino | Natureza |
|---|---|---|
| 4.1 Dashboard (indicadores, gráficos por unidade) | doc 01 PLT-16 | Literal (+ KPIs acionáveis) |
| 4.2 Alunos (cadastro, busca parente por CPF, login automático, edição com auditoria, cancelamento, relatórios CSV) | doc 02 PES-01/02, MAT-01/02/07; doc 01 PLT-15 | Literal (login automático→convite, corrigida) |
| 4.3 Turmas (ocupação regular/inclusiva, histórico, alunos soltos, contratos não assinados, frequência) | doc 02 ACA-09, MAT-09 | Literal |
| 4.4 Serviços e Preços (catálogo por atributos, valores com vigência) | doc 03 FIN-02 | Literal |
| 4.5 Financeiro (boletos CMAE/CC/CRV, remessa/retorno CNAB, controle de cobrança, NFS-e, plano de contas, fornecedores) | doc 03 FIN-01..21 | Literal / Generalizada |
| 4.6 Requerimentos (8 tipos, status 0–5, contrato, implementação automática) | doc 02 MAT-06 | Literal |
| 4.7 Prospects (cadastro, agendamento público, funil, interações, gráficos) | doc 02 ACA-01/02 | Literal |
| 4.8 Anos Letivos e Calendário (ano corrente, renovação em lote, dias letivos) | doc 02 ACA-05, MAT-08 | Literal |
| 4.9 Dia a Dia (timeline de comunicados/fotos, menções, dupla aprovação) | doc 04 COM-01/04 | Literal |
| 4.10 Comunicados e Ocorrências | doc 04 COM-01/03 | Literal |
| 4.11 Circulares (individualizadas, confirmação, reenvio) | doc 04 COM-02 | Literal |
| 4.12 Fotos (dupla aprovação, compartilhamento) | doc 04 COM-04 | Literal |
| 4.13 Notificações (centro + campanhas push com filtro por atribuição) | doc 04 COM-06 | Literal |
| 4.14 Colaboradores (cadastro, permissões, importação CSV) | doc 02 PES-04 | Literal |
| 4.15 Documentos (biblioteca em capítulos) | doc 04 COM-08 | Literal |
| 4.16 Configuração (parametrização por módulo) | doc 01 PLT-18/19 | Literal |

## 5. API / App (base §5)

| Capacidade da base | Destino | Natureza |
|---|---|---|
| 5.1 Responsáveis (diário, financeiro c/ PDF, circulares, fotos, renovação, dados, notificações, calendário) | doc 06 API-12..21; doc 04 POR-02 | Literal (contrato novo) |
| 5.2 Colaboradores (turmas, rotina de sala completa, comunicados, fotos) | doc 06 API-13/14/16/19; doc 04 POR-04 | Literal |
| 5.3 Contratos da API (token, envelope, versão mínima, deep links, 2 gerações) | doc 06 API-01/02/03/23 | Generalizada (unificação) + Corrigida (2 gerações→1) |

## 6. Fluxos de negócio (base §6)

| Fluxo da base | Destino | Natureza |
|---|---|---|
| 6.1 Captação → Matrícula | doc 02 ACA-01/02, MAT-03/04/05 | Literal (+ matrícula online, gatilho C3) |
| 6.2 Cobrança mensal | doc 03 FIN-05/06/09 | Literal (+ Regra de Cobrança, PIX) |
| 6.3 Alteração contratual via Requerimento | doc 02 MAT-06 | Literal |
| 6.4 Renovação anual | doc 02 MAT-08 | Literal |
| 6.5 Rotina diária do aluno (creche) | doc 02 ROT-01..03 | Literal |
| 6.6 Comunicação institucional | doc 04 COM-01..06 | Literal |
| 6.7 Fiscal (NFS-e) | doc 03 FIN-20/21 | Generalizada (hub Focus) |

## 7. Regras de negócio críticas (base §7)

| Regra da base | Destino | Natureza |
|---|---|---|
| 7.1 Financeiro: centavos, mora 2%, juro diário, valor atualizado, desconto até vencimento, vencimento em dia útil, dia 1–28, boleto liquidado imutável, bolsas com vigência | doc 03 FIN-07/10, invariantes §7 | Literal (multa/juros→seed) |
| 7.2 Acadêmico: ano corrente único, agrupamento por idade, ocupação regular/inclusiva, extensão de unidade | doc 02 invariantes 1–4; doc 01 PLT-01 | Literal |
| 7.3 Conteúdo/privacidade: dupla aprovação de fotos, atribuições, auditoria | doc 04 COM-04, POR-01; doc 01 PLT-14 | Literal |
| 7.4 Estados canônicos: requerimento 0–5, conteúdo, interação, cancelamento, consumo de refeição | doc 02 MAT-06, ROT-02, ACA-01; doc 04 COM-01 | Literal |

## 8. Integrações externas (base §8)

| Integração da base | Destino | Natureza |
|---|---|---|
| Autentique (assinatura) | doc 06 INT-01 | Literal (default do conector dual) |
| Santander e Sicoob (CNAB) | doc 06 INT-03; doc 03 FIN-08/09 | Literal (seeds do agente de cobrança) |
| Prefeitura (NFS-e) | doc 06 INT-02; doc 03 FIN-20 | Generalizada (hub fiscal; direta não reconstruída) |
| Google (login + e-mail) | doc 06 INT-06/09 | Literal |
| Apple/Facebook (login) | doc 06 INT-09 | Literal |
| OneSignal (push) | doc 06 INT-05 | Generalizada (provedor de push abstrato) |
| Provedor de SMS | doc 06 INT-07 | Literal |
| Exportação publicitária (CSV) | doc 01 PLT-15; doc 04 LGP-03 | Literal (condicionada a consentimento) |

## 9. Processos automáticos / rotinas (base §9)

| Rotina da base | Destino | Natureza |
|---|---|---|
| Financeiro: baixa de boletos, notificações de vencimento, conciliação, prova real, vencimento de bolsas, verificação de descontos, relatório de boletos | doc 01 PLT-12; doc 08 V2-02 (Agentes Cobrança/Conciliação/Bolsas) | Literal (como serviços de fila) |
| Fiscal: envio/consulta/verificação de RPS, controle mensal de notas | doc 03 FIN-21; doc 08 V2-02 (Agente Fiscal) | Generalizada (hub) |
| Matrículas/contratos: renovações, requerimentos, cancelamentos, verificação de assinaturas, testemunhas | doc 02 MAT-06/08; doc 08 V2-02 (Agentes Matrícula/Contratos) | Literal |
| Operação diária: fechamento de presenças, comunicado de presença | doc 02 ROT-03; doc 08 V2-02 (Agente Presença) | Literal |
| Dados/relatórios: exportação de contatos, relatórios de apoio | doc 01 PLT-15 | Literal |
| Rotinas de correção de dados (§9 obs / §12.5) | doc 00 PIL-09; doc 07 TEC-10 | Corrigida (invariantes na escrita — eliminadas) |

## 10. Comunicações geradas (base §10)

| Item da base | Destino | Natureza |
|---|---|---|
| E-mails transacionais (9 gatilhos) | doc 04 COM-07 §5 | Literal (templates seed) |
| Push e notificações internas (com reaviso 3 dias) | doc 04 COM-06; doc 03 FIN-06 | Literal (reaviso→seed da régua) |
| Documentos PDF (contrato, boleto, circulares) | doc 04 §5; doc 03 FIN-01 | Literal (com white-label) |

## 11. Requisitos não funcionais (base §11)

| RNF da base | Destino | Natureza |
|---|---|---|
| Idioma/localização PT-BR, fuso, formatos, moeda | doc 00 RQ-LOC | Literal |
| Multiunidade segregada, extensão | doc 01 PLT-01 | Literal |
| Auditoria de entidades sensíveis | doc 01 PLT-14 | Literal (onipresente) |
| Exclusão lógica/lixeira/reativação | doc 00 PIL-10 | Literal |
| Controle de versão do app | doc 04 POR-08; doc 06 API-23 | Literal |
| Anexos e mídia com controle de acesso | doc 06 API-02 (RQ-SEG-04) | Corrigida (binário autorizado) |
| Disponibilidade da rotina diária | doc 00 RQ-LOC | Literal |
| Concorrência (dupla aprovação, assinatura) | doc 04 COM-04; doc 02 MAT-05 | Literal |
| Sigilo (menores, saúde, finanças) — LGPD | doc 00 RQ-SEG; doc 04 LGP-01..03 | Literal (+ termos digitais) |
| Certificados digitais (fiscal) | doc 03 FIN-20 | Generalizada (hub assume) |

## 12. Pontos de atenção / §12 da base (defeitos a corrigir)

| §12 da base | Destino | Natureza |
|---|---|---|
| 12.1 Senha-mestre global | doc 01 PLT-06 | Corrigida |
| 12.2 Segredos no código | doc 07 TEC-05; RQ-SEG-02 | Corrigida |
| 12.3 Senha derivada de CPF | doc 02 PES-06; RQ-SEG-03 | Corrigida |
| 12.4 Relações em JSON desnormalizado | doc 07 TEC-20; PIL-09 | Corrigida |
| 12.5 Rotinas de correção de dados | doc 00 PIL-09 | Corrigida |
| 12.6 Duas gerações de API | doc 06 API-01 | Corrigida |
| 12.7 Feriados fixos em código | doc 02 ACA-05; RQ-PAR-02 | Corrigida |
| 12.8 Token sem prefixo/revogação | doc 06 API-02; RQ-SEG-01 | Corrigida |
| 12.9 Limites operacionais herdados | doc 00 RQ-PAR-03 | Corrigida |
| 12.10 Código morto/duplicado | doc 06 §2 (sem destino); inventário §6 | Corrigida (descartado) |

## 13. Glossário (base §13)

Todos os termos da base estão preservados e ampliados no glossário unificado (doc 00 §7): Atendimento, Agrupamento, Atribuição, CMAE/CC/CRV, CNAB, Diário, NFS-e/RPS, Presença parcial, Prospect, Requerimento, Responsável legal, Unidade.

## 14. Resultado da auditoria

- **Capacidades funcionais da base (§§1–11): 100% com destino.** Nenhum item MANTER ficou sem seção correspondente no novo BP.
- **Defeitos (§12): 100% endereçados** por solução explícita (Corrigida).
- **Itens descontinuados: somente os defeitos do §12** (nenhuma funcionalidade de usuário).
- **Naturezas**: predominância de *Literal*; *Seed* nos pontos onde a base vira default de mecanismo configurável (multa/juros, régua, situações, cotas, agentes de cobrança, templates de e-mail); *Generalizada* em estrutura acadêmica, Título, NFS-e e API; *Corrigida* nos 10 pontos do §12.

> Garantia G1 atendida: tudo que o sistema atual faz existe na v1, e nenhuma funcionalidade da escola foi descontinuada.
