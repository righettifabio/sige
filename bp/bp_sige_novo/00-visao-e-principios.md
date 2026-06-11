# 00 — Visão e Princípios do SIGE

> **Blueprint funcional do novo SIGE — documento 00 de 10.** Define posicionamento, escopo, personas, pilares e requisitos de qualidade. Os documentos 01–06 detalham os domínios; 07 traz diretrizes tecnológicas (não-vinculantes); 08 a visão v2; 09 a rastreabilidade da base; 10 o backlog.
>
> Fontes vinculantes: `bp/bp_sige_atual/bp_sige.md` (base fixa — nada do que ela faz é descontinuado), `analise_comparativa.md` (matriz + decisões §11) e `inventario_api_atual.md`.

---

## 1. O que é o SIGE

O SIGE é uma **plataforma SaaS multi-escola de gestão educacional** para instituições de ensino particulares brasileiras, **da creche ao ensino médio**, cobrindo o ciclo completo:

```
captação → matrícula online → contrato (assinatura eletrônica) → enturmação
→ rotina diária (infantil) / diário de classe, notas e boletim (seriado)
→ cobrança (boleto CNAB + PIX) → caixa e retaguarda contábil → NFS-e
→ comunicação com as famílias → compliance (Educacenso, LGPD)
```

Cada escola é um **tenant** com subdomínio próprio, marca própria (white-label) e configuração própria — estrutura acadêmica, situações de aluno, regras de cobrança, permissões de portal. A plataforma opera dois planos de uso:

1. **Uso da escola** — web administrativa, portais (responsável, aluno, professor) e App iOS/Android (desenvolvido em Flutter/Dart) para famílias e colaboradores.
2. **Operação do SaaS** — backoffice do operador da plataforma: onboarding de escolas, feature flags, impersonação auditada, monitoramento.

### 1.1 Proposta de valor

- **Para a escola de educação infantil/creche:** o diário de rotina mais profundo do mercado (refeições com escala de consumo, sono, medicação ministrada por horário, troca de fraldas, presença parcial), acompanhado pela família em tempo quase real.
- **Para a escola seriada (fundamental/médio):** motor de avaliação configurável (fases, fórmulas, aprovação por nota E frequência), boletim e histórico escolar, sem hardcodar bimestres.
- **Para qualquer escola:** cobrança automatizada de ponta a ponta, formalização contratual com assinatura eletrônica e implementação automática de mudanças, comunicação multicanal com prova de entrega, e privacidade familiar granular.

### 1.2 Diferenciais estruturais (herdados e elevados a requisito)

| # | Diferencial | Origem |
|---|---|---|
| DF-1 | **Rotina diária infantil integral** com presença parcial e acompanhamento em tempo quase real | Base (MANTER literal) |
| DF-2 | **Requerimentos formais** com workflow de aprovação, assinatura eletrônica multi-signatário e **implementação automática na data prevista** | Base (MANTER literal) |
| DF-3 | **Privacidade familiar granular**: atribuições por parente×aluno + dupla aprovação de fotos + termo de uso de imagem | Base + acréscimo LGPD |
| DF-4 | **Caixa de saída**: trilha de auditoria universal de toda comunicação enviada às famílias | Acréscimo (análise) |
| DF-5 | **Configurabilidade profunda por escola com seeds protegidos**: a escola monta sua estrutura; as invariantes do sistema não podem ser quebradas | Decisões C1/C6 |
| DF-6 | **Regra de Cobrança como conceito de 1ª classe**: o que cobrar × de quem × quando × quanto × recorrência, configurável | Decisão G7 |

---

## 2. Escopo da v1

### 2.1 Módulos (visão de mapa — detalhe nos docs 01–06)

1. **Plataforma & Administração do SaaS** (doc 01) — multi-tenancy, onboarding, white-label, flags, impersonação.
2. **Pessoas & Acesso** (doc 02 §2) — Pessoa única, parentes/atribuições, colaboradores, autenticação, papéis.
3. **Estrutura Acadêmica** (doc 02) — configurável por escola: períodos, cursos, agrupamentos etários/séries, turmas, disciplinas/grade (por segmento).
4. **Matrícula & Captação** (doc 02) — funil, ficha pública, matrícula online, procedimentos documentais, requerimentos, contratos, renovação.
5. **Rotina Diária & Pedagógico** (doc 02) — diário de rotina (infantil), diário de classe, frequência (2 modos), motor de avaliação, boletim, histórico.
6. **Financeiro** (doc 03) — títulos/cotas, Regra de Cobrança, CNAB+PIX, régua, negociação, caixa físico, contas a pagar, contábil, NFS-e (hub), impedimentos.
7. **Comunicação & Portais** (doc 04) — comunicados, circulares, ocorrências, fotos, notificações/push, caixa de saída, portais por persona, App iOS/Android, LGPD/termos.
8. **Relatórios & Indicadores** (docs 01/03/04) — central de relatórios, KPIs acionáveis, Educacenso.
9. **API & Integrações** (doc 06) — contrato unificado do app, API pública de parceiros, conectores (assinatura, fiscal, bancário, push, e-mail/SMS).

