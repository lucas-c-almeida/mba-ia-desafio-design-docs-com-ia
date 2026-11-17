# ADR-006: Reuso dos padrões existentes do projeto no módulo de webhooks

## Status

Aceito

Data: 2026-11-17
ADRs relacionados: [ADR-001](./ADR-001-outbox-no-mysql.md), [ADR-002](./ADR-002-worker-separado-com-polling.md)

## Contexto

A codebase do OMS tem um padrão claro. Cada domínio é um módulo em `src/modules/<nome>/` com controller, service, repository, routes e schemas ([09:27] Bruno). É o que se vê em `src/modules/orders/`, com `order.routes.ts`, `order.controller.ts`, `order.service.ts`, `order.repository.ts` e `order.schemas.ts`. A montagem das dependências é manual, em `buildControllers()` de `src/app.ts`, e os routers são registrados em `src/routes/index.ts`.

Os erros seguem a classe `AppError` de `src/shared/errors/app-error.ts`, com subclasses em `src/shared/errors/http-errors.ts`, como `InsufficientStockError` e `InvalidStatusTransitionError`, e códigos como `INSUFFICIENT_STOCK` e `INVALID_STATUS_TRANSITION` ([09:28] Bruno). O middleware `src/middlewares/error.middleware.ts` já trata `AppError`, `ZodError` e erros do Prisma. O logger é o Pino configurado em `src/shared/logger/index.ts`, e a validação usa Zod com `src/middlewares/validate.middleware.ts`.

A feature de webhooks toca a parte mais crítica do sistema, `OrderService.changeStatus`, e ainda cria um módulo novo e um processo novo. A reunião precisou decidir se ela seguiria esses padrões ou traria estruturas próprias ([09:27] Larissa).

### Direcionadores da decisão

- Manter a codebase consistente para um time pequeno ([09:07] Diego).
- Não introduzir ferramentas novas quando as atuais resolvem ([09:29] Bruno).
- Erros do módulo novo devem ser tratados pelo middleware atual sem alterá-lo ([09:29] Bruno).
- A integração com `changeStatus` precisa ser a menor e mais segura possível ([09:40] Bruno).
- A autorização deve reaproveitar o que já existe ([09:36] Larissa).

## Decisão

Opção escolhida: **reuso máximo dos padrões existentes**, porque `AppError`, Pino, o error middleware, o padrão de módulos, os schemas Zod e o padrão de códigos de erro já cobrem o que a feature precisa ([09:30] Larissa).

O webhook vira um módulo como os outros, em `src/modules/webhooks`, com a mesma divisão de camadas ([09:27] Bruno). Os códigos de erro do módulo usam o prefixo `WEBHOOK_` ([09:29] Larissa). O endpoint de replay reaproveita o `requireRole` de `src/middlewares/auth.middleware.ts` ([09:36] Larissa). Os identificadores das novas tabelas são UUID, como no resto do projeto ([09:51] Larissa), em linha com o `@default(uuid()) @db.Char(36)` usado em `prisma/schema.prisma`.

A integração com pedidos é feita por uma função que recebe o client da transação em curso, no formato `publishWebhookEvent(tx, order, fromStatus, toStatus)`, chamada por `OrderService.changeStatus` ([09:41] Bruno).

## Alternativas Consideradas

### Opção 1: Reuso máximo dos padrões existentes (escolhida)

- Prós: o módulo novo fica legível para quem já conhece os outros ([09:27] Bruno).
- Prós: erros `WEBHOOK_*` são tratados pelo error middleware sem mudança nele ([09:29] Bruno).
- Prós: nenhuma dependência nova para log, validação ou autorização ([09:29] Bruno).
- Contras: o módulo herda as limitações dos padrões atuais.
- Contras: o worker, que não é HTTP, precisa se encaixar em um padrão pensado para a API.

### Opção 2: Injetar um repository de webhooks inteiro no OrderService

- Prós: segue a forma como o `OrderService` já recebe o `OrderRepository` no construtor.
- Contras: acopla o service de pedidos a todo o repository de webhooks para usar uma única operação.
- Motivo do descarte: uma função que recebe o `tx` resolve sem precisar injetar o repository inteiro ([09:41] Diego).

### Opção 3: Estruturas e bibliotecas próprias para o módulo de webhooks

- Prós: liberdade para escolher ferramentas específicas para entrega de eventos.
- Contras: quebra a consistência da codebase e aumenta o custo de manutenção.
- Motivo do descarte: não chegou a ser proposta na reunião. A direção foi explícita no sentido contrário: "Não vamos botar nada novo" ([09:29] Bruno).

## Consequências

**Positivas**
- A curva de aprendizado do módulo novo é mínima para o time.
- A resposta de erro dos endpoints de webhook sai no mesmo formato dos demais.
- A mudança em `OrderService` fica restrita a uma chamada dentro da transação existente.

**Negativas**
- `src/app.ts` e `src/routes/index.ts` precisam ser alterados para registrar o módulo novo, pois a montagem é manual.
- `NotFoundError` e `ValidationError` têm código fixo (`NOT_FOUND` e `VALIDATION_ERROR`). Erros com código `WEBHOOK_*` precisam de classes próprias ou das classes que aceitam código, como `ConflictError` e `UnprocessableEntityError`.
- O `requireRole` hoje só é usado em `src/modules/users/user.routes.ts`. O replay de DLQ passa a ser o segundo ponto de uso.

**Trade-off explícito:** ganhamos consistência e velocidade de entrega. Aceitamos adaptar a feature aos padrões atuais em vez de otimizar cada parte com ferramentas próprias.

## Referências

- Transcrição: [09:27] Bruno, [09:28] Bruno, [09:29] Bruno, [09:29] Larissa, [09:30] Larissa, [09:36] Larissa, [09:41] Bruno, [09:51] Larissa
- Código: `src/modules/orders/order.service.ts`, `src/shared/errors/app-error.ts`, `src/shared/errors/http-errors.ts`, `src/middlewares/error.middleware.ts`, `src/middlewares/auth.middleware.ts`, `src/shared/logger/index.ts`, `src/app.ts`, `src/routes/index.ts`
- Documentos: [RFC](../RFC.md), [FDD, seção 9](../FDD.md#9-integração-com-o-sistema-existente)
