# 05 — Design System e Qualidade de UI

> Extraído da spec Janus em `_janus_spec/` (318 telas capturadas, 56 arquivos CSS analisados,
> 1.232 tokens primitivos + 305 aliases inferidos).
> Fontes: `design-system/design-system.md`, `primitive.md`, `semantic.md`, `composite.md`,
> `tokens.dtcg.json`, `audits/a11y.md`, `audits/css-health.md`, `ui/SDD.md` e specs de tela.

---

## 1. Inventário do design system atual

### 1.1 Contexto estrutural (achado central)

O "design system" atual **não é um sistema — é a sobreposição de pelo menos 5 ecossistemas de UI coexistindo no mesmo produto**, identificáveis pelos prefixos de tokens e seletores:

| Camada | Evidência | Papel |
|---|---|---|
| **Mantine** (React) | tokens `mantine-*`, seletores `.m_*` | UI nova/moderna (app shell, inputs, modais, stepper) |
| **Tema "ngv" / isaac** (fintech) | `:root,[data-theme=ngv]`, `global-colors-isaac-branding-*` | módulo financeiro integrado (cobrança isaac) |
| **Metabase embarcado** | tokens `mb-color-*` | dashboards/BI embutidos |
| **Bootstrap legado** | `.form-control`, `.col-md-6`, `.nav-link`, `.progress-bar` | telas CRUD antigas |
| **Tailwind + libs avulsas** | `tw-*`, CKEditor (`.cke_*`), Monaco editor, AG Grid (`ag-*`), react-datepicker, Swagger UI | editores, planilhas, docs de API |

Consequência: **4 famílias tipográficas concorrentes, 3 paletas de marca, 2+ escalas de espaçamento e 1.267 "buckets" de cor distintos**. O blueprint do novo sistema deve consolidar tudo isso em **uma única fonte de tokens** (DTCG canônico já existe como ponto de partida: `tokens.dtcg.json`, com top-level `color`, `dimension`, `font`, `tokens`).

### 1.2 Paleta de cores

**Cores de marca (camada isaac/ngv — a mais bem estruturada, recomendada como base):**

| Token | Valor | Uso |
|---|---|---|
| `color.brand-primary` | `#0055d4` | azul principal (ações, links) |
| `color.brand-secondary` | `#ff6500` | laranja de acento |
| `color.brand-tertiary` | `#146ef5` | azul claro |
| `color.brand-quaternary` | `#f2a516` | âmbar |
| `colors-isaac-branding-main` | `#3d4ed7` | marca isaac (financeiro) |
| `colors-isaac-branding-goiabinha` | `#f9756b` | marca isaac secundária |

**Rampas semânticas** — cada estado tem 6 degraus (`light → soft → subtle → mid → intense → deep`):

| Estado | light | soft | subtle | mid (base) | intense | deep |
|---|---|---|---|---|---|---|
| **success** | `#ecf4ec` | `#d1e4d2` | `#acceaf` | `#3f8d44` | `#36783a` | `#2d6430` |
| **danger** | `#fbe9e8` | `#f5cbc9` | `#eda29d` | `#d5271c` | `#b52118` | `#791610` |
| **warning** | `#fef6e8` | `#fce9c7` | `#f9d89b` | `#f2a516` | `#ac7510` | `#6d4a0a` |
| **info / action** | `#e6eefb` | `#c2d6f5` | `#91b6ed` | `#0055d4` | `#0048b4` | `#003c97` |
| **accent** | — | `#ffdac2` | `#ff812e` | `#ff6500` | — | — |

**Neutros:** `neutral-white #fff` · `light #f7f8fa` · `soft #e6eaef` · `subtle #dadfe7` · `mid #9fa9b9` · `deep #69707a` · `dark #383b41` · `black #151719`.