### 2.2 Fora da v1 (mapeado — ver doc 08)

Chat escola↔família; BI embarcado; biblioteca; filantropia/CEBAS; ranking de alunos; catraca/acesso físico; Google for Education; gerador de consultas; conector de garantidora de recebíveis; registro online de boleto via API bancária; billing/tiers comerciais do SaaS; builds white-label do app por escola; **SIGE Intelligence** (camada de agentes — v2, com anti-obstáculos obrigatórios na v1, ver doc 08 §2).

---

## 3. Personas

| # | Persona | Característica-chave | Canal |
|---|---|---|---|
| 1 | **Operador do SaaS** | Administra a plataforma: tenants, flags, modelos de estrutura, suporte com impersonação auditada | Backoffice |
| 2 | **Administrador da escola** | Acesso total ao tenant; configura estrutura, situações, regras, permissões, marca | Web |
| 3 | **Secretaria** | Opera cadastros, matrículas, requerimentos, comunicação; trabalha centrada no hub do aluno | Web |
| 4 | **Financeiro/Tesouraria** | Títulos, régua, negociação, caixa físico (abertura/fechamento nominal), contábil, NFS-e | Web |
| 5 | **Coordenação/Direção** | Aprova requerimentos, conteúdo e notas; assina documentos; consome KPIs | Web + App |
| 6 | **Professor / colaborador de sala** | Infantil: registra a rotina diária. Seriado: diário de classe, chamada, notas (acesso derivado da alocação) | App + Portal |
| 7 | **Responsável / parente** | Acompanha o(s) aluno(s) conforme atribuições; responsável legal tem acesso pleno aos dados do filho (regra fixa) | App + Portal |
| 8 | **Aluno** (segmentos com idade para tal) | Boletim, atividades, quadro de horários, carteirinha | Portal |
| 9 | **Prospect** | Interessado externo; formulários públicos (visita, ficha de inscrição) | Web pública |
| 10 | **Parceiro de API** | Não-humano; consome a API pública com OAuth2 + escopos | API |

Um mesmo usuário pode acumular perfis (ex.: colaborador que é parente) e **alterna a visão** pelo seletor de perfil no AppHeader (doc 05) — preservando a troca de perfil que a base já tem.

---

## 4. Pilares (princípios de produto e modelagem)

| ID | Pilar | Enunciado |
|---|---|---|
| PIL-01 | **Base preservada** | Tudo que o sistema atual faz existe na v1 — literalmente ou como seed/default de mecanismo generalizado. Auditado no doc 09. |
| PIL-02 | **Multi-tenancy em 3 níveis** | Instituição (tenant/escola) → Unidade (campus) → Empresa (CNPJ de faturamento). Tenant resolvido por **subdomínio/claim, nunca por path de URL**. Unidades podem estender outras (herança de estrutura, herdada da base). |
| PIL-03 | **Configurável com seeds protegidos** | Vocabulários, estruturas e regras são configuráveis por escola; os comportamentos críticos da base existem como registros seed de sistema, não removíveis, com efeitos declarativos. |
| PIL-04 | **Situação como meta-modelo** | "Situação do aluno na turma" é cadastro da escola mapeado a domínios fixos (sistema: ativo/inativo; acadêmico; Educacenso) + flags de comportamento (digita nota, assina contrato, mantém acesso, corta cobrança). Quase todo módulo filtra por ela. |
| PIL-05 | **Aprovação humana nos fluxos sensíveis** | Conteúdo às famílias passa por aprovação (dupla, no caso de fotos); notas passam por confirmação; contratos por assinatura. Esse gate é generalizado e — na v2 — é o que os agentes de IA reutilizam. |
| PIL-06 | **Impedimento como mecanismo transversal configurável** | Pendências (financeiras, documentais, de ocorrência) geram bloqueios **por canal**, totalmente configuráveis pela escola, com badges, overrides auditados e disclaimers legais exibidos pelo produto (Lei 9.870/1999). Regra fixa inviolável: o responsável legal nunca perde acesso aos **dados** do próprio filho. |
| PIL-07 | **Auditoria onipresente** | Toda entidade sensível tem trilha (quem, o quê, quando); todo override registra autor + justificativa + data; toda comunicação enviada é logada na caixa de saída. Autoria não-humana (agente) já prevista no modelo de log. |
| PIL-08 | **Operações em lote com simulação** | Framework único: filtrar → editar → **simular** → processar, com fila assíncrona visível, log por item e retry. Nenhum lote sem simulação prévia e trilha. |
| PIL-09 | **Invariantes na escrita** | Regras garantidas transacionalmente no momento da gravação — o novo sistema não possui "rotinas de correção de dados" (a existência delas no legado é tratada como defeito, §12.5 da base). |
| PIL-10 | **Soft delete e ciclo de vida** | Nada se apaga fisicamente: inativa-se com motivo (cadastro), comentário e, quando aplicável, destino (ex.: escola de transferência). Reativação onde a base prevê (cancelamento não finalizado). |
| PIL-11 | **Encadeamento temporal** | `próximo período` e `próxima série/agrupamento` como ligações explícitas que habilitam renovação e promoção em lote; no infantil, a progressão derivada da idade permanece como regra automática. |
| PIL-12 | **Eventos de domínio** | Fatos relevantes (matrícula efetivada, boleto vencido, presença fechada, contrato assinado…) são publicados como eventos internos — base das automações da v1 e dos agentes da v2 (anti-obstáculo G11). |
| PIL-13 | **Dinheiro em centavos** | Valores monetários são inteiros em centavos em todo o sistema (herdado da base); exibição com conversão. |
| PIL-14 | **Jornadas, não telas-ilha** | Fluxos de negócio (matrícula, cobrança, fechamento de notas) são wizards/checklists de primeira classe, com estados empty/erro/carregando explícitos. |

