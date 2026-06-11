# 07 — Diretrizes Tecnológicas

> Documento 07 de 10. **Capítulo de recomendações não-vinculantes** (decisão G4). O blueprint é funcional e independente de tecnologia; este documento reúne os constrangimentos que o desenho funcional impõe ao SSD e sugere uma arquitetura de referência. **A decisão vinculante de stack pertence ao SSD** — etapa seguinte. IDs: `TEC-xx`.

---

## 1. Natureza deste documento

As decisões funcionais tomadas no blueprint **constrangem** a arquitetura, mas não a determinam. Este capítulo:
1. Lista os **constrangimentos duros** (não-negociáveis, derivados de requisitos funcionais).
2. Apresenta uma **arquitetura de referência** (sugestão, para acelerar o SSD).
3. Registra o **stack legado** como ponto de partida da migração.

---

## 2. Constrangimentos arquiteturais duros (não-negociáveis)

Derivam de decisões já fechadas — qualquer stack escolhido no SSD deve satisfazê-los:

| ID | Constrangimento | Origem |
|---|---|---|
| TEC-01 | **Multi-tenancy com isolamento por tenant** em todas as camadas; tenant resolvido por subdomínio/claim, nunca por path | PIL-02, RQ-SEG-04 |
| TEC-02 | **Filas e processamento assíncrono** com estado visível, retry e log por item (operações em lote, rotinas, integrações) | PLT-11/12 |
| TEC-03 | **Barramento de eventos de domínio** interno (publicação/assinatura) — base das automações da v1 e dos agentes da v2 | PLT-13, doc 08 |
| TEC-04 | **Autenticação por token assinado** com expiração/renovação/revogação; validação antes de qualquer efeito; **OAuth2 + escopos** para a API pública | RQ-SEG-01, PUB-02 |
| TEC-05 | **Gestão segura de segredos** (fora do código/versionamento); nenhum segredo hardcoded | RQ-SEG-02 |
| TEC-06 | **Frontend React** (decorrência da escolha do shadcn/ui como biblioteca-base) com tokens em 3 camadas e a11y como gate de CI | UI-01, RQ-A11Y-06 |
| TEC-07 | **App iOS/Android em Flutter/Dart** mantido, adaptado ao contrato de API novo (virada coordenada) | API-01 |
| TEC-08 | **Conectores externos por interface abstrata** (assinatura, fiscal, bancário, push, e-mail, SMS) com log auditável e provedor trocável | INT-10 |
| TEC-09 | **Auditoria e soft delete** transversais; autoria não-humana (agente) prevista no modelo de log | PIL-07/10 |
| TEC-10 | **Invariantes garantidas na escrita** (transações/validações), eliminando rotinas de correção de dados | PIL-09 |
| TEC-11 | **Valores monetários em inteiros (centavos)** em toda a stack | PIL-13 |
| TEC-12 | **Parametrização em camadas** (global→tenant→unidade→série×período→turma→diário) com resolução da regra mais específica | PLT-18 |
| TEC-13 | **Idempotência** em escritas sensíveis (cobrança, emissão fiscal, webhooks) | API-03, INT-10 |

---

## 3. Arquitetura de referência (sugestão para o SSD)

> Recomendação, não imposição. Serve de ponto de partida; o SSD pode divergir justificadamente.

### TEC-20 — Camadas
- **Frontend web**: React + shadcn/ui (UI-01), tokens DTCG em 3 camadas, build com tree-shaking de estilos; SPA ou app server-rendered conforme o SSD avaliar para SEO das páginas públicas.
- **App mobile**: Flutter/Dart (existente), camada de rede reescrita para o contrato novo.
- **Backend**: API de aplicação (REST + envelope único) + workers de fila + scheduler de rotinas + publisher/consumer de eventos. Modular por domínio (plataforma, acadêmico, financeiro, comunicação).
- **Persistência**: banco relacional (modelo normalizado — corrige as listas JSON desnormalizadas do legado, §12.4 da base), com isolamento por tenant.
- **Infra**: containerizada (o legado já roda em Docker), com gestão de segredos, observabilidade (logs/erros/telemetria com base legal LGPD) e CI com gate de a11y (axe-core em browser real).

### TEC-21 — Decisões deixadas explicitamente ao SSD
- Linguagem/framework do backend (o legado é PHP/CakePHP 4; o SSD avalia continuidade vs. migração).
- SGBD específico (o legado é MySQL 8).
- Estratégia de isolamento multi-tenant (schema por tenant × discriminador por linha × banco por tenant) — decisão de trade-off custo/isolamento.
- Tecnologia de fila e de barramento de eventos.
- Estilo de arquitetura (monólito modular × serviços) — recomendação inicial: **monólito modular** com fronteiras de domínio claras, dada a equipe e o estágio do produto.
- Hospedagem e estratégia de subdomínios/wildcard TLS para white-label.

---

## 4. Stack legado (ponto de partida da migração)

Confirmado no código (`C:\Users\Fabio\Documents\sige`):

| Camada | Tecnologia atual |
|---|---|
| Backend | PHP ≥ 7.2, **CakePHP 4.0.4** |
| Banco | **MySQL 8.0** |
| PDF | mPDF 8 |
| Assinatura | Autentique (lib `autentique-v2`) |
| App | iOS/Android em **Flutter/Dart** |
| Infra | Docker (app + MySQL + phpMyAdmin) |
| Auth atual | Token sem assinatura/expiração verificadas (a corrigir — TEC-04) |

**Dívidas técnicas do legado a não repetir** (consolidadas da análise e do inventário): duas gerações de API, token frágil, segredos no código, relações em JSON desnormalizado, rotinas de correção de dados, datas/IDs hardcoded, código morto (10 controllers), PDFs/fotos públicos por ID, frontend com múltiplos frameworks empilhados e dívidas de a11y/CSS.

---

## 5. Recomendação de sequência técnica (informativa)

Sem constituir roadmap (que é etapa posterior), a ordem natural de fundação sugerida ao SSD:
1. Núcleo multi-tenant + auth + auditoria + parametrização em camadas.
2. Modelo de dados normalizado dos domínios acadêmico e financeiro.
3. Framework de lote + filas + barramento de eventos.
4. Contrato de API novo + adaptação do app (virada coordenada).
5. Conectores (assinatura, fiscal, bancário/PIX, push, e-mail/SMS).
6. Frontend (shell + design system + hub do aluno) e portais.