Em paralelo, a camada Mantine mantém **rampas 0–9 para 13 cores** (blue, red, green, yellow, orange, teal, cyan, grape, violet, indigo, pink, lime, gray, dark) com aliases derivados (`*-filled` = degrau 8, `*-filled-hover` = 9, `*-outline`/`*-text` = 4, `*-light-color` = 3) e `mantine-primary-color-* → blue`. Há ainda **suporte parcial a dark mode** (`:root[data-mantine-color-scheme='dark']` remapeia body→dark-7, text→dark-0 etc.) — só na camada Mantine, não no resto.

### 1.3 Tipografia

| Token | Valor | Camada |
|---|---|---|
| `font.default` | `"Inter", sans-serif` | ngv/isaac (texto corrente) |
| `font.brand` | `Nunito, sans-serif` | ngv/isaac (marca/títulos) |
| `font.system` | `Roboto, sans-serif` | ngv/isaac |
| `mantine-font-family` | system stack (-apple-system, Segoe UI, Roboto…) | Mantine (corpo e headings) |
| `font.monospace` / `monaco-monospace-font` | SFMono/Menlo/Consolas… | código/editores |

**Escala de tamanhos (Mantine):** xs `0.75rem` · sm `0.875rem` · md `1rem` · lg `1.125rem` · xl `1.25rem` — todos multiplicados por `--mantine-scale` (escala global de zoom acessível, padrão que vale a pena manter).

**Pesos:** 300 (light), 400 (normal), 500 (medium/brand-regular), 600 (semibold), 700 (bold), 800 (brand-black). Inconsistência observada: `font.system-bold = 500` e `font.system-black = 700` (nomenclatura mente sobre o valor).

### 1.4 Espaçamento, raio, sombra

- **Espaçamento (Mantine):** xs `0.625rem` · sm `0.75rem` · md `1rem` · lg `1.25rem` · xl `2rem` (× `--mantine-scale`).
- **Escala dimensional (ngv):** `dimension.0–20` em incrementos de `0.25rem` (0 a 5rem) — escala tipo Tailwind 4px-grid.
- **Border radius (ngv):** xs `2px` · s `4px` · m `8px` · l `12px` · xl `16px` · xxl `24px` · xxxl `32px` · full `9999px`. Mantine: default `0.25rem`, md `0.5rem`, lg `1rem`, xl `2rem`. Pílulas/badges/avatares usam radius "infinito" (`62.5rem`). Inputs Metabase: `8px`; botões: `6px` — mais um exemplo de duplicidade.
- **Sombras:** duas escalas — `mantine-shadow-{xs..xl}` (multi-layer rgba suave) e `shadow-2..6` (ngv, elevação progressiva). Ambas sutis, com alpha ≤ 0.17.

### 1.5 Componentes identificados via tokens (camada Mantine + Metabase)

Os aliases em `semantic.md` revelam o catálogo de componentes realmente em produção: **Accordion, ActionIcon, Alert, AppShell, Avatar, Badge, Breadcrumbs, Burger, Button, Card/Paper, Checkbox, Chip, ColorPicker/ColorInput, Combobox/Select, Container, DatePicker/Calendar, Dialog, Divider, Drawer, Indicator, Input (com left/right sections, estados disabled/error/focus), Kbd, List, Loader, Modal, Notification/Toast, NavLink, Pagination, Pill, PinInput, Popover, Progress, Radio, SegmentedControl, Slider, Stepper, Switch, Tabs, Table (striped/hover/border), Timeline, Tooltip**.

### 1.6 Padrões de layout (shell da aplicação)

| Elemento | Token/evidência | Valor |
|---|---|---|
| App shell | Mantine AppShell (`app-shell-border-color`, offsets para header/navbar/aside/footer) | bordas `gray-3` |
| Header | `tokens.header-height` | `48px` |
| Sub-header | `tokens.app-subheader-height` | `48px` |
| Sidebar de dados | `tokens.data-sidebar-width` | `320px` |
| Sidebar de configurações | `tokens.settings-sidebar-width` (alias `tokens.right`) | painel direito |
| Navbar principal | `#MainNavbar` com `z-index: 2147483641` (!) | topo fixo |
| Breadcrumbs | `breadcrumbs-color` + `breadcrumb-page-color`, divider `0.75em` | trilha de navegação |
| Navegação lateral | NavLink com estado ativo `nl-bg` = primary-light, hover = primary-light-hover | item ativo destacado por fundo |

