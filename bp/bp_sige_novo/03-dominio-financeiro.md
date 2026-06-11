# 03 — Domínio Financeiro

> Documento 03 de 10. Receita (títulos, cotas, planos, Regra de Cobrança, CNAB+PIX, régua, negociação, descontos/bolsas), caixa físico, despesa e contábil, fiscal (NFS-e via hub), e o mecanismo transversal de impedimentos. IDs: `FIN-xx`.
>
> Regra-mestra: o ciclo financeiro da base é MANTER; o repertório agrega profundidade (caixa, contábil, negociação, PIX); os conflitos C2 (Título+cotas+plano) e C7 (impedimentos) aplicam a estratégia validada. **Dinheiro sempre em centavos** (PIL-13). Boleto/título liquidado é imutável (MANTER).

---

## 1. Núcleo de receita

### FIN-01 — Título (entidade central, generalização do boleto) (C2)
O **Título** é a entidade central de cobrança — generalização do "boleto" da base, necessária para acomodar boleto CNAB, PIX e caixa sob o mesmo conceito. Campos: número (sequencial nomeado — PLT-21), aluno e responsável **pagador**, serviço(s) cobrado(s), **cota** (classificação — FIN-03), competência (mês/ano), vencimento, valor do documento × valor a receber (com multa/juros) × valor recebido, parcela ("NN/NN"), datas de pagamento/baixa, tipo de liquidação, situação, vínculo a turma, **estados paralelos** (situação de cobrança registrada no banco, situação no agente, pendências). Forma de recebimento associada (FIN-08). Boleto é a representação bancária de um título registrado.

**Invariante:** título liquidado é imutável (MANTER). Estados paralelos independentes (cobrança registrada × situação no agente × pendência) coexistem no mesmo título.

### FIN-02 — Serviço e Valor (motor de preço) — MANTER
Serviço = oferta comercializável (combinação de unidade + tipo de serviço + curso + agrupamento/série + nível + turno + permanência + horário), obrigatório/opcional, ordenável (MANTER, base §3.4). **Valor** = preço de um serviço com **vigência** (início/fim); múltiplas vigências por serviço; o sistema resolve o valor vigente pela data. Este é o motor de precificação — preservado integralmente.

### FIN-03 — Cota (classificação do título, seeds) (C2)
Tipo de cota é classificação configurável do título, com seeds reproduzindo a base: **CMAE** (Cota Mensal de Anuidade Escolar), **CC** (Cota de Composição), **CRV** (Cota de Reserva de Vaga). A cota não desaparece — é atributo obrigatório no segmento que a usa. A CRV conecta-se à reserva de vaga paga (MAT-04).

### FIN-04 — Plano de Pagamento (camada de oferta) (C2) — ACRESCENTAR
Pacote ofertável na matrícula: serviços × quantidade de parcelas × valores × descontos, com vínculo a período e disponibilidade na matrícula online. É uma **camada de oferta** acima do motor de serviços/valores (FIN-02) — empacota e parcela para apresentar na matrícula; não substitui o motor de preço.

### FIN-05 — Regra de Cobrança (conceito de 1ª classe) (G7) — formalização nova
Entidade configurável por escola que unifica a geração de cobrança, hoje implícita no código da base:

| Dimensão | Conteúdo |
|---|---|
| **Origem** | serviço contratado · plano de pagamento · matrícula/pré-matrícula · avulso |
| **Alvo** | responsável financeiro do aluno |
| **Gatilho** | mensal (recorrente) · na matrícula · na renovação · manual |
| **Valor** | resolvido do Serviço/Valor vigente (FIN-02) ou do plano (FIN-04) |
| **Cota** | classificação do título gerado (FIN-03) |
| **Recorrência** | periodicidade, nº de parcelas, competência |
| **Vencimento** | dia preferencial do responsável (FIN-07), deslocado para dia útil |

