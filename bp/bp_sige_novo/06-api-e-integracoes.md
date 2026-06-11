# 06 — API e Integrações

> Documento 06 de 10. Define o contrato unificado da API (que substitui as duas gerações legadas consumidas pelo App iOS/Android), a estratégia de migração do app, a API pública de parceiros e o catálogo de conectores externos. IDs: `API-xx` (contrato/app), `PUB-xx` (API pública), `INT-xx` (conectores).
>
> Fonte do estado atual: `inventario_api_atual.md` (135 endpoints reais — 36 em `/api/v2`, 99 em `/api` legado, 3 especiais). **Nenhum endpoint atual é replicado literalmente**; todos mapeiam para o contrato novo (rastreabilidade em §3 e doc 09).

---

## 1. Princípios do contrato novo (G3 — virada coordenada)

### API-01 — Uma API unificada
Substitui as **duas gerações** (`/api` legada + `/api/v2`) por **um único contrato versionado** (`/api/v1` do novo sistema), eliminando a duplicação (login, recuperação de senha, permissões em 3 implementações, alunos, circulares, boletos, fotos, chamada — todos duplicados hoje). O App iOS/Android é **adaptado à camada de rede** e lançado junto com o backend novo (**virada coordenada**, sem camada de compatibilidade). As telas/lógica do app não mudam; muda o cliente HTTP.

### API-02 — Correções estruturais obrigatórias (dívidas do legado)
O contrato novo corrige, por requisito:
| Dívida do legado (inventário §5) | Correção |
|---|---|
| Middleware executa a ação **antes** de validar o token | Validação de token **antes** de qualquer efeito (RQ-SEG-01) |
| "Token" sem assinatura nem expiração verificadas; senha-mestre e chave de SMS no código | Token assinado, com expiração/renovação/revogação; segredos fora do código (RQ-SEG-01/02) |
| PDFs (boleto/circular) e fotos públicos por ID sequencial | Recursos binários sempre autorizados por sessão/escopo (RQ-SEG-04) |
| POST para leitura; `id` da pessoa no body sem cruzar com o token | REST coerente (GET lê); autorização cruzada com o sujeito do token |
| Envelopes inconsistentes (`{status,mensagem}`, `{sucesso}`, arrays crus, text/plain; `status` sobrescrito por inteiro) | **Envelope único** `{ sucesso, mensagem, dados, erros[] }` + códigos HTTP corretos |
| Ano letivo e IDs hardcoded; endpoint de feriados retornando lista vazia fixa | Tudo derivado de parametrização (RQ-PAR-01); sem stubs inócuos |
| Lista hardcoded de IDs autorizados a fotos; bypass de renovação para pessoa específica | Autorização por papel/atribuição/escopo, sem listas no código |

### API-03 — Convenções
- **Multi-tenant**: tenant por subdomínio/claim, nunca em path (RQ-SEG-04).
- **Autenticação**: token assinado (sessão para portais web; OAuth2 para parceiros — PUB-02).
- **Envelope** padrão único; paginação consistente; filtros como query params; erros com código + mensagem.
- **Versionamento** explícito do contrato; sem snapshots por migração de banco (anti-padrão da referência).
- **Idempotência** em operações de escrita sensíveis (ex.: geração de cobrança, emissão fiscal — chave de referência).

---

## 2. Domínios do contrato do app (substituem os ~105 endpoints)

Organização por domínio (consolidando os grupos do inventário):

| ID | Domínio | Substitui (legado/v2) | Capacidades |
|---|---|---|---|
| API-10 | **Autenticação e conta** | token, autenticar, auth social, validar-identidade, recuperação e-mail/SMS, verificar-código, redefinir, permissões (3 impls), troca de perfil, verificar-versão | Login tradicional + social; token assinado; recuperação e-mail/SMS; **um** modelo de permissões; troca de perfil; versão mínima |
| API-11 | **Notificações** | notificações-usuario, marcar, contar, ler | Listagem, contagem, leitura/remoção |
| API-12 | **Alunos e cadastro** | alunos-dados, dados-pessoais, dados-atendimento, parentes, listar-por-parente/colaborador/turma, inserir-avatar | Lista contextualizada por atribuição; dados do aluno; avatar |
| API-13 | **Rotina diária** | inserir/excluir refeições, repouso, chamada, saída, medicação (ministrar/remover), troca-fralda, diário-completo, refeições/repousos/evacuação por aluno, listar medicações | Toda a rotina infantil (ROT-01..03); escrita autorizada pelo token |
| API-14 | **Comunicados do diário** | comunicados, inserir/excluir-comunicado, destinatários, info, trocar-status, comunicado-geral | Comunicados do diário com aprovação |
| API-15 | **Circulares** | circulares por-aluno, última, pdf, data-leitura, dados-circular | Lista, leitura, confirmação; PDF **autorizado** |
| API-16 | **Fotos** | fotos por-aluno/usuário/pendentes/lixeira, inserir, alterar-status, log, carregar-mais, foto-jpg | Galeria conforme atribuição; dupla aprovação; binário autorizado |
| API-17 | **Financeiro do aluno** | boletos, pdf-boleto, linha-digitável, parcelas-crv | Títulos do aluno; PDF/linha **autorizados**; parcelas de cota |
| API-18 | **Renovação de matrícula** | renovar, não-renovar, verificar-renovação | Janela sazonal; restrita a responsável legal/atribuição |
| API-19 | **Turmas e frequência** | turmas, turmas-frequência, alunos por turma/refeição/sono/chamada/evacuação, registrar presença/ausência/saída | Operação de sala; ambos os modos de frequência (C5) |
| API-20 | **Calendário** | feriados, dias-letivos, próximas-aulas | Derivado do calendário real (sem stub vazio) |
| API-21 | **Parentes/perfil** | destinatários, configurações, dados-pessoais, avatar | Perfil e preferências do responsável |
| API-22 | **Colaboradores/perfil** | configurações, dados-pessoais | Perfil do colaborador |
| API-23 | **Infraestrutura** | verificar-versão, deep links (assetlinks/apple-app-site-association), webhook de assinatura | Versão mínima; deep links; webhooks de conectores |

