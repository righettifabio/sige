# 08 — Visão v2 e Diferimentos

> Documento 08 de 10. Registra o que **não** entra na v1 mas orienta o seu desenho: a camada transversal de inteligência (SIGE Intelligence) e os módulos/integrações diferidos. O propósito é garantir que a v1 não crie obstáculos a estas evoluções. IDs: `V2-xx`.

---

## 1. SIGE Intelligence — camada transversal de inteligência (v2)

### V2-01 — Conceito
Camada de inteligência composta por **agentes especializados por domínio** que atravessam todos os módulos e operam em segundo plano — **não é um módulo isolado**. Duas classes com naturezas distintas. A v1 **não** implementa os agentes, mas deve cumprir os anti-obstáculos (§1.5).

### V2-02 — Classe 1: Agentes determinísticos (CRON) — sem LLM, sempre ativos, custo zero
Rodam em background, calculam/monitoram/alertam por regras; não podem ser desligados pois alimentam dados críticos. **As ~20 rotinas agendadas da base são a semente desta classe**, reorganizadas por domínio (já executando como serviços de fila na v1 — PLT-12):

| Agente | Absorve da base | Evolução na v2 |
|---|---|---|
| Cobrança | Baixa de boletos, notificações de vencimento | Executa a Regra de Cobrança (FIN-05) e a régua (FIN-06) |
| Conciliação | Conciliação título×movimento, prova real | + conciliação bancária e PIX |
| Bolsas e Descontos | Vencimento de bolsas, verificação de descontos | + workflow de solicitação |
| Fiscal | Envio/consulta/verificação de RPS, controle mensal | Opera via hub Focus NFe (FIN-20) |
| Matrícula | Renovações, requerimentos, cancelamentos | + gatilhos configuráveis (MAT-04), promoção em lote |
| Contratos | Verificação de assinaturas, testemunhas | + SLA de assinatura, cancelamento por prazo |
| Presença | Fechamento de presenças, comunicado de presença | + apuração nos dois modos de frequência (C5) |
| Ocupação | (novo) | Monitora vagas×matrículas, alerta lotação/ociosidade |
| Funil | Relatórios de apoio (interações) | Monitora interações atrasadas, leads parados, indicações |

### V2-03 — Classe 2: Agentes LLM — opcionais, custo por uso, ativados por quem governa o domínio
Cada agente **redige/sugere, nunca executa sozinho** (V2-04). Candidatos para o domínio escolar:

| Agente | Função | Ativação |
|---|---|---|
| Relatório Pedagógico | Auxilia o professor a redigir avaliações descritivas a partir do diário/rotina | Escola |
| Comunicação | Redige comunicados/circulares/respostas para aprovação | Escola |
| Cobrança Assistida | Redige mensagens de cobrança personalizadas para aprovação | Escola |
| Insights de Gestão | Narra KPIs (inadimplência, ocupação, evasão, funil) em linguagem natural | Escola |
| Retenção | Cruza sinais de evasão (frequência, inadimplência, ocorrências) e propõe plano | Escola |
| Captação | Qualifica prospects e redige follow-ups para aprovação do comercial | Escola |
| Onboarding | Conduz a configuração inicial de uma escola nova de forma conversacional | Operador do SaaS |
| Suporte | Analisa tickets e sugere respostas com base na base de conhecimento | Operador do SaaS |

### V2-04 — Princípios invioláveis
- A inteligência **nunca age sozinha**: toda comunicação às famílias e todo ato administrativo passam por **aprovação humana** (reaproveita o gate de aprovação já generalizado na v1 — PIL-05).
- A **escola é sempre a autoridade final**.
- **A camada é invisível para as famílias**: pais e alunos nunca interagem com a IA; tudo chega como mensagem da escola, após aprovação.
- Toda ação de agente é **auditada** (ator "agente X", aprovador humano, data — PIL-07) e logada na caixa de saída quando gera comunicação.

### V2-05 — Governança de motorização (camada de administração de LLMs)
Três níveis:
1. **Operador do SaaS** — define os modelos disponíveis via **OpenRouter**; liga/desliga globalmente os agentes de plataforma (onboarding, suporte).
2. **Escola (tenant)** — insere seu **próprio token OpenRouter**, escolhe o modelo e liga/desliga cada agente LLM do seu domínio. **Cada escola decide como conectar seu modelo e arca com seu custo.**
3. **Usuário final** — preferências individuais onde aplicável.

**Evolução futura (v3+):** outros provedores além do OpenRouter e **modelos locais** (self-hosted), mantendo a mesma camada de administração de LLMs como abstração.

### V2-06 — Interface
O **painel lateral direito da shell** (UI-07) é a casa da SIGE Intelligence — sempre presente, expansível/recolhível, com nome e avatar personalizáveis por escola; recolhido, exibe indicador discreto de atividade.

### V2-07 — Anti-obstáculos obrigatórios na v1
Para a v2 ser possível sem refatoração, a v1 entrega:
1. Rotinas como **serviços nomeados por domínio** (não scripts soltos) — PLT-12.
2. **Eventos de domínio** publicáveis (matrícula efetivada, boleto vencido, presença fechada…) — PLT-13.
3. **Painel lateral direito reservado** na shell — UI-07.
4. **Gate de aprovação humana generalizado** (reaproveitável pelos agentes) — PIL-05.
5. **Auditoria com autoria não-humana** prevista no modelo de log — PIL-07.

---

## 2. Módulos e integrações diferidos

Itens conscientemente fora da v1, com a razão e o que a v1 preserva para não bloqueá-los:

| Item | Razão do diferimento | Não-obstáculo garantido na v1 |
|---|---|---|
| **Chat escola↔família** | Muda a operação (SLA de resposta); decisão de não incluir | Caixa de saída e modelo de comunicação extensíveis |
| **BI embarcado** | v1 atende com KPIs nativos + central de relatórios | Dados e métricas já estruturados por tenant/unidade |
| **Biblioteca** | Sem demanda; sai junto o badge BIB de impedimento | Mecanismo de impedimento aceita novos canais |
| **Filantropia/CEBAS** | Nicho de escolas com certificação | Bolsas/descontos e ficha já modelados |
| **Ranking de alunos** | Incompatível com parte do público; desligável no futuro | Boletim já tem flag de exibição prevista |
| **Catraca/acesso físico** | Depende de hardware | API pública pronta para marcação de frequência |
| **Google for Education** | Só se a escola usar o ecossistema | Provisionamento via API pública |
| **Gerador de consultas (SQL)** | Risco de segurança; v1 usa central de relatórios + futuro BI | Relatórios configuráveis cobrem a necessidade comum |
| **Conector de garantidora de recebíveis** | Sem parceiro; acopla ciclo do título | Conector abstrato de cobrança + estados paralelos do título |
| **Registro online de boleto por API bancária** | Esforço de homologação; v1 usa CNAB+PIX | Arquitetura de adaptadores bancários plugáveis (FIN-08) |
| **Builds white-label do app por escola** | Custo operacional nas lojas | App único com tema dinâmico (PLT-10) |
| **Billing/tiers comerciais do SaaS** | Gestão comercial manual na v1 | Feature flags por tenant já existem (PLT-05) |
| **Teams/editoras (provisionamento de conteúdo)** | Sem parceria | API pública de parceiros |

---

## 3. Invariante de evolução

A v1 não pode introduzir decisão que torne os itens acima inviáveis sem refatoração estrutural. Em especial: rotinas como serviços, eventos de domínio, conectores abstratos, gate de aprovação e auditoria com autoria de agente são **requisitos da v1** justamente por serem as fundações da v2.
