# ADR-001: Padrão Outbox no MySQL para registrar eventos de mudança de status

## Status

Aceito

Data: 2026-11-17
ADRs relacionados: [ADR-002](./ADR-002-worker-separado-com-polling.md), [ADR-003](./ADR-003-retry-com-backoff-e-dlq.md), [ADR-007](./ADR-007-payload-snapshot-na-insercao.md)

## Contexto

Três clientes B2B pediram para ser notificados quando o status dos pedidos muda. Hoje eles consultam `GET /orders` de tempos em tempos, o que deixa a integração lenta e cara ([09:00] Marcos). Para eles, qualquer atraso abaixo de 10 segundos já conta como tempo real ([09:02] Marcos). O OMS não tem hoje nenhum mecanismo de eventos, fila ou webhook.

A mudança de status acontece em `OrderService.changeStatus`, em `src/modules/orders/order.service.ts`. O método roda dentro de `prisma.$transaction`: valida a transição, movimenta estoque, atualiza o pedido e grava `OrderStatusHistory`. O time considera essa transação já pesada ([09:04] Bruno). A primeira pergunta da reunião foi se o disparo do webhook seria síncrono dentro desse service ou se passaria por fila ou outbox ([09:03] Larissa).

O ponto central é a consistência: não pode existir mudança de status confirmada sem evento registrado, nem evento registrado para uma mudança que sofreu rollback ([09:06] Diego, [09:40] Bruno).

### Direcionadores da decisão

- Um cliente lento não pode travar a mudança de status de outros pedidos ([09:04] Bruno).
- A indisponibilidade do cliente não pode causar rollback da mudança de status ([09:04] Bruno).
- O evento precisa nascer ou sumir junto com a transação do pedido ([09:06] Diego).
- O time é pequeno e não quer subir infraestrutura nova para esta feature ([09:07] Diego).
- A latência total precisa ficar abaixo de 10 segundos ([09:02] Marcos).

## Decisão

Opção escolhida: **padrão Outbox em uma tabela do MySQL existente**, porque a gravação do evento na mesma transação SQL da mudança de status elimina a inconsistência entre pedido e evento sem exigir infraestrutura nova ([09:06] Diego, [09:07] Diego, [09:08] Larissa).

O evento é inserido em uma tabela do tipo `webhook_outbox` dentro da transação de `changeStatus`. Se a inserção na outbox falhar, a transação inteira sofre rollback ([09:40] Bruno). A entrega HTTP fica fora da transação e é feita por um worker separado (ver [ADR-002](./ADR-002-worker-separado-com-polling.md)).

## Alternativas Consideradas

### Opção 1: Outbox em tabela do MySQL existente (escolhida)

- Prós: atomicidade entre mudança de status e registro do evento ([09:06] Diego).
- Prós: reaproveita o banco e a stack atuais, sem infraestrutura nova ([09:07] Diego).
- Prós: a chamada HTTP sai do caminho da transação do pedido.
- Contras: a tabela cresce com o tempo e depende de índices e de leitura em lotes pequenos ([09:07] Bruno, [09:08] Diego).
- Contras: a entrega passa a depender de um worker lendo a tabela, com latência de polling.

### Opção 2: Disparo síncrono dentro do service de pedidos

- Prós: implementação mais direta, sem tabela nem worker.
- Contras: acrescenta uma chamada HTTP no meio de uma transação já pesada ([09:04] Bruno).
- Contras: cliente lento trava a mudança de status de outros pedidos ([09:04] Bruno).
- Contras: cliente fora do ar forçaria rollback do status, o que não é aceitável ([09:04] Bruno).
- Motivo do descarte: "Síncrono está fora de questão" ([09:06] Diego).

### Opção 3: Redis Streams ou broker equivalente

- Prós: mecanismo próprio para consumo de eventos.
- Contras: exige subir e operar mais infraestrutura ([09:07] Larissa).
- Contras: não participa da transação SQL do pedido.
- Motivo do descarte: subir Redis Cluster para isso é overengineering para um time pequeno ([09:07] Diego).

## Consequências

**Positivas**
- Se a transação do pedido foi confirmada, o evento está registrado. Se sofreu rollback, o evento some junto ([09:06] Diego).
- Nenhum componente novo de infraestrutura entra em produção.
- A mudança de status deixa de depender da disponibilidade do cliente.

**Negativas**
- A transação de `changeStatus` ganha mais uma escrita, e uma falha na inserção da outbox derruba a mudança de status ([09:40] Bruno).
- A tabela acumula linhas. O arquivamento de linhas entregues (algo como 30 dias) foi deixado fora do escopo desta feature ([09:08] Diego).
- O desempenho do worker depende de índices no status e em `created_at` da outbox ([09:08] Diego).

**Trade-off explícito:** ganhamos consistência forte entre pedido e evento e zero infraestrutura nova. Aceitamos uma escrita a mais na transação crítica e a gestão de crescimento de uma tabela no banco transacional.

## Referências

- Transcrição: [09:03] Larissa, [09:04] Bruno, [09:06] Diego, [09:07] Diego, [09:08] Larissa, [09:40] Bruno
- Código: `src/modules/orders/order.service.ts` (método `changeStatus`), `prisma/schema.prisma` (models `Order` e `OrderStatusHistory`)
- Documentos: [RFC](../RFC.md), [FDD](../FDD.md)
