# 01 — Plataforma e Administração do SaaS

> Documento 01 de 10. Define o modelo de tenancy, o backoffice do operador, o white-label, os serviços transversais de plataforma (flags, auditoria, lotes, filas, relatórios) e a administração de configurações pela escola. IDs: `PLT-xx`.

---

## 1. Modelo de tenancy

### PLT-01 — Hierarquia organizacional
Três níveis:

```
Instituição (tenant = escola cliente)
└── Unidade (campus/sede física; 1..N)
    └── Empresa (CNPJ de faturamento; 1..N por unidade)
```

- **Instituição** é a fronteira de isolamento: dados, usuários, configuração, marca e subdomínio.
- **Unidade** é a dimensão operacional: cadastros, permissões, relatórios, calendários e parâmetros são segregáveis por unidade. Uma unidade pode **estender** outra, herdando estrutura (agrupamentos, serviços) — comportamento da base, preservado.
- **Empresa** é a dimensão fiscal: contas bancárias, caixas, títulos e NFS-e referenciam uma empresa. Uma unidade pode faturar por múltiplas empresas; o financeiro parametriza recebimento e aglutinação multi-empresa (doc 03).

### PLT-02 — Resolução de tenant
O tenant é resolvido por **subdomínio** (`escola.sige.app.br` *(exemplo)*) e confirmado por claim no token de sessão/API. **Proibido** tenant em path de URL (RQ-SEG-04). Domínio próprio da escola (CNAME) é suportado como evolução sem mudança de modelo.

### PLT-03 — Ciclo de vida do tenant
Estados: `provisionando → ativo → suspenso → desativado`. Suspensão bloqueia login de usuários da escola (exceto operador do SaaS) preservando dados; desativação congela o tenant (somente leitura para o operador, exportação de dados disponível). Nada é apagado fisicamente (PIL-10). Toda transição é auditada.

---

## 2. Backoffice do operador do SaaS (G2 — nível "operacional essencial")

### PLT-04 — Onboarding de escolas (wizard)
Fluxo de provisionamento em etapas, com simulação/preview antes de ativar (PIL-08):
1. **Instituição** — razão social, nome fantasia, subdomínio, contato do administrador.
2. **Unidades e Empresas** — ao menos 1 unidade e 1 empresa (CNPJ).
3. **Template de estrutura acadêmica** — escolha de um ou mais modelos de partida: *Educação Infantil (agrupamentos etários + permanência/horário)*, *Fundamental seriado*, *Médio (com itinerários)* — a escola ajusta depois (doc 02 §3).
4. **Seeds** — criação automática dos registros de sistema: situações de aluno, motivos de inativação, cotas (CMAE/CC/CRV), regra de multa/juros default, regras default da régua de cobrança, templates de e-mail, termos LGPD modelo (texto a editar).
5. **Usuário inicial** — convite por e-mail com definição de senha pelo próprio administrador (RQ-SEG-03).
6. **Ativação** — checklist de pendências mínimas (ex.: período letivo criado) antes de liberar o acesso.

### PLT-05 — Gestão de feature flags
Flags com **3 escopos**: global (plataforma), por tenant, por perfil/persona. Usos: lançamento gradual de funcionalidades, módulos opcionais por escola (ex.: caixa físico, secretaria digital), gates da v2 (agentes). Toda mudança de flag é auditada. Flags **não** implementam tiers comerciais na v1 (fora de escopo — G2).

### PLT-06 — Impersonação auditada
Substitui a senha-mestre do legado (RQ-SEG-02). O operador de suporte pode assumir a visão de um usuário de um tenant mediante: permissão específica, justificativa obrigatória, prazo de expiração da sessão impersonada e registro integral (quem, quem-alvo, quando, ações realizadas). O usuário impersonado é identificável na trilha de auditoria (`ator = suporte X como usuário Y`). A escola pode visualizar o histórico de impersonações no seu tenant.

### PLT-07 — Monitoramento operacional por tenant
Painel do operador com: estado das filas e rotinas agendadas por tenant (últimas execuções, erros, itens pendentes — ver PLT-12), saúde das integrações (conector fiscal, assinatura, e-mail/SMS/push), versão mínima do app em vigor e tenants suspensos. Ações: reprocessar item de fila, reexecutar rotina, pausar rotina por tenant.

### PLT-08 — Fora da v1 (registrado)
Billing do SaaS (cobrança das escolas), planos/tiers comerciais, métricas comerciais de uso, trial automatizado. Gestão comercial é manual na v1.

---

## 3. White-label por escola (G8)

### PLT-09 — Identidade visual do tenant
Configuração pela escola (com preview): **logomarca** (claro/escuro), **favicon**, **paleta de cores** (cor primária + derivadas geradas pela camada de tokens — doc 05 §3), nome de exibição. Aplicação: web administrativa, portais, e-mails (cabeçalho/rodapé dos templates), documentos PDF emitidos (cabeçalho), telas públicas (ficha de inscrição, agendamento de visita).

### PLT-10 — App iOS/Android com tema dinâmico
App único nas lojas. Ao identificar a escola no login (por subdomínio/código da escola), o app carrega logomarca, paleta e splash do tenant. **Sem** ícone/nome personalizado na loja (builds dedicados fora do produto — G8). O contraste e a legibilidade da paleta são validados na configuração (gate de a11y — doc 05).

---

## 4. Serviços transversais de plataforma