---

## 5. Requisitos de qualidade

### 5.1 Segurança e LGPD (RQ-SEG)

| ID | Requisito |
|---|---|
| RQ-SEG-01 | Autenticação com tokens **assinados, com expiração verificada, renovação e revogação granular**. A validação ocorre **antes** de qualquer execução da ação (defeito grave do legado: middleware validava depois — requisições inválidas produziam efeitos colaterais). |
| RQ-SEG-02 | **Proibido**: senha-mestre, credenciais/segredos no código ou em configuração versionada, chaves de serviço hardcoded. Gestão segura de segredos. Suporte assistido somente via **impersonação auditada** (permissão, trilha, prazo). |
| RQ-SEG-03 | Provisionamento de acesso por **convite com definição de senha pelo usuário** (nunca senha derivada de dados pessoais). Política de senha moderna (sem teto curto de caracteres). |
| RQ-SEG-04 | **Zero PII em URLs**, documentação pública de API ou exemplos. Tenant fora do path. Recursos binários (PDFs de boleto/circular, fotos) **sempre autorizados por sessão/token + escopo** — nunca públicos por ID sequencial (defeito do legado). |
| RQ-SEG-05 | Endpoints de **dados sensíveis segregados** com escopo de autorização próprio e auditoria por token. CPF mascarado em telas de consulta. |
| RQ-SEG-06 | Analytics somente com consentimento e **sem session-replay** em telas com PII de menores. Telemetria anonimizada com base legal explícita. |
| RQ-SEG-07 | Termos de consentimento LGPD e de uso de imagem por escola, com **aceite datado por titular**; fluxos que exibem os termos: matrícula online, reserva, captação, alteração cadastral; exportações de contatos condicionadas ao consentimento. |
| RQ-SEG-08 | Sem rotas de desenvolvimento em produção; sem SQL ad-hoc exposto a usuário final. |

### 5.2 Datas, anos e parametrização (RQ-PAR)

| ID | Requisito |
|---|---|
| RQ-PAR-01 | **Nenhuma data, ano letivo ou identificador de negócio fixado em código** (legado tinha ano 2024/2026 e listas de IDs hardcoded). Tudo deriva de cadastro/parametrização. |
| RQ-PAR-02 | Feriados e calendário 100% administráveis, com efeitos tipados (financeiro etc.) e cópia entre períodos. |
| RQ-PAR-03 | Limites operacionais (tamanhos de lote, paginação) são parâmetros de plataforma, não constantes. |

### 5.3 Interface e acessibilidade (RQ-UI) — detalhe no doc 05

Stack única de UI sobre shadcn/ui com tokens em 3 camadas; **zero violações critical/serious de a11y como gate de release**; `lang="pt-BR"`; todo controle com rótulo acessível; dark mode completo; estados empty/erro/carregando distintos; governança de rotas e nomenclatura.