Padrão recorrente nas 318 telas (tags do SDD): **list-view com busca + paginação + formulário de filtro**, **detail-view com sub-abas por entidade** (aluno: detalhe/cobranca/desconto/enturmacao/ficha_medica/historico/observacoes; colaborador: detalhe/disciplinas/escolaridade/obs/quadro_horarios; responsável: detalhe/alunos_vinculados/observacoes).

---

## 2. Linguagem visual semântica

Como o sistema comunica estados hoje:

- **Sucesso / dinheiro recebido:** rampa verde (`success-*`); no Metabase, `tokens.change → mb-color-success` (variação positiva em verde).
- **Erro / inadimplência / remoção:** rampa vermelha (`danger-*`); inputs com `[data-error]` trocam borda para `mantine-color-error` (= red-8). Telas de cobrança dependem dessa rampa para sinalizar títulos vencidos.
- **Pendência / atenção:** rampa âmbar (`warning-*`, base `#f2a516`) — usada em filas de pendência (`integracoes--pendencias`, `fila_processamento`, `titulos_isaac_fila_registro`).
- **Informação / ação primária:** azul (`info-*` ≡ `action-*` — mesmos hexes, o azul de ação É o azul de informação). Alert padrão = `primary-color-light` (fundo azul claro + texto azul).
- **Padrão de par fundo claro + texto escuro:** badges/alertas usam `*-light` como fundo e `*-deep`/`*-intense` como texto — convenção sólida para contraste, a preservar.
- **Estados interativos:** filled (degrau 8) → hover (degrau 9); item ativo de navegação = fundo `primary-light`; disabled = cinza dedicado; focus de input = borda `primary-filled`; placeholder = cinza médio dedicado.
- **Marca de contexto:** o módulo financeiro isaac troca a identidade (azul `#3d4ed7` + "goiabinha" `#f9756b`), criando uma segunda linguagem visual dentro do mesmo produto — fonte de inconsistência percebida pelo usuário.

**Lacuna importante:** não existe rampa semântica para estados de domínio escolar (matrícula ativa/trancada/transferida, aprovado/recuperação/reprovado, bolsista). Hoje isso é improvisado com as 13 cores Mantine ad hoc. O novo sistema deve definir **tokens semânticos de domínio** (`status.matricula.ativa`, `status.financeiro.inadimplente`, `status.academico.recuperacao`...) mapeados às rampas base, em vez de cores diretas.

---

## 3. Catálogo de componentes compostos que o novo sistema precisa

Derivado do cruzamento tokens × 318 telas × tags do SDD:

### 3.1 Estruturais
1. **App Shell** — header fixo 48px + sub-header contextual 48px + sidebar de navegação colapsável (burger em mobile) + área de conteúdo; painéis laterais auxiliares (320px dados / settings à direita). Transições controladas por token.
2. **Navegação lateral hierárquica** — NavLink com níveis indentados, estado ativo por fundo, descrição secundária, ícone + chevron.
3. **Breadcrumbs** — trilha com página atual em peso/cor distintos.
4. **Tabs** — usadas massivamente nos detail-views (ficha do aluno com 7 abas, diário, contas a receber com "modo de exibição"). Precisa de variante "modo de exibição" (alterna tabela/cards) e abas roláveis.

### 3.2 Dados
5. **Data table com filtros, busca e paginação** — o componente mais crítico (padrão `list-view + has-search + has-pagination + has-form` em dezenas de telas). Comportamentos: striped/hover, filtro por status na URL (`?status=ativo`), paginação server-side, linha clicável para detalhe, ações em lote.
6. **Planilha editável (grid)** — AG Grid hoje: `planilha_notas_faltas`, `planilha_avaliacao_competencia`, frequência em lote. Edição inline de notas/faltas com navegação por teclado.
7. **Tree view com checkboxes** — `plano_contas` (react-checkbox-tree hoje), grade curricular, perfis de permissão.
8. **Dashboard embarcado / KPI cards** — hoje Metabase embutido; cards de indicador com variação (`change` verde / `remove` vermelho).
9. **Query builder visual** — `gerador_consultas--criador_de_consultas` (relatórios ad hoc por usuário final).