> Cada endpoint atual tem destino nesta tabela; o mapa linha-a-linha vai no doc 09. Código morto do legado (10 controllers) **não tem destino** — descartado.

---

## 3. API pública de parceiros (G — "tomada na parede")

### PUB-01 — Escopo na v1
A API pública **nasce na v1 sem nenhum parceiro integrado** — é a fundação para conectar, no futuro e sem mexer no núcleo: catracas/portarias, garantidoras de recebíveis, editoras/plataformas de conteúdo, plataformas de matrícula, contabilidade.

### PUB-02 — Autenticação e segurança
**OAuth2 client-credentials com escopos** (ex.: `cadastros:read`, `financeiro:write`, `dados_sensiveis:read`, `notas:write`, `frequencia:write`) — corrige o anti-padrão do Bearer único por instituição. Auditoria por token; **endpoints de dados sensíveis segregados** com escopo próprio (RQ-SEG-05). Tenant resolvido fora do path.

### PUB-03 — Superfície mínima (quando ativada)
Sync de cadastros (alunos/responsáveis/turmas/colaboradores), leitura de boletim/frequência, escrita de notas e frequência em lote, geração/consulta de cobrança e boleto, e SSO de portal. Documentação OpenAPI **sem PII real** (RQ-SEG-04).

---

## 4. Catálogo de conectores externos (v1)

| ID | Conector | Papel | Estratégia |
|---|---|---|---|
| INT-01 | **Assinatura eletrônica** | Contratos/requerimentos (MAT-05) | **Conector abstrato** (criar documento, signatários, status, webhook). **Autentique E Clicksign homologados na v1** (C8); default por tenant; outros como adaptadores |
| INT-02 | **Fiscal (NFS-e)** | Emissão de NFS-e (FIN-20) | **Conector abstrato**; **Focus NFe** adaptador default; NFE.io alternativa; PlugNotas/Nuvem Fiscal plano B. Substitui a integração direta com a prefeitura |
| INT-03 | **Cobrança bancária** | Boleto registrado (FIN-08/09) | **Adaptadores plugáveis**; CNAB Santander e Sicoob na v1 (seeds); registro online por API bancária fora da v1 |
| INT-04 | **PIX** | Recebimento instantâneo (FIN-09) | QR dinâmico + boleto híbrido + conciliação; adaptador por PSP |
| INT-05 | **Push** | Notificações do app próprio (COM-06) | Provedor de push; endereçamento por identificador de login |
| INT-06 | **E-mail (SMTP)** | Transacionais e régua (COM-07) | Conta/remetente por unidade; templates com variáveis |
| INT-07 | **SMS** | Código de recuperação e régua (PES-06/FIN-06) | Provedor de SMS (gerido na plataforma) |
| INT-08 | **Lookup de CEP** | Autocomplete de endereço (PES-01) | ViaCEP/BrasilAPI |
| INT-09 | **Login social** | Google/Apple/Facebook (PES-06) | Federação convergindo ao token interno; deep links de retorno |

### INT-10 — Princípios de conector
Todo conector nasce com: interface abstrata (provider-agnostic), **log de integração auditável** e tela de pendências, configuração de credenciais por tenant (segredos seguros), e tratamento de webhook idempotente. Trocar/adicionar provedor não toca o núcleo.

---

## 5. Invariantes de API e integrações

1. Token é validado **antes** de qualquer efeito colateral; token inválido não grava nada (RQ-SEG-01).
2. Recursos binários (PDF, foto) só são servidos com autorização por sessão/escopo — nunca por ID público (RQ-SEG-04).
3. Toda resposta usa o envelope único; nenhum campo de status é reaproveitado para outro significado.
4. Escrita autoriza cruzando o sujeito do token com o recurso — nunca confia em `id` vindo do corpo.
5. Nenhuma data/ano/lista de IDs no código (RQ-PAR-01).
6. API pública: OAuth2 com escopos; dados sensíveis em escopo próprio com auditoria por token.
7. Cada integração externa passa por conector abstrato com log auditável; provedor é trocável sem tocar o núcleo.
8. Documentação pública de API não contém PII real.
