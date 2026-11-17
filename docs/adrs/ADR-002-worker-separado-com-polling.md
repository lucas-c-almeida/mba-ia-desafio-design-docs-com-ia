# ADR-002: Worker em processo separado com polling de 2 segundos

## Status

Aceito

Data: 2026-11-17
ADRs relacionados: [ADR-001](./ADR-001-outbox-no-mysql.md), [ADR-003](./ADR-003-retry-com-backoff-e-dlq.md), [ADR-006](./ADR-006-reuso-dos-padroes-do-projeto.md)

## Contexto

Com a outbox no MySQL decidida ([ADR-001](./ADR-001-outbox-no-mysql.md)), falta definir quem lê a tabela e como ([09:08] Larissa). O requisito de negócio é entregar a notificação em menos de 10 segundos ([09:02] Marcos).

Hoje a aplicação tem um único ponto de entrada, `src/server.ts`, que sobe a API Express e usa o `PrismaClient` exportado por `src/config/database.ts`. Não existe processo de background no projeto.

Também foi discutida a ordem de entrega: se um pedido passa por vários status em sequência rápida, o cliente precisa receber os eventos na ordem certa ([09:12] Larissa).

### Direcionadores da decisão

- A latência total precisa ficar abaixo de 10 segundos ([09:02] Marcos).
- Um reinício da API não pode derrubar o processamento de eventos ([09:11] Diego).
- O MySQL não oferece notificação nativa para processos externos ([09:09] Diego).
- O worker deve usar o mesmo banco e a mesma stack ([09:11] Diego).
- Os clientes precisam receber os eventos de cada pedido em ordem ([09:12] Larissa).

## Decisão

Opção escolhida: **worker em processo separado da API, lendo a outbox em polling a cada 2 segundos**, porque o polling atende com folga o requisito de 10 segundos sem improvisos no banco, e o processo separado isola o processamento dos reinícios da API ([09:09] Diego, [09:10] Larissa, [09:11] Diego).

O worker ganha um ponto de entrada próprio no projeto (um `src/worker.ts` com script `npm run worker`), nos moldes de `src/server.ts` ([09:11] Larissa). Ele se conecta ao mesmo banco com a mesma `DATABASE_URL`, mas com uma instância própria de `PrismaClient`, porque o client é por processo ([09:30] Bruno).

Nesta fase roda um único worker, que processa os eventos por ordem de `created_at` ([09:12] Diego).

## Alternativas Consideradas

### Opção 1: Processo separado com polling de 2 segundos (escolhida)

- Prós: atende o requisito de menos de 10 segundos ([09:09] Diego, [09:10] Marcos).
- Prós: sobrevive a reinícios da API ([09:11] Diego).
- Prós: usa só MySQL e a stack atual ([09:11] Diego).
- Contras: latência de até 2 segundos no pior caso antes do primeiro envio ([09:10] Larissa).
- Contras: mais um processo para subir e operar.

### Opção 2: Trigger do banco para reagir à inserção

- Prós: seria mais reativo que polling ([09:09] Bruno).
- Contras: o MySQL não tem listener nativo como o NOTIFY/LISTEN do Postgres ([09:09] Diego).
- Contras: a trigger só executa SQL e não notifica processo externo ([09:09] Diego).
- Motivo do descarte: avisar o worker exigiria improviso, como escrever em arquivo ou chamar um endpoint ([09:09] Diego).

### Opção 3: Worker dentro do processo da API

- Prós: nenhum processo novo para operar.
- Contras: se a API reinicia, o worker cai junto ([09:11] Diego).
- Motivo do descarte: "Só não pode ser o mesmo processo" ([09:11] Diego).

## Consequências

**Positivas**
- Latência previsível: no pior caso, 2 segundos de espera até o worker pegar o evento ([09:10] Larissa).
- API e worker falham e reiniciam de forma independente.
- Com um único worker, os eventos saem na ordem de `created_at`, o que preserva a ordem por pedido ([09:12] Diego).

**Negativas**
- A ordem não é garantida globalmente. Vale por `order_id` e apenas enquanto houver um único worker ([09:13] Larissa).
- Escalar para vários workers em paralelo quebra essa garantia. Particionar por `order_id` ou usar lock pessimista foram citados como caminhos futuros, sem decisão ([09:13] Diego).
- O projeto passa a ter dois processos Node e duas instâncias de `PrismaClient` no mesmo banco ([09:30] Bruno).

**Trade-off explícito:** ganhamos simplicidade e isolamento. Aceitamos latência mínima de até 2 segundos e uma limitação conhecida de escala horizontal do worker.

## Referências

- Transcrição: [09:09] Diego, [09:10] Larissa, [09:11] Diego, [09:11] Larissa, [09:12] Diego, [09:13] Larissa, [09:30] Bruno
- Código: `src/server.ts`, `src/config/database.ts`, `package.json` (scripts)
- Documentos: [RFC](../RFC.md), [FDD](../FDD.md)