### 3.3 Entrada e fluxo
10. **Formulários compostos** — input com seções esquerda/direita, label/description/error, máscaras (CPF, moeda), selects com busca (Combobox), DatePicker com calendário, PinInput, Switch/Checkbox/Radio, ColorPicker (personalização de tema da escola).
11. **Wizard com Stepper** — tokens de Stepper completos + telas `matricula_online`, `confirmacao_matricula`, `procedimento_matricula`, `turmas--cadastrar`: fluxo de matrícula multi-etapa (dados → responsáveis → plano de pagamento → contrato → confirmação).
12. **Editor rich-text de templates** — CKEditor hoje (`comunicados--cadastrar`, `texto_personalizado`, `contrato`, `termo_consentimento`): comunicados, contratos e termos com merge-fields. Inclui editor de código (Monaco) para casos avançados.
13. **Calendário escolar** — visão mensal com eventos, feriados e horários de aula (hoje react-datepicker esticado além do propósito).

### 3.4 Feedback e overlay
14. **Modal / Drawer / Dialog** — tamanhos tokenizados, drawer lateral para edição rápida. Atenção: hoje há modal via iframe (`#ug-modal-frame`) — eliminar.
15. **Notification/Toast + Alert** — variantes por estado semântico; central de notificações.
16. **Badge / Pill / Chip de status** — badge pílula (radius full, fundo filled) para status de matrícula, situação financeira, status de boleto/registro. Deve nascer mapeado aos tokens semânticos de domínio (§2).
17. **Progress + Loader + Indicator** — barras de progresso nomeadas (fila de processamento, roteiro de treinamento), skeleton/loading.
18. **Avatar + Timeline** — foto do aluno/colaborador; timeline para histórico escolar e auditoria.

---

## 4. Dívidas a NÃO repetir — requisitos de qualidade para o novo sistema

### 4.1 Acessibilidade (audit axe-core: 53 violações — 46 críticas, 7 sérias — em 315 de 318 telas)

Distribuição por critério WCAG: **4.1.2** (35), **1.3.1** (16), **1.1.1** (1), **2.4.4** (1), **3.1.1** (1).

| Categoria | Regra axe | WCAG | Escala do problema no legado | Requisito para o novo sistema |
|---|---|---|---|---|
| **Formulários sem rótulo** | `label` (29 ocorrências, maior ofensor) | 4.1.2 | inputs `.form-control` sem label em telas inteiras | **RQ-A11Y-01:** Todo campo de formulário DEVE ter `<label>` programaticamente associado (ou `aria-label`/`aria-labelledby`). O componente Input do DS não deve aceitar render sem rótulo acessível (prop obrigatória; lint/CI bloqueante com axe). |
| **Estrutura ARIA inválida** | `aria-required-parent` (14), `aria-allowed-attr` (2) | 1.3.1 / 4.1.2 | tabs Bootstrap com `role="tab"` fora de `tablist` em TODAS as telas com abas; `aria-haspopup="dialog"` em div de popover presente em ~300 telas (componente global de troca de unidade) | **RQ-A11Y-02:** Componentes compostos (Tabs, Menu, Combobox, Popover) DEVEM implementar os padrões WAI-ARIA APG completos. Um único componente global defeituoso contaminou 300+ telas — testar componentes compartilhados com rigor proporcional ao seu raio de alcance. |
| **Controles sem nome acessível** | `button-name` (1, global), `link-name` (1, global), `aria-toggle-field-name`, `aria-progressbar-name` | 4.1.2 / 2.4.4 / 1.1.1 | botões e links icon-only do menu principal sem texto discernível — replicados em todas as telas | **RQ-A11Y-03:** Todo ActionIcon/link icon-only DEVE exigir `aria-label`. Progressbars e toggles DEVEM ter nome acessível. Ícones de fonte DEVEM ser `aria-hidden` com rótulo textual paralelo. |
| **Listas malformadas** | `list`, `listitem` | 1.3.1 | `<ul> > div > li` | **RQ-A11Y-04:** Markup semanticamente válido; proibir wrappers de layout dentro de listas (resolver com CSS). |
| **Iframes e idioma** | `frame-title` (global), `html-has-lang` | 4.1.2 / 3.1.1 | iframe de modal sem título em todas as telas; docs sem `lang` | **RQ-A11Y-05:** `<html lang="pt-BR">` em toda página; evitar arquitetura por iframes — se inevitável (BI embarcado), `title` obrigatório. |