### 5.4 Localização e operação (RQ-LOC)

Português do Brasil; fuso de Brasília; DD/MM/AAAA e HH:MM; moeda Real (centavos); multiunidade com segregação de dados e permissões; verificação de versão mínima do app por plataforma; deep links Android/iOS; disponibilidade prioritária para a rotina de sala em horário letivo.

---

## 6. Convenções deste blueprint

- **IDs estáveis** por domínio: `PLT-xx` (doc 01), `PES-xx`/`ACA-xx` (doc 02), `FIN-xx` (doc 03), `COM-xx`/`POR-xx` (doc 04), `UI-xx` (doc 05), `API-xx`/`INT-xx` (doc 06). O doc 09 e o backlog (doc 10) referenciam esses IDs.
- **Marcadores**: `[LACUNA]` (fontes omissas), `[SUPOSIÇÃO]` (premissa declarada), `[SUGESTÃO ALÉM DAS FONTES]` (proposta sem lastro nas fontes), `(exemplo)` para valores ilustrativos — nenhum valor de exemplo é default do produto.
- **"Seed"**: registro criado pelo sistema na ativação do tenant, que reproduz comportamento da base; seeds de sistema são imutáveis/não removíveis.

---

## 7. Glossário unificado

| Termo | Significado no SIGE |
|---|---|
| **Tenant / Escola** | Instituição cliente do SaaS; isolamento de dados, subdomínio e marca próprios. |
| **Unidade** | Sede física da escola; pode estender outra (herda estrutura). |
| **Empresa** | CNPJ de faturamento; uma unidade pode faturar por múltiplas empresas. |
| **Pessoa** | Registro único de indivíduo ou organização (PF/PJ); base de alunos, parentes, colaboradores e fornecedores. |
| **Parente** | Pessoa vinculada a aluno(s) com parentesco e **atribuições** por aluno. |
| **Responsável legal** | Parente com acesso e poderes plenos sobre o aluno (regra fixa, não configurável). |
| **Responsável financeiro / pedagógico** | Papéis do vínculo: pagador dos títulos / destinatário pedagógico. |
| **Atribuição** | Permissão granular concedida a um parente sobre um aluno específico. |
| **Atendimento** | Conjunto acadêmico do aluno no período: unidade, curso, agrupamento/série, nível, turno, permanência, horário, turmas e serviços. |
| **Agrupamento** | Faixa etária em meses que classifica aluno/prospect automaticamente pela data de nascimento (segmento infantil). |
| **Série** | Ano escolar dentro de um curso (segmentos seriados); carrega serviço financeiro padrão e próxima série. |
| **Enturmação** | Vínculo aluno×turma×período — registro central de matrícula, situação, plano e responsáveis. |
| **Situação do aluno** | Estado configurável da enturmação, mapeado a domínios de sistema/acadêmico/Educacenso + flags. |
| **Diário de rotina** | Agregado do dia do aluno no segmento infantil: presença, refeições, sono, medicações, higiene, comunicados. |
| **Diário de classe** | Registro turma×disciplina×fase nos segmentos seriados: aulas, conteúdo, tarefas, chamada, notas. |
| **Presença parcial** | Permanência menor que o horário contratado no dia; carimba os registros do diário de rotina. |
| **Fase de nota** | Etapa avaliativa (bimestre, recuperação, exame) com fórmulas, datas-limite e roteamento. |
| **Requerimento** | Solicitação formal sobre a matrícula/atendimento, com workflow de aprovação, assinatura (quando contratual) e implementação automática. |
| **Título** | Documento de cobrança (generalização do boleto); pago por boleto CNAB, PIX ou caixa. |
| **Cota** | Classificação configurável do título; seeds: CMAE (mensalidade), CC (composição), CRV (reserva de vaga). |
| **Regra de Cobrança** | Entidade que define o que cobrar × de quem × quando × quanto × recorrência (G7). |
| **Régua de cobrança** | Regras de comunicação automática sobre títulos (offset de dias × canal × texto). |
| **Impedimento** | Bloqueio operacional configurável por canal derivado de pendências, com override auditado. |
| **Caixa de saída** | Log universal de toda comunicação enviada (canal, destinatário, status de entrega). |
| **Procedimento de matrícula** | Item documental exigido na matrícula, com prazo, obrigatoriedade e upload. |
| **Seed** | Registro de sistema que reproduz comportamento da base; protegido contra remoção. |
| **Backoffice do SaaS** | Interface do operador da plataforma (onboarding, flags, impersonação, monitoramento). |
| **SIGE Intelligence** | Camada transversal de agentes (determinísticos + LLM) — v2, doc 08. |
