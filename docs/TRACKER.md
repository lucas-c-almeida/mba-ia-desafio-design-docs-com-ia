# Tracker de Rastreabilidade

Cada linha liga um item dos documentos à sua origem: uma fala da transcrição (`TRANSCRICAO`, no formato `[hh:mm] Nome`) ou um arquivo real do repositório (`CODIGO`).

Como ler:
- O ID é o mesmo que aparece escrito no documento de origem.
- Quando um item tem mais de uma origem, a coluna Localização traz a principal. As demais estão citadas no próprio documento.
- Itens marcados como `[HIPÓTESE]` nos documentos aparecem aqui ligados à fala ou ao arquivo de que foram derivados, e estão listados na seção 3.1 do [FDD](./FDD.md) para revisão.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Outbox em tabela do MySQL, gravada na mesma transação da mudança de status | TRANSCRICAO | [09:08] Larissa |
| ADR-002 | docs/adrs/ADR-002-worker-separado-com-polling.md | Decisão | Worker em processo separado, com polling de 2 segundos | TRANSCRICAO | [09:10] Larissa |
| ADR-003 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | 5 tentativas com backoff 1m/5m/30m/2h/12h e DLQ em tabela separada | TRANSCRICAO | [09:17] Larissa |
| ADR-004 | docs/adrs/ADR-004-at-least-once-com-x-event-id.md | Decisão | Entrega at-least-once com X-Event-Id para deduplicação no cliente | TRANSCRICAO | [09:26] Larissa |
| ADR-005 | docs/adrs/ADR-005-hmac-sha256-secret-por-endpoint.md | Decisão | HMAC-SHA256 sobre o corpo, secret por endpoint, rotação com 24h de grace period | TRANSCRICAO | [09:22] Sofia |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Reuso de AppError, Pino, error middleware, padrão de módulos e schemas Zod | TRANSCRICAO | [09:30] Larissa |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Integração | Padrão de erros com código, base dos erros WEBHOOK_* | CODIGO | src/shared/errors/http-errors.ts |
| ADR-007 | docs/adrs/ADR-007-payload-snapshot-na-insercao.md | Decisão | Payload gravado já renderizado na inserção da outbox | TRANSCRICAO | [09:52] Larissa |
| RFC-ALT-01 | docs/RFC.md | Alternativa | Disparo síncrono no service de pedidos, descartado | TRANSCRICAO | [09:06] Diego |
| RFC-ALT-02 | docs/RFC.md | Alternativa | Redis Streams, descartado por exigir mais infraestrutura | TRANSCRICAO | [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa | Trigger do banco em vez de polling, descartada | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | docs/RFC.md | Alternativa | Exactly-once, descartado pela complexidade | TRANSCRICAO | [09:25] Diego |
| RFC-OQ-01 | docs/RFC.md | Questão em Aberto | Rate limiting de envio: observar e decidir depois | TRANSCRICAO | [09:39] Larissa |
| RFC-OQ-02 | docs/RFC.md | Questão em Aberto | Como escalar o worker sem perder a ordem por pedido | TRANSCRICAO | [09:13] Diego |
| RFC-OQ-03 | docs/RFC.md | Questão em Aberto | Endurecer as permissões do CRUD de configuração | TRANSCRICAO | [09:37] Sofia |
| RFC-OQ-04 | docs/RFC.md | Questão em Aberto | Arquivamento das linhas entregues da outbox | TRANSCRICAO | [09:08] Diego |
| RFC-OQ-05 | docs/RFC.md | Questão em Aberto | Se o envio inicial conta entre as 5 tentativas | TRANSCRICAO | [09:17] Diego |
| RFC-RISK-01 | docs/RFC.md | Risco | Falha na outbox derruba a mudança de status | TRANSCRICAO | [09:40] Bruno |
| RFC-RISK-02 | docs/RFC.md | Risco | Perda de ordem se o worker for escalado | TRANSCRICAO | [09:12] Diego |
| RFC-RISK-03 | docs/RFC.md | Risco | Vazamento de secret do lado do cliente | TRANSCRICAO | [09:22] Diego |
| RFC-RISK-04 | docs/RFC.md | Risco | Defeito em HMAC ou geração de secret, mitigado por revisão de segurança | TRANSCRICAO | [09:46] Sofia |
| RFC-RISK-05 | docs/RFC.md | Risco | Cliente fora do ar além da janela de retry, mitigado por DLQ e replay | TRANSCRICAO | [09:18] Diego |
| PRD-OBJ-01 | docs/PRD.md | Métrica | Notificação em menos de 10 segundos | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | docs/PRD.md | Métrica | Atender os 3 clientes B2B que fizeram o pedido formal | TRANSCRICAO | [09:00] Marcos |
| PRD-OBJ-03 | docs/PRD.md | Métrica | Entrega em 3 sprints, com revisão de segurança incluída | TRANSCRICAO | [09:47] Larissa |
| PRD-OBJ-04 | docs/PRD.md | Métrica | Janela de retentativas de quase 15 horas | TRANSCRICAO | [09:17] Diego |
| PRD-OOS-01 | docs/PRD.md | Fora de Escopo | Email quando o webhook falha, adiado para próxima fase | TRANSCRICAO | [09:37] Larissa |
| PRD-OOS-02 | docs/PRD.md | Fora de Escopo | Dashboard visual, projeto separado do frontend | TRANSCRICAO | [09:40] Larissa |
| PRD-OOS-03 | docs/PRD.md | Fora de Escopo | Rate limiting de envio, observar e decidir depois | TRANSCRICAO | [09:39] Larissa |
| PRD-OOS-04 | docs/PRD.md | Fora de Escopo | Webhooks de entrada, do cliente para o OMS | TRANSCRICAO | [09:02] Marcos |
| PRD-OOS-05 | docs/PRD.md | Fora de Escopo | Garantia de ordem global dos eventos | TRANSCRICAO | [09:13] Larissa |
| PRD-OOS-06 | docs/PRD.md | Fora de Escopo | Garantia exactly-once | TRANSCRICAO | [09:25] Diego |
| PRD-OOS-07 | docs/PRD.md | Fora de Escopo | Itens do pedido no payload | TRANSCRICAO | [09:43] Diego |
| PRD-OOS-08 | docs/PRD.md | Fora de Escopo | Arquivamento de eventos antigos | TRANSCRICAO | [09:08] Diego |
| PRD-OOS-09 | docs/PRD.md | Fora de Escopo | Permissões mais restritas no CRUD de configuração | TRANSCRICAO | [09:37] Sofia |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Notificar o cliente a cada mudança de status de pedido | TRANSCRICAO | [09:00] Marcos |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Cadastrar endpoint com URL e status, secret gerada pelo OMS | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Editar endpoint de webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Remover endpoint de webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Listar os webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Filtro de status por endpoint | TRANSCRICAO | [09:33] Marcos |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Rotação de secret com 24h de validade da antiga | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Histórico das últimas 100 entregas por endpoint | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Assinatura para o cliente validar origem e integridade | TRANSCRICAO | [09:19] Sofia |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Retentativa automática quando a entrega falha | TRANSCRICAO | [09:17] Larissa |
| PRD-FR-11 | docs/PRD.md | Requisito Funcional | DLQ para falhas permanentes e replay manual por ADMIN | TRANSCRICAO | [09:18] Diego |
| PRD-FR-12 | docs/PRD.md | Requisito Funcional | Identificador único por evento para deduplicação | TRANSCRICAO | [09:25] Diego |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | Latência abaixo de 10 segundos | TRANSCRICAO | [09:02] Marcos |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | Timeout de 10 segundos na chamada ao cliente | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Limite de 64KB no payload, com erro se ultrapassar | TRANSCRICAO | [09:24] Larissa |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | Cliente lento não trava nem desfaz a mudança de status | TRANSCRICAO | [09:04] Bruno |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | Processamento não para quando a API reinicia | TRANSCRICAO | [09:11] Diego |
| PRD-NFR-06 | docs/PRD.md | Restrição | Só URLs https são aceitas | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-07 | docs/PRD.md | Requisito Não Funcional | Secret única por endpoint, sem secret global | TRANSCRICAO | [09:21] Sofia |
| PRD-NFR-08 | docs/PRD.md | Requisito Não Funcional | CRUD para qualquer role autenticada, replay só para ADMIN | TRANSCRICAO | [09:36] Larissa |
| PRD-NFR-09 | docs/PRD.md | Restrição | Revisão de segurança antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-NFR-10 | docs/PRD.md | Requisito Não Funcional | Replay registra quem executou, para auditoria | TRANSCRICAO | [09:36] Sofia |
| PRD-NFR-11 | docs/PRD.md | Requisito Não Funcional | Logs com o logger já existente | TRANSCRICAO | [09:29] Bruno |
| PRD-NFR-12 | docs/PRD.md | Requisito Não Funcional | Não existe mudança de status sem evento registrado | TRANSCRICAO | [09:40] Bruno |
| PRD-NFR-13 | docs/PRD.md | Requisito Não Funcional | Entrega at-least-once | TRANSCRICAO | [09:24] Diego |
| PRD-NFR-14 | docs/PRD.md | Requisito Não Funcional | Ordem por pedido enquanto houver um único worker | TRANSCRICAO | [09:13] Larissa |
| PRD-NFR-15 | docs/PRD.md | Requisito Não Funcional | Evento reflete o pedido no momento da mudança | TRANSCRICAO | [09:52] Larissa |
| PRD-NFR-16 | docs/PRD.md | Restrição | MySQL e stack atuais, sem infraestrutura nova | TRANSCRICAO | [09:07] Diego |
| PRD-NFR-17 | docs/PRD.md | Restrição | Módulo segue os padrões de código do projeto | TRANSCRICAO | [09:30] Larissa |
| PRD-RISK-01 | docs/PRD.md | Risco | Atraso na entrega e saída da Atlas para o concorrente | TRANSCRICAO | [09:00] Marcos |
| PRD-RISK-02 | docs/PRD.md | Risco | Cliente indisponível além da janela de tentativas | TRANSCRICAO | [09:17] Marcos |
| PRD-RISK-03 | docs/PRD.md | Risco | Vazamento da secret de um cliente | TRANSCRICAO | [09:22] Diego |
| PRD-RISK-04 | docs/PRD.md | Risco | Cliente não trata eventos duplicados | TRANSCRICAO | [09:25] Sofia |
| PRD-RISK-05 | docs/PRD.md | Risco | Rajadas de notificações para um cliente | TRANSCRICAO | [09:38] Diego |
| PRD-AC-01 | docs/PRD.md | Critério de Aceite | Notificação recebida em menos de 10 segundos | TRANSCRICAO | [09:02] Marcos |
| PRD-AC-02 | docs/PRD.md | Critério de Aceite | Cadastrar, editar, remover e listar endpoints pela API | TRANSCRICAO | [09:33] Bruno |
| PRD-AC-03 | docs/PRD.md | Critério de Aceite | Secret devolvida na criação | TRANSCRICAO | [09:31] Marcos |
| PRD-AC-04 | docs/PRD.md | Critério de Aceite | Cliente só recebe os status escolhidos | TRANSCRICAO | [09:33] Marcos |
| PRD-AC-05 | docs/PRD.md | Critério de Aceite | URL sem https é recusada | TRANSCRICAO | [09:23] Sofia |
| PRD-AC-06 | docs/PRD.md | Critério de Aceite | Notificação com assinatura e identificador único | TRANSCRICAO | [09:22] Sofia |
| PRD-AC-07 | docs/PRD.md | Critério de Aceite | Retentativas nos intervalos definidos e DLQ ao final | TRANSCRICAO | [09:17] Larissa |
| PRD-AC-08 | docs/PRD.md | Critério de Aceite | Replay só por ADMIN, com registro | TRANSCRICAO | [09:36] Sofia |
| PRD-AC-09 | docs/PRD.md | Critério de Aceite | Consulta ao histórico de entregas | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-10 | docs/PRD.md | Critério de Aceite | Secret antiga funciona por 24h após a rotação | TRANSCRICAO | [09:21] Sofia |
| PRD-AC-11 | docs/PRD.md | Critério de Aceite | Cliente fora do ar não afeta mudança de status | TRANSCRICAO | [09:04] Bruno |
| PRD-AC-12 | docs/PRD.md | Critério de Aceite | Revisão de segurança concluída antes do deploy | TRANSCRICAO | [09:46] Sofia |
| FDD-FLUXO-01 | docs/FDD.md | Decisão | Evento inserido na outbox dentro da transação de changeStatus, via publishWebhookEvent | TRANSCRICAO | [09:41] Bruno |
| FDD-FLUXO-01 | docs/FDD.md | Integração | Transação atual de changeStatus onde a inserção acontece | CODIGO | src/modules/orders/order.service.ts |
| FDD-FLUXO-02 | docs/FDD.md | Decisão | Worker busca os pendentes mais antigos a cada 2 segundos, envia e marca | TRANSCRICAO | [09:09] Diego |
| FDD-FLUXO-03 | docs/FDD.md | Decisão | Retry com 5 tentativas e intervalos 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Larissa |
| FDD-FLUXO-04 | docs/FDD.md | Decisão | DLQ em webhook_dead_letter e replay manual que recoloca como pendente | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-05 | docs/FDD.md | Decisão | Rotação de secret com a antiga válida por 24 horas | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | POST de cadastro de webhook, secret devolvida na criação | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato | GET para listar os webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | PATCH para editar webhook | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04 | docs/FDD.md | Contrato | DELETE para remover webhook | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-05 | docs/FDD.md | Contrato | Endpoint para o cliente pedir nova secret | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | GET /webhooks/:id/deliveries com as últimas 100 entregas | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | POST /admin/webhooks/dead-letter/:id/replay, role ADMIN | TRANSCRICAO | [09:35] Diego |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | POST de saída: payload JSON e headers X-Event-Id, X-Signature, X-Timestamp, X-Webhook-Id | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | Campos do payload: event_id, event_type, timestamp, pedido, status e total_cents | TRANSCRICAO | [09:43] Diego |
| FDD-ERR-01 | docs/FDD.md | Erro | WEBHOOK_NOT_FOUND | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-02 | docs/FDD.md | Erro | WEBHOOK_INVALID_URL quando a URL não é https | TRANSCRICAO | [09:23] Sofia |
| FDD-ERR-03 | docs/FDD.md | Erro | WEBHOOK_SECRET_REQUIRED, situação a confirmar | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-04 | docs/FDD.md | Erro | Payload acima de 64KB gera erro e não é enviado | TRANSCRICAO | [09:24] Larissa |
| FDD-ERR-05 | docs/FDD.md | Erro | Id de DLQ inexistente no replay | TRANSCRICAO | [09:18] Diego |
| FDD-ERR-06 | docs/FDD.md | Erro | Cliente sem resposta em 10 segundos é falha | TRANSCRICAO | [09:42] Diego |
| FDD-ERR-07 | docs/FDD.md | Erro | Falha de entrega com cliente indisponível | TRANSCRICAO | [09:15] Diego |
| FDD-RES-01 | docs/FDD.md | Requisito Não Funcional | Timeout de 10 segundos por chamada | TRANSCRICAO | [09:42] Diego |
| FDD-RES-02 | docs/FDD.md | Decisão | Retries com backoff 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Larissa |
| FDD-RES-03 | docs/FDD.md | Decisão | DLQ com reprocessamento manual | TRANSCRICAO | [09:18] Diego |
| FDD-RES-04 | docs/FDD.md | Decisão | Worker isolado do processo da API | TRANSCRICAO | [09:11] Diego |
| FDD-RES-05 | docs/FDD.md | Restrição | Nenhuma chamada HTTP dentro da transação do pedido | TRANSCRICAO | [09:04] Bruno |
| FDD-RES-06 | docs/FDD.md | Requisito Não Funcional | Limite de 64KB, erro em vez de truncar | TRANSCRICAO | [09:23] Sofia |
| FDD-OBS-01 | docs/FDD.md | Métrica | Entregas por resultado, sucesso ou falha | TRANSCRICAO | [09:34] Marcos |
| FDD-OBS-02 | docs/FDD.md | Métrica | Tempo de resposta do cliente | TRANSCRICAO | [09:34] Marcos |
| FDD-OBS-03 | docs/FDD.md | Métrica | Latência entre inserção na outbox e entrega | TRANSCRICAO | [09:02] Marcos |
| FDD-OBS-04 | docs/FDD.md | Métrica | Eventos pendentes na outbox | TRANSCRICAO | [09:07] Bruno |
| FDD-OBS-05 | docs/FDD.md | Métrica | Eventos movidos para a DLQ | TRANSCRICAO | [09:18] Diego |
| FDD-OBS-06 | docs/FDD.md | Métrica | Volume de eventos por endpoint, para decidir sobre rate limiting | TRANSCRICAO | [09:39] Diego |
| FDD-OBS-07 | docs/FDD.md | Requisito Não Funcional | Logs estruturados com Pino, sem biblioteca nova | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-07 | docs/FDD.md | Integração | Logger com redactPaths, que ainda não cobre secret | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-08 | docs/FDD.md | Requisito Não Funcional | Log de auditoria do replay com o usuário | TRANSCRICAO | [09:36] Sofia |
| FDD-OBS-09 | docs/FDD.md | Integração | Correlação por X-Request-Id já existente, base da proposta de tracing | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-INT-01 | docs/FDD.md | Integração | changeStatus chama publishWebhookEvent dentro da transação | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Integração | Transições válidas que são candidatas a evento | CODIGO | src/modules/orders/order.status.ts |
| FDD-INT-03 | docs/FDD.md | Integração | Novos models com UUID e índices, no padrão do schema | CODIGO | prisma/schema.prisma |
| FDD-INT-04 | docs/FDD.md | Integração | AppError como base dos erros WEBHOOK_* | CODIGO | src/shared/errors/app-error.ts |
| FDD-INT-05 | docs/FDD.md | Integração | Subclasses de erro que aceitam código próprio | CODIGO | src/shared/errors/http-errors.ts |
| FDD-INT-06 | docs/FDD.md | Integração | Error middleware trata WEBHOOK_* sem alteração | CODIGO | src/middlewares/error.middleware.ts |
| FDD-INT-07 | docs/FDD.md | Integração | authenticate nas rotas e requireRole('ADMIN') no replay | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-INT-08 | docs/FDD.md | Integração | validate com schemas Zod do módulo | CODIGO | src/middlewares/validate.middleware.ts |
| FDD-INT-09 | docs/FDD.md | Integração | Montagem manual do módulo em buildControllers | CODIGO | src/app.ts |
| FDD-INT-10 | docs/FDD.md | Integração | Registro das rotas do módulo sob /api/v1 | CODIGO | src/routes/index.ts |
| FDD-INT-11 | docs/FDD.md | Integração | server.ts como modelo para o novo src/worker.ts | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-12 | docs/FDD.md | Integração | Worker com instância própria de PrismaClient, mesma DATABASE_URL | TRANSCRICAO | [09:30] Bruno |
| FDD-INT-13 | docs/FDD.md | Integração | Reuso do logger Pino no módulo e no worker | TRANSCRICAO | [09:29] Bruno |
| FDD-INT-14 | docs/FDD.md | Integração | paginated() nas listagens de webhooks e entregas | CODIGO | src/shared/http/response.ts |
| FDD-INT-15 | docs/FDD.md | Integração | Novo script npm run worker | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-16 | docs/FDD.md | Integração | Limpeza das novas tabelas antes de cada teste | CODIGO | tests/setup.ts |
| FDD-INT-17 | docs/FDD.md | Integração | Testes ponta a ponta da integração em changeStatus | TRANSCRICAO | [09:46] Larissa |
| FDD-AC-01 | docs/FDD.md | Critério de Aceite | Outbox na mesma transação, com rollback se a inserção falhar | TRANSCRICAO | [09:40] Bruno |
| FDD-AC-02 | docs/FDD.md | Critério de Aceite | Sem endpoint interessado, nada é inserido | TRANSCRICAO | [09:34] Bruno |
| FDD-AC-03 | docs/FDD.md | Critério de Aceite | Evento pendente enviado em menos de 10 segundos | TRANSCRICAO | [09:02] Marcos |
| FDD-AC-04 | docs/FDD.md | Critério de Aceite | Worker em processo separado, via npm run worker | TRANSCRICAO | [09:11] Diego |
| FDD-AC-05 | docs/FDD.md | Critério de Aceite | Headers obrigatórios em toda requisição de saída | TRANSCRICAO | [09:44] Diego |
| FDD-AC-06 | docs/FDD.md | Critério de Aceite | X-Signature confere com o HMAC-SHA256 do corpo | TRANSCRICAO | [09:22] Sofia |
| FDD-AC-07 | docs/FDD.md | Critério de Aceite | Sem resposta em 10 segundos gera tentativa com falha | TRANSCRICAO | [09:42] Diego |
| FDD-AC-08 | docs/FDD.md | Critério de Aceite | Intervalos de retry respeitados e DLQ ao final | TRANSCRICAO | [09:17] Larissa |
| FDD-AC-09 | docs/FDD.md | Critério de Aceite | Replay por ADMIN com log, 403 para OPERATOR | TRANSCRICAO | [09:36] Sofia |
| FDD-AC-10 | docs/FDD.md | Critério de Aceite | URL http recusada com erro de validação | TRANSCRICAO | [09:23] Sofia |
| FDD-AC-11 | docs/FDD.md | Critério de Aceite | Secret devolvida na criação e grace period de 24h | TRANSCRICAO | [09:21] Sofia |
| FDD-AC-12 | docs/FDD.md | Critério de Aceite | Payload sem itens e idêntico em todas as tentativas | TRANSCRICAO | [09:43] Diego |
| FDD-AC-13 | docs/FDD.md | Critério de Aceite | Deliveries mostra resultado, payload, response e tempo | TRANSCRICAO | [09:34] Marcos |
| FDD-AC-14 | docs/FDD.md | Critério de Aceite | Erros com prefixo WEBHOOK_ no formato padrão | TRANSCRICAO | [09:29] Larissa |
| FDD-AC-15 | docs/FDD.md | Critério de Aceite | Eventos do mesmo pedido chegam em ordem com um worker | TRANSCRICAO | [09:12] Diego |
| FDD-AC-16 | docs/FDD.md | Critério de Aceite | Secret não aparece em log, seguindo a redação já feita pelo logger | CODIGO | src/shared/logger/index.ts |
| FDD-AC-17 | docs/FDD.md | Critério de Aceite | Revisão de segurança feita antes do deploy | TRANSCRICAO | [09:46] Sofia |
| FDD-RISK-01 | docs/FDD.md | Risco | Falha na outbox bloqueia mudanças de status | TRANSCRICAO | [09:40] Bruno |
| FDD-RISK-02 | docs/FDD.md | Risco | Perda da ordem ao escalar o worker | TRANSCRICAO | [09:12] Diego |
| FDD-RISK-03 | docs/FDD.md | Risco | Crescimento da outbox degrada o polling | TRANSCRICAO | [09:07] Bruno |
| FDD-RISK-04 | docs/FDD.md | Risco | Rajada de chamadas para um cliente | TRANSCRICAO | [09:38] Diego |
| FDD-RISK-05 | docs/FDD.md | Risco | Vazamento de secret | TRANSCRICAO | [09:22] Diego |
| FDD-RISK-06 | docs/FDD.md | Risco | Evento preso no status "processando" se o worker cair. Mitigação não discutida | TRANSCRICAO | [09:08] Diego |
