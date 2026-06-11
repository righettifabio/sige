# 04 — Comunicação e Portais

> Documento 04 de 10. Canais de comunicação escola↔família, caixa de saída, fotos, notificações/push, permissões dos portais em camadas, portais por persona, App iOS/Android e LGPD/termos. IDs: `COM-xx` (comunicação), `POR-xx` (portais), `LGP-xx` (privacidade).
>
> Regra-mestra: os canais da base (comunicados, circulares, ocorrências, fotos com dupla aprovação) são MANTER; o repertório agrega caixa de saída, templates e termos LGPD; o conflito C4 (permissões em camadas) é aplicado. O lado família dos portais é a **fonte confiável da base** — o repertório é inferido nessa área.

---

## 1. Canais de comunicação

### COM-01 — Comunicados — MANTER + APRIMORAR
Recado/anotação entre escola e família (base §3.5): mensagem rich-text, autor (colaborador ou parente), menções (@pessoa, #tag), anexos, vínculo a aluno ou observação geral, **destinatários por turma/aluno**, controle de leitura por destinatário, status (ativo / pendente de aprovação / excluído — lixeira), **fluxo de aprovação**. Acréscimos: **audiência estruturada** (multi-select de turmas × filtro por situação do aluno × toggle de visibilidade — depende de MAT-10) e **agendamento por data de publicação**. E-mail automático aos destinatários; tudo logado na caixa de saída (COM-05).

### COM-02 — Circulares — MANTER literal
Comunicado formal **individualizado por aluno/destinatário**, derivado de um lote (título, mensagem, anexo, turmas destinatárias, data de envio), com rastreio de entrega e **confirmação de leitura** (lido / lido em) e **reenvio aos não confirmados** (base §4.11). E-mail automático. Diferencial da base sem equivalente no repertório — preservado integral.

### COM-03 — Ocorrências — MANTER + APRIMORAR
Registro de incidente/observação (base §3.5): autor, texto com menções e tags, **comentários encadeados (respostas)**, controle de visualização. Acréscimos: **tipos** (pedagógica, disciplinar, financeira, psicológica), **data de liberação** ao responsável (controle de quando fica visível), **visibilidade por categoria** no portal, flag **gera impedimento** (doc 03). Os comentários encadeados (só a base tem) permanecem.

### COM-04 — Fotos com dupla aprovação — MANTER literal (DF-3)
Foto de evento/rotina com descrição, alunos e colaboradores envolvidos, **fluxo de dupla aprovação** (duas aprovações independentes ativam a publicação), status ativo/pendente/excluído, compartilhamento seletivo com parentes (conforme atribuição), registro de quem visualizou/compartilhou. Soma-se ao **termo de uso de imagem** (LGP-02): a publicação verifica o consentimento vigente.

### COM-05 — Caixa de saída (trilha universal) (DF-4) — ACRESCENTAR
Log central de **toda** comunicação enviada às famílias, sob os canais existentes: cada envio registra inserção, tipo (e-mail/SMS/push), destinatário e **tipo de destinatário**, canal, agendamento, data de envio e **log de entrega**. Reenvio em lote. É a "prova de entrega" da escola e o ponto único de auditoria de comunicação (PIL-07). Alimentada por eventos de domínio (PLT-13) e pela régua de cobrança (FIN-06).

### COM-06 — Notificações internas e push — MANTER
Centro de notificações internas ("sino") por usuário: tipo, metadados de vínculo (ex.: aluno + boleto), lida/removida (base §3.5). **Push** pelo app próprio (provedor de push), endereçado por identificador do login. **Campanhas push** do administrador: seleção de alunos/turmas, título e texto, **filtro por atribuição do parente**, envio em lote, registro individual e espelhamento como notificação interna. Toda push relevante espelha em notificação interna.

### COM-07 — Templates de e-mail — APRIMORAR
Templates HTML por escola com **variáveis** (nome do aluno, matrícula, parentesco etc.) e cabeçalho/rodapé com a marca (white-label — PLT-09). Os e-mails transacionais da base (cobrança, circular, comunicado, ocorrência, criação de usuário, recuperação de senha, prospect inscrito/duplicado/convertido) passam a referenciar **templates seed** idênticos aos textos atuais, editáveis pela escola. Remetente SMTP configurável por unidade.

### COM-08 — Documentos institucionais e porta-arquivos — MANTER + ACRESCENTAR
Biblioteca de documentos institucionais em capítulos ordenáveis com visualização autenticada (MANTER). **Porta-arquivos** (ACRESCENTAR): upload de arquivos pela família/professores, também usado pelos **procedimentos de matrícula** (MAT-03) para envio de documentos.

### COM-09 — Avaliação institucional (canal família→escola) — ACRESCENTAR
Pesquisa de satisfação publicada pela escola, respondida pela família no portal/app, com relatório. Canal de retenção de baixo custo.

> **Provedores de e-mail/SMS/push** são geridos no nível da plataforma (configuração de remetente/conta por escola onde aplicável); a decisão de o que é configurável por tenant é explícita (doc 06 §conectores).

---

## 2. Portais e permissões

### POR-01 — Permissões em camadas (C4, PIL-04)
O que cada pessoa vê nos portais resulta da **composição de três camadas**, e **o mais restritivo prevalece**:
1. **Atribuição** (parente×aluno — PES-02): "quem vê o quê de qual aluno" (fonte primária, da base).
2. **Situação do aluno** (MAT-10): "quando" (ex.: situação inativa-finalizada desliga recursos).
3. **Configuração da escola**: "quais recursos existem em cada portal" (toggles por persona × recurso).

**Exceção fixa (PES-03):** o responsável legal sempre acessa os dados do próprio filho — nenhuma camada desliga isso.

### POR-02 — Portal/App do responsável — MANTER (fonte confiável: base §5.1)
Capacidades: lista de alunos vinculados contextualizada por atribuição; **diário do filho** (presença/frequência, refeições, sono, medicações, trocas de fralda, comunicados do dia; inserção de anotações); **financeiro** (títulos do aluno, detalhe, linha digitável e PDF do boleto, parcelas de cota); **circulares** (lista, última, leitura do arquivo, confirmação); **fotos** (galeria por aluno conforme permissão, registro de visualização/compartilhamento); **renovação de matrícula** na janela sazonal (restrita ao responsável legal ou atribuição específica); dados pessoais (consulta/edição de cadastro, endereço, telefones, avatar); **ficha médica** (visualizar/editar conforme toggle — ROT-07); notificações; calendário (feriados, dias letivos, próximas aulas). Acréscimos: **agendamento de reunião self-service** (sobre disponibilidades da escola), **declaração de IR**, **carteirinha digital**, **contato com a instituição por setor**.

### POR-03 — Portal web do responsável — ACRESCENTAR
Paridade web com o app (mesma API), reduzindo exclusão de famílias sem smartphone adequado. Mesmas permissões em camadas (POR-01).

### POR-04 — Portal/App do colaborador (operação de sala) — MANTER (base §5.2)
Turmas do colaborador; alunos por turma e por contexto (chamada, refeição, sono, evacuação, medicação); **registro da rotina** (presença/ausência, entrada/saída, refeições com escala, repouso, medicações por horário, trocas de fralda, saída com pessoa autorizada); comunicados (criar/editar/excluir, menções, destinatários); fotos (upload, fila de aprovação, lixeira); dados próprios.

### POR-05 — Portal do professor (segmentos seriados) — ACRESCENTAR
Acesso ativado pela alocação didática (PES-05): diário de classe, chamada online, lançamento e confirmação de notas, avaliação por relatório/competência — regido pelos parâmetros do motor de avaliação (AVA-07). Visível só conforme situação ativa.

### POR-06 — Portal do aluno (segmentos com idade para tal) — ACRESCENTAR
Boletim/notas, atividades avaliativas, quadro de horários, carteirinha digital, comunicados, porta-arquivos. Sem sentido no infantil; habilitado por segmento.

### POR-07 — Matriz de configuração dos portais (escola)
Por persona × situação × recurso: critérios de acesso ao produto (situações que dão acesso × períodos liberados), toggles de funcionalidade (financeiro, boletim, frequência, comunicados, ocorrências por categoria, ficha médica editável, lista de turma, agendamento, etc.), permissões de edição cadastral pelo responsável (por campo), descontos ocultáveis no portal. Comportamento da base é o conjunto seed dessa matriz.

---

## 3. Aplicativo iOS/Android

### POR-08 — App próprio (MANTER) + tema dinâmico (G8)
App único iOS/Android (desenvolvido em Flutter/Dart) para responsáveis e colaboradores, sobre o contrato de API novo (doc 06). Tema dinâmico por tenant (PLT-10). Mantidos da base: **verificação de versão mínima** por plataforma (bloqueio de versões antigas), **deep links** Android/iOS (arquivos de associação de domínio), login social, troca de perfil. ClassApp descartado (C9): o requisito de log de entrega é atendido pela caixa de saída (COM-05).

---

## 4. LGPD, termos e privacidade

### LGP-01 — Termo de consentimento — ACRESCENTAR (prioridade legal)
Texto por escola (editável), com flag ativo e **aceite datado por titular**, exibido nos fluxos: matrícula online/ficha de inscrição, reserva de vaga, captação, inscrição em eventos, solicitação de alteração cadastral, solicitação de desconto. Registra finalidade e canal do titular (DPO). Seeds: termo modelo a editar (PLT-04).

### LGP-02 — Termo de uso de imagem — ACRESCENTAR
Texto rico por escola, com aceite datado, exibido na ficha de inscrição, reserva e alteração cadastral. **Compõe com a dupla aprovação de fotos** (COM-04): a publicação de foto verifica o consentimento de imagem vigente do aluno.

### LGP-03 — Dados sensíveis e minimização
Endpoints de dados sensíveis segregados com escopo próprio e auditoria (RQ-SEG-05, doc 06); CPF mascarado em telas de consulta; solicitação de alteração cadastral pelo responsável com granularidade por campo e consentimento. Exportação publicitária de contatos (MANTER) **condicionada ao consentimento** (RQ-SEG-07).

---

## 5. Comunicações geradas pelo sistema (preservadas da base)

- **E-mails transacionais** (via templates COM-07): boletos vencidos (painel de cobrança), circular publicada, comunicado/ocorrência, criação de usuário/redefinição (agora convite), recuperação de senha, prospect inscrito/duplicado/convertido.
- **Push e notificações internas** (COM-06): novo título/vencendo/vencido (reaviso 3 dias — seed da régua), comunicado publicado, foto aprovada, circular publicada, campanhas do administrador.
- **Documentos PDF**: contrato de serviços, boleto bancário (com a marca da escola — PLT-09), circulares e documentos anexados, e (segmentos seriados) boletim, histórico, declarações.

---

## 6. Invariantes do domínio de comunicação e portais

1. Toda comunicação enviada às famílias é registrada na caixa de saída (COM-05).
2. Foto só fica visível após duas aprovações independentes E consentimento de imagem vigente (COM-04 + LGP-02).
3. O que o responsável vê resulta da composição atribuição × situação × configuração; o mais restritivo vence — exceto o acesso pleno do responsável legal aos dados do filho (POR-01/PES-03).
4. Circular não confirmada pode ser reenviada; a confirmação de leitura é rastreada por destinatário (COM-02).
5. Push relevante é sempre espelhada como notificação interna (COM-06).
6. Termos de consentimento e de uso de imagem têm aceite datado por titular (LGP-01/02).
7. Exportação de contatos para mídia exige consentimento vigente (LGP-03).
8. O app verifica versão mínima por plataforma antes de operar (POR-08).