**Requisito transversal RQ-A11Y-06:** pipeline de CI com axe-core em browser real (a auditoria atual rodou em jsdom — falsos negativos para checks visuais como contraste de cor; o novo sistema deve auditar contraste 1.4.3 explicitamente, hoje não coberto). Meta: **zero violações critical/serious por tela** como gate de release.

### 4.2 Saúde de CSS (6 métricas em estado de erro de 7 medidas)

| Métrica | Legado | Threshold | Requisito para o novo sistema |
|---|---|---|---|
| `!important` | **10.462** declarações | 20 | **RQ-CSS-01:** Proibir `!important` exceto em utilities geradas (orçamento ≤20, lint stylelint bloqueante). A causa-raiz (guerra de especificidade entre 5 frameworks empilhados) desaparece com stack única + CSS layers/escopo por componente. |
| Buckets de cor | **1.267** | 15 | **RQ-CSS-02:** Toda cor DEVE referenciar token DTCG; zero hex literais em código de componente. Paleta-alvo: ~15 buckets (rampas semânticas + neutros + marca). |
| Breakpoints | **26** distintos (1024px, 1199.98px, 1200px, 120em, 1344px, 1400px, 1440px, 1536px...) | 6 | **RQ-CSS-03:** Grade única de ≤6 breakpoints tokenizados; proibir media queries com valores literais. |
| z-index | **67 valores**, máx **2.147.483.647** (datepicker, swagger; navbar 2.147.483.641) | 10 / 999 | **RQ-CSS-04:** Escala de elevação tokenizada (~6 camadas: base, dropdown, sticky, overlay, modal, toast; máx ≤999). Overlays via portal + um gerenciador de stacking — nunca "z-index nuclear". |
| Especificidade máx | **220** (média 20) | 100 | **RQ-CSS-05:** Seletores planos (CSS Modules/vanilla-extract/data-attrs como o Mantine já faz); especificidade máx ≤ (0,3,0); proibir seletores com ID e cadeias profundas. |
| Volume | 56 arquivos, 32.863 regras, 79.269 declarações | — | **RQ-CSS-06:** Stack de UI única (não 5); orçamento de CSS por rota; tree-shaking de estilos. Para BI e rich-text embarcados, isolar via Shadow DOM/iframe sandboxed com tema injetado por tokens. |

### 4.3 Dívidas de arquitetura de tokens

- **RQ-TOK-01:** Hierarquia de 3 camadas obrigatória — primitive → semantic → component (a v1 do Janus já mapeou primitive+alias; o novo sistema nasce com a camada componente definida).
- **RQ-TOK-02:** Um único tema raiz com suporte a white-label (escolas personalizam cor da marca — hoje há ColorPicker para isso) e **dark mode completo** (hoje só a camada Mantine tem).
- **RQ-TOK-03:** Nomenclatura honesta e única (eliminar casos como `font.system-bold = 500`, `border-radius-*` com type `color`, e os duplos `info` ≡ `action`).
- **RQ-TOK-04:** Tokens semânticos de domínio escolar (status de matrícula, situação financeira, resultado acadêmico) como contrato entre design e produto — badges/alertas consomem o token de domínio, nunca a cor crua.