**Seed:** a regra "geração mensal de títulos para alunos ativos conforme serviços contratados, dia de vencimento e valores vigentes" reproduz exatamente o comportamento atual da base. Régua de comunicação (FIN-06), multa/juros (FIN-07) e descontos (FIN-10) permanecem cadastros próprios **referenciados** pela regra, cada um com sua responsabilidade. A geração executa via Agente de Cobrança (rotina/fila — PLT-12) com simulação disponível.

### FIN-06 — Régua de cobrança (comunicação automática) — APRIMORAR
Regras de comunicação sobre títulos: **offset de dias do vencimento** (ex.: *-15, -5, 0, +N (exemplo)*) × **canal** (e-mail/SMS/push) × lista de serviços × forma de recebimento × **texto por regra** (template) × conta de envio; opção "só enviar se o título estiver registrado". **Seeds** reproduzem os avisos da base: "vence hoje" e "vencido com reaviso a cada 3 dias". Histórico de envios no detalhe do título; tudo logado na caixa de saída (doc 04). Execução diária (horário parametrizável; *14h (exemplo)*) — não é tempo real.

### FIN-07 — Multa, juros, mora e dia de vencimento — APRIMORAR
**Cálculo de multa/juros** como cadastro nomeado reutilizável (percentual OU valor de multa; percentual ao mês OU valor diário de juros), com default por unidade. **Seed** = regra atual da base: **multa 2%**, **juro diário mínimo R$ 0,01/dia × dias de atraso**, valor atualizado = valor sem desconto + multa + mora, **desconto válido só até a data-limite** (pagamento pontual). Vencimento **nunca em sábado/domingo/feriado** — desloca para dia útil (MANTER). **Dia de vencimento** passa do aluno para o **responsável financeiro** (migração 1:1), com override por aluno; restrito a 1–28 (MANTER).

### FIN-08 — Formas de recebimento e Agente de Cobrança — MANTER + APRIMORAR
**Forma de recebimento**: Direto (caixa), Boleto bancário (CNAB), PIX (FIN-09). **Agente de Cobrança** como cadastro (APRIMORAR — generaliza os dois bancos fixos da base): banco/carteira, formato do arquivo (CNAB), flags (gera impedimento, apenas títulos em aberto, liquidar direto na escola, permite cobrança registrada). **Seeds**: Santander e Sicoob, reproduzindo os layouts atuais. Arquitetura de **adaptadores bancários plugáveis** — novos bancos entram sem código de núcleo (registro online via API bancária fica para depois — fora da v1).

### FIN-09 — CNAB + PIX (cobrança bancária da v1)
- **CNAB remessa/retorno** (MANTER literal): geração de arquivo de intercâmbio por agente (entrada de título e pedido de baixa), sequenciamento de lotes, notificação aos responsáveis após emissão; importação de retorno com **baixa automática** (data de liquidação + valor) e **baixa manual** por código de barras (validação 44/47 dígitos). Seeds Santander e Sicoob.
- **PIX** (ACRESCENTAR): QR dinâmico, boleto híbrido (copia-e-cola), conciliação automática de recebimento. Forma de recebimento adicional sobre a mesma arquitetura de adaptadores.

### FIN-10 — Descontos e bolsas — MANTER + APRIMORAR
Desconto como cadastro: **condicional** (por pontualidade, expira) × **permanente** (não condicionado), **ordem de cálculo** (cascata), classificação contábil, regra de concessão. **Seed** = o desconto-pontualidade embutido no boleto da base. **Bolsa** por aluno×turma×vigência com autorização (autor) e origem da solicitação; vigência monitorada por rotina (alerta de expiração) e auditoria periódica — MANTER, com workflow de autorização e solicitação pelo portal com termo de consentimento (APRIMORAR). Relatórios contábeis tratam condicionais à parte (invariante).