### PLT-11 — Framework de operações em lote (PIL-08)
Toda operação em massa do sistema usa o mesmo arcabouço: **(1) Filtrar → (2) Editar/parametrizar → (3) Simular → (4) Processar**. A simulação apresenta o resultado previsto item a item sem efetivar. O processamento é assíncrono (PLT-12), com log por item, contagem de sucesso/erro e reprocessamento dos falhos. Operações irreversíveis (ex.: finalizar turmas — doc 02) exigem confirmação explícita com aviso. Consumidores na v1: renovação de matrículas, promoção entre séries, inativação de enturmações, alteração de notas, frequência em lote, geração de títulos, envio de mensagens/cobranças, RPS/NFS-e, envio de convites de acesso, importações (alunos, colaboradores).

### PLT-12 — Fila de processamento visível
Jobs assíncronos com estado consultável pelo usuário da escola (e pelo operador): inserido → processando → concluído/erro, com data/hora, contexto (entidade alvo), log e erro por item, retry manual e em lote. As ~20 rotinas agendadas herdadas da base executam como **serviços nomeados por domínio** nesta infraestrutura (anti-obstáculo da v2 — doc 08): Cobrança, Conciliação, Bolsas/Descontos, Fiscal, Matrícula, Contratos, Presença, Ocupação, Funil.

### PLT-13 — Eventos de domínio (PIL-12)
Catálogo inicial de eventos publicados internamente: `matricula.efetivada`, `matricula.cancelada`, `requerimento.aprovado`, `contrato.assinado`, `titulo.gerado`, `titulo.vencido`, `titulo.liquidado`, `presenca.fechada`, `conteudo.publicado`, `nota.confirmada`, `boletim.publicado`, `termo.aceito`. Consumidores na v1: notificações, caixa de saída, rotinas; na v2: agentes (doc 08).

### PLT-14 — Auditoria (PIL-07)
Modelo único de trilha: ator (usuário, sistema, *agente* — reservado), ação, entidade, registro, dados antes/depois (quando aplicável), justificativa (em overrides), data/hora, tenant/unidade. Telas de configuração e operações financeiras exibem acesso direto à trilha da entidade ("botão Auditoria"). Retenção e exportação da trilha por tenant.

### PLT-15 — Central de relatórios
Catálogo de relatórios por domínio com: filtros padronizados (período, unidade, situação), favoritos por usuário, fila de exportação (relatórios pesados geram arquivo via PLT-12), histórico de exportados. Relatórios da base preservados (alunos por unidade, pais/mães por faixa etária, exportação CSV de contatos — esta condicionada a consentimento, RQ-SEG-07).

### PLT-16 — Dashboard de KPIs acionáveis
Home da escola com indicadores que linkam para a lista-alvo: matrículas/cancelamentos/prospects no período (herdados da base), ocupação de turmas (vagas regulares × inclusivas), inadimplência em R$, títulos com pendência, contratos aguardando assinatura há mais de N dias (parâmetro; *14 dias (exemplo)*), funil de captação (ganhas/perdidas). Filtros por unidade e período.

### PLT-17 — Notificações de plataforma
Banner/aviso de tela inicial (mensagem do operador ou da escola, com vigência); centro de notificações internas por usuário (tipos, vínculo a entidade, lida/removida — herdado da base, doc 04 §5).

---

## 5. Administração de configurações pela escola

### PLT-18 — Parametrização em camadas
Precedência: **global (plataforma) → tenant → unidade → série×período → turma → diário** — a regra mais específica prevalece. Toda tela de parâmetro indica de qual camada o valor efetivo veio.

### PLT-19 — Cadastros auxiliares configuráveis (com seeds)
Vocabulários da escola: parentescos, funções de colaborador, tipos de interação comercial, meios de atendimento/conhecimento, tipos de refeição, tipos de ocorrência (com categoria e flags — doc 04), motivos de inativação/cancelamento (seeds: fim de ciclo, solicitação do responsável, não renovação), situações de aluno (doc 02 §5 — seeds protegidos), cotas de cobrança (doc 03), classificações contábeis, formas de entrega de documento. Validação de duplicidade/normalização na entrada (lição dos dados sujos da referência analisada).

### PLT-20 — Campos dinâmicos
Extensão de cadastro sem migração: campos customizados por escola para aluno e unidade (tipo, obrigatoriedade, visibilidade em portal/fichas). Exibidos no hub do aluno e disponíveis em relatórios.

### PLT-21 — Sequenciais nomeados
Numeradores configuráveis por contexto (matrícula por unidade/ano, títulos, requerimentos), com prefixo e valor inicial. Seeds reproduzem a numeração atual da base na migração.

---

## 6. Invariantes do domínio de plataforma

1. Nenhum dado de um tenant é acessível a outro tenant, em nenhuma camada (consulta, relatório, exportação, API).
2. Tenant suspenso não autentica usuários da escola; dados permanecem íntegros.
3. Seeds de sistema não podem ser removidos nem ter sua semântica de sistema alterada pela escola.
4. Toda impersonação tem autor, alvo, justificativa, prazo e trilha completa — sem exceção.
5. Operação em lote sem simulação prévia não existe; lote irreversível exige confirmação explícita.
6. Mudança de feature flag, parâmetro ou cadastro auxiliar gera registro de auditoria.
7. A camada de parametrização mais específica prevalece, e a origem do valor efetivo é sempre identificável.
8. Subdomínio é único na plataforma; alteração de subdomínio preserva redirecionamento temporário.
