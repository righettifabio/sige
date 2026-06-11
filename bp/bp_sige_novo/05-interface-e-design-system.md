# 05 — Interface e Design System

> Documento 05 de 10. Define a biblioteca-base, a arquitetura de tokens, a shell de 5 áreas, os componentes compostos, o white-label visual, o dark mode e os gates de qualidade (acessibilidade e CSS). IDs: `UI-xx`. Requisitos de qualidade herdados: RQ-UI, RQ-A11Y, RQ-CSS, RQ-TOK.
>
> O design system corrige as dívidas observadas na análise: 5 frameworks de UI empilhados, 10.462 `!important`, 1.267 buckets de cor, z-index 2.147.483.647, 318 telas reprovadas no gate de a11y. O novo sistema nasce com **stack única**, **tokens em 3 camadas** e **a11y como gate de release**.

---

## 1. Biblioteca-base

### UI-01 — shadcn/ui sobre stack única
A interface web é construída sobre os componentes do **shadcn/ui** (https://ui.shadcn.com/) como biblioteca-base — componentes acessíveis (padrões WAI-ARIA APG), headless e estilizados por tokens, copiados para o projeto (sem dependência de versão opaca). É a **única** stack de UI do produto (RQ-CSS-06): proibido empilhar frameworks concorrentes. Implicação técnica: ecossistema React (alimenta o doc 07).

### UI-02 — Orçamentos de saúde de CSS (gates, RQ-CSS)
| Métrica | Orçamento | Como |
|---|---|---|
| `!important` | ≤ 20 (só utilities geradas) | lint stylelint bloqueante; especificidade resolvida por camadas/escopo |
| Buckets de cor | ~15 (rampas semânticas + neutros + marca) | toda cor referencia token; zero hex literal em componente |
| Breakpoints | ≤ 6, tokenizados | proibido valor literal em media query |
| z-index | escala de ~6 camadas, máx ≤ 999 | overlays via portal + gerenciador de stacking |
| Especificidade máx | ≤ (0,3,0) | seletores planos (CSS Modules/data-attrs) |

---

## 2. Shell de 5 áreas

### UI-03 — Estrutura persistente
Shell com NavSidebar + AppHeader persistentes, painéis laterais colapsáveis e conteúdo central fluido. Base de cálculo: viewport 1600px (percentuais proporcionais em qualquer resolução).

```
┌──────┬────────────────────────────────────┐
│      │            2 — AppHeader            │
│      ├────────────────────────────────────┤
│  1   │      3 — ContextBar (condicional)   │
│ Nav  ├──────────────────────┬─────────────┤
│Side  │   5 — MainContent    │  4 — Painel  │
│ bar  │   (1–3 colunas)      │   lateral    │
└──────┴──────────────────────┴─────────────┘
```

### UI-04 — Área 1: NavSidebar
Menu estrutural full-height (100vh), colapsável: expandido 290px (ícones + labels + sub-itens), colapsado 80px (só ícones). Botão de colapsar na borda direita. Navegação **por persona** (secretaria/direção, professor, responsável, aluno, admin do SaaS) com categorias, sub-itens e indicadores de status. Ao colapsar, MainContent e painel direito absorvem o espaço.

### UI-05 — Área 2: AppHeader
Topo fixo, 64px, à direita da NavSidebar. Conteúdo (esq.→dir.): botão de colapsar menu · **seletor de perfil/visão** (alterna entre perfis acumulados do usuário — preserva a troca de perfil da base) · breadcrumb da área · busca global · alternador claro/escuro · notificações · perfil/conta.

### UI-06 — Área 3: ContextBar
Condicional (~50px), abaixo do AppHeader, presente só nas telas que precisam: filtros de contexto, seletor de período/ano letivo, tabs locais, ações da página. Quando presente, empurra MainContent e painel direito para baixo.

### UI-07 — Área 4: Painel lateral direito
Colapsável (290px↔80px), começa abaixo das barras superiores (nunca toca o topo). **Na v1 é área reservada** para futuros painéis contextuais; na v2 hospeda o assistente da SIGE Intelligence (doc 08) — sempre presente, expansível/recolhível, com nome/avatar personalizáveis por escola; recolhido, exibe indicador discreto.

### UI-08 — Área 5: MainContent
Espaço de trabalho fluido, subdividido em 1–3 colunas auto-ajustáveis conforme espaço e conteúdo. Larguras se redistribuem automaticamente ao colapsar as laterais (tudo expandido → ~64%; ambos colapsados → ~90%). Hospeda os padrões de página (UI-10).

---

## 3. Arquitetura de tokens (RQ-TOK)

### UI-09 — Três camadas + white-label + dark mode
Hierarquia obrigatória **primitive → semantic → component**:
1. **Primitivos**: rampas de cor (neutros + estados), escala tipográfica, espaçamento (grade base 4px), raios, sombras, motion.
2. **Semânticos**: papéis (`bg`, `surface`, `border`, `text`, `text-muted`, `link`, `accent`; estados `success`/`danger`/`warning`/`info`) — e **tokens semânticos de domínio escolar** (RQ-TOK-04): `status.matricula.*`, `status.financeiro.inadimplente`, `status.academico.recuperacao`, `status.foto.pendente` etc. Badges/alertas consomem o token de domínio, nunca a cor crua.
3. **Componente**: tokens por componente shadcn/ui.

Requisitos:
- **White-label** (PLT-09): a cor de marca da escola é injetada na camada primitiva; as rampas semânticas derivam dela. Logomarca e favicon por tenant.
- **Dark mode completo** desde o início (RQ-TOK-02), alternado pelo AppHeader (UI-05).
- **Nomenclatura honesta** (RQ-TOK-03): nada de nome que mente sobre o valor; `info` e `action` não duplicados.
- Validação de **contraste** (WCAG 1.4.3) na configuração da paleta da escola — uma marca que falha o contraste é rejeitada.

---

## 4. Componentes compostos (catálogo mínimo)

### UI-10 — Estruturais e de dados
- **App Shell** (UI-03), **NavSidebar hierárquica**, **Breadcrumbs**, **Tabs** (com variante "modo de exibição" tabela/cards e abas roláveis — usadas no hub do aluno e nos detalhes).
- **Data table** com busca, filtros (incl. por status na URL), paginação server-side, linha clicável, **ações em lote** — o componente mais crítico do produto.
- **Planilha editável (grid)** com edição inline e navegação por teclado — planilha de notas/faltas, frequência em lote.
- **Tree view com checkboxes** — plano de contas, grade curricular, perfis de permissão.
- **KPI cards** com variação e link para a lista-alvo (PLT-16).

### UI-11 — Entrada e fluxo
- **Formulários compostos**: input com seções, label/description/error, **máscaras** (CPF/CNPJ, moeda em centavos, data DD/MM/AAAA), selects com busca, date picker, switch/checkbox/radio, **color picker** (tema da escola).
- **Wizard com Stepper** — matrícula online, procedimentos de matrícula, cadastro de turma, e todo o **framework de lote** (filtrar → editar → simular → processar, PLT-11).
- **Editor rich-text** com merge-fields — comunicados, circulares, contratos, termos, templates de e-mail. Editor moderno (substitui CKEditor 4 EOL da referência).
- **Calendário escolar** — visão mensal com eventos, feriados e horários.

### UI-12 — Feedback e overlay
- **Modal / Drawer / Dialog** tokenizados (sem modal via iframe).
- **Notification/Toast + Alert** por estado semântico; centro de notificações.
- **Badge/Pill/Chip de status** mapeados aos tokens semânticos de domínio (UI-09) — situação de matrícula, situação financeira, status de boleto/foto.
- **Progress + Loader + Indicator** — fila de processamento, uploads.
- **Avatar + Timeline** — foto do aluno/colaborador; timeline para histórico e auditoria.

### UI-13 — Hub 360° do aluno (padrão-chave)
Detalhe com abas persistentes (cadastro, ficha médica, turmas, ocorrências, histórico, descontos, cobranças, diário) — o padrão de navegação mais valioso da base/referência, elevado a requisito. Mesmo padrão de hub para colaborador e responsável.

---

## 5. Linguagem visual e estados

### UI-14 — Semântica de estados
- Sucesso/recebido → verde; erro/inadimplência → vermelho; pendência/atenção → âmbar (filas, aprovações pendentes); informação/ação → azul.
- Convenção de contraste: fundo claro do estado + texto escuro do estado.
- **Estados de página explícitos e distintos** (RQ-UI): vazio, carregando, erro — nunca um empty state renderizado como alerta de aviso (anti-padrão da referência).

---

## 6. Acessibilidade — gate de release (RQ-A11Y)

| ID | Requisito |
|---|---|
| RQ-A11Y-01 | Todo campo de formulário tem `<label>` associado (ou `aria-label`/`aria-labelledby`); o componente Input não renderiza sem rótulo acessível. |
| RQ-A11Y-02 | Componentes compostos (Tabs, Menu, Combobox, Popover) seguem WAI-ARIA APG completo; componentes compartilhados testados com rigor proporcional ao alcance. |
| RQ-A11Y-03 | Todo controle icon-only exige `aria-label`; ícones decorativos `aria-hidden` com rótulo textual paralelo; progressbars/toggles com nome acessível. |
| RQ-A11Y-04 | Markup semanticamente válido (sem wrappers de layout dentro de listas). |
| RQ-A11Y-05 | `<html lang="pt-BR">`; evitar iframes (se inevitável p/ BI futuro, `title` obrigatório). |
| RQ-A11Y-06 | CI com axe-core em **browser real**; contraste 1.4.3 auditado; **meta: zero violações critical/serious por tela como gate de release**. |

---

## 7. Invariantes do design system

1. Stack de UI única (shadcn/ui); proibido empilhar frameworks concorrentes.
2. Toda cor referencia um token; zero hex literal em código de componente.
3. Badges/alertas de status consomem token semântico de domínio, nunca cor crua.
4. White-label: cor de marca por tenant injetada na camada primitiva, com contraste validado.
5. Dark mode completo desde a v1.
6. Gate de release de a11y: zero violações critical/serious por tela.
7. Estados vazio/carregando/erro são distintos e explícitos em toda tela.
8. Nenhum `!important` além do orçamento; nenhum z-index "nuclear".