### FIN-11 — Negociação de dívida — ACRESCENTAR
Entidade de 1ª classe: título derivado com regras próprias de multa/juros/desconto, mantendo descontos condicionais conforme parâmetro; "multa/juros de negociação" vira serviço do título; negociação de título em cobrança registrada conforme parâmetro. Excluída dos relatórios contábeis de faturamento (invariante 18). Respeita a imutabilidade do título liquidado.

### FIN-12 — Pagamento a menor / a maior — ACRESCENTAR
Regras na liquidação (antes da imutabilidade): a menor → desconto automático limitado (apenas multa/juros até um valor-limite) ou **cobrar diferença em novo título**; a maior → considerar como juros até um limite, com devolução rastreada. Travas de sanidade em lote: bloquear liquidação com >40% de pagamentos a menor ou de títulos vencidos há >180 dias, com janela de carência configurável.

### FIN-13 — Telecobrança — ACRESCENTAR
Cobrança ativa com agenda de próximo contato por responsável, complementando o painel de cobrança da base (e-mail consolidado de débitos com valores atualizados — MANTER).

---

## 2. Caixa físico / tesouraria (G2 — completo) — ACRESCENTAR

### FIN-14 — Caixa
Sessão de tesouraria por operador: abertura/fechamento nominal, saldo, numeração sequencial global, retirada automática ao fechar (conta de destino), reabertura conforme permissão. Multi-empresa: recebimento e aglutinação de títulos de empresas diferentes conforme parâmetro.

### FIN-15 — Meios de recebimento no caixa
Tipos (cadastro, código de 3 letras): dinheiro, cartão (crédito/débito) com **operadora e repasse D+n**, cheque e **cheque pré-datado** (valorizado pelo "bom-para"), **custódia de cheques** (destinos: depósito, devolução, tesouraria), depósito, PIX presencial, liquidação manual. Limites de desconto concedido no balcão por percentual e valor.

---

## 3. Despesa e contábil (G2 — retaguarda completa) — ACRESCENTAR/APRIMORAR

### FIN-16 — Contas a pagar e favorecidos
Fornecedor/favorecido (PF/PJ), despesa com empresa, plano de contas (multi-select), centro de resultado, datas (emissão/vencimento/baixa), situação, "pago pelo caixa?". Generaliza o Movimento de despesa + Fornecedor da base.

### FIN-17 — Plano de contas e centro de resultado
Plano de contas **hierárquico** (sintética/analítica, conta-pai, natureza entrada/saída, nº contábil para exportação) — a classificação de custo plana da base migra como contas analíticas sob uma hierarquia mínima. Centro de resultado para classificação gerencial.

### FIN-18 — Conciliação, fluxo de caixa e exportação
Conciliação bancária por conta (com data de início e importação de extrato); conciliação título×movimento e **prova real** da base preservadas (rotinas — PLT-12). **DFC** previsto/realizado. **Exportação contábil** (com nº de conta contábil e data de corte) para o contador.

### FIN-19 — Conta financeira
Conta bancária por empresa/unidade: banco/agência/conta com dígitos, saldo inicial, permite saldo negativo, usa conciliação (com data de início), layouts de remessa/retorno, conta contábil de integração, disponível para retirada de caixa.

---

## 4. Fiscal — NFS-e via hub (G6)

### FIN-20 — Emissão de NFS-e por conector fiscal abstrato
A emissão de NFS-e ocorre por **conector fiscal abstrato** com **Focus NFe** como adaptador default (NFE.io como alternativa homologável; PlugNotas/Nuvem Fiscal como plano B). O conector assume assinatura XML, mTLS, certificado e layout municipal/Nacional. A **integração direta com o webservice da prefeitura não é reconstruída** (era mecanismo, não funcionalidade). Alinhado à **NFS-e Nacional** (DPS, reforma tributária IBS/CBS); dedup por referência idempotente.

### FIN-21 — Funcionalidade fiscal preservada (MANTER)
Sobre o conector: geração de RPS/DPS por período, **simulação prévia**, envio em lote, consulta de status, recriação de lotes com itens rejeitados, lista de notas emitidas, **controle mensal de emissão por unidade** (contagem de alunos e valor, excluindo cancelados e não emissores), vínculo de notas a alunos, variáveis na discriminação do serviço (nome/CPF/matrícula do aluno), retenções configuráveis. Multi-CNPJ (uma nota por empresa). Rotinas fiscais como serviços de fila (Agente Fiscal — doc 08).

---

## 5. Impedimentos — mecanismo transversal (C7, PIL-06)

### FIN-22 — Impedimentos configuráveis por canal
Pendências geram bloqueios configuráveis por canal, com badges na listagem e no cadastro de aluno/responsável: **OBS** (observação com impedimento), **FNC** (inadimplente), **COB** (título em cobrança registrada em aberto), **PDM** (procedimentos de matrícula pendentes). *(BIB/biblioteca fora da v1.)* Canais bloqueáveis (cada um com tipo de impedimento próprio na unidade): matrícula, rematrícula, agendamento, impressão de boleto web, recebimento no caixa, RM online, web/portal, **boletim**.

### FIN-23 — Configuração pela escola + guard-rails de produto (D4)
A escola decide o que bloquear (modelo do repertório, decisão D4) — **inclusive boletim**. O produto:
- Exibe **disclaimers legais** nas telas de configuração de impedimento que tocam boletim/documentos (a Lei 9.870/1999 e o ECA vedam reter documentos escolares e aplicar penalidade pedagógica por inadimplência; recusar **re**matrícula é lícito). A responsabilidade da configuração é da escola.
- **Regra fixa inviolável** (acima da configuração): o responsável legal nunca perde acesso aos **dados do próprio filho** (PES-03).
- Todo override de impedimento é auditado (autor + justificativa + data — PIL-07); a escola pode oferecer "corrigir impedimento" direto no alerta.

### FIN-24 — Data de bloqueio de operações financeiras — ACRESCENTAR
Fechamento contábil retroativo: operações antes da data de bloqueio só por operadores autorizados (override auditado).

---

## 6. Garantidora de recebíveis — fora da v1

O conceito de "conector de garantidora" (espelhamento de títulos, bloqueio de edição local de título garantido, fila de conciliação) fica **registrado para o futuro** (doc 08), sem parceiro nem implementação na v1.

---

## 7. Invariantes do domínio financeiro

1. Todos os valores em centavos (inteiros); exibição com conversão (PIL-13).
2. Título/boleto liquidado é imutável.
3. Vencimento nunca cai em sábado, domingo ou feriado — desloca para dia útil; dia base 1–28.
4. Desconto por pontualidade vale somente até a data-limite.
5. A geração de cobrança é sempre expressa por uma Regra de Cobrança; a seed reproduz a geração mensal da base (FIN-05).
6. A cota (CMAE/CC/CRV) é classificação obrigatória do título nos segmentos que a usam.
7. Multa/juros é cadastro com default por unidade; a regra atual da base é seed imutável de migração.
8. Liquidação com >40% a menor ou de títulos vencidos há >180 dias é bloqueada, salvo carência configurada.
9. Operação financeira anterior à data de bloqueio contábil exige operador autorizado (override auditado).
10. Relatórios contábeis ignoram serviços do tipo "negociação"; cancelamentos só repercutem após o mês do faturamento.
11. NFS-e é emitida exclusivamente via conector fiscal; nenhuma integração direta com prefeitura é mantida.
12. Impedimentos são configuráveis pela escola, mas o responsável legal nunca perde acesso aos dados do filho.
13. Caixa: numeração sequencial global; repasses de cartão para a conta da operadora; cheque pré-datado valorizado pelo "bom-para".
14. Bolsa/desconto tem vigência monitorada e auditoria; concessão registra autor e origem.
