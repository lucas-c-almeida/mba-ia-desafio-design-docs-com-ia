# ADR-004: Garantia at-least-once com X-Event-Id para deduplicação no cliente

## Status

Aceito

Data: 2026-11-17
ADRs relacionados: [ADR-001](./ADR-001-outbox-no-mysql.md), [ADR-003](./ADR-003-retry-com-backoff-e-dlq.md), [ADR-005](./ADR-005-hmac-sha256-secret-por-endpoint.md)

## Contexto

Com outbox, worker e retry, o mesmo evento pode chegar ao cliente mais de uma vez. Basta, por exemplo, uma entrega bem-sucedida cuja confirmação não chegue a tempo e gere nova tentativa. A reunião precisou definir qual garantia de entrega o sistema oferece e como o cliente distingue uma repetição ([09:24] Diego, [09:25] Bruno).

Garantir entrega exatamente uma vez exigiria coordenação entre os dois lados, e os clientes são sistemas de terceiros fora do nosso controle ([09:25] Diego).

### Direcionadores da decisão

- Nenhum evento de mudança de status pode deixar de ser entregue por causa de falha transitória.
- O cliente precisa conseguir identificar uma entrega repetida ([09:25] Bruno).
- A solução deve seguir um padrão que os clientes já conhecem ([09:25] Diego).
- A complexidade precisa caber em um time pequeno ([09:07] Diego).

## Decisão

Opção escolhida: **entrega at-least-once, com um identificador único por evento enviado no header `X-Event-Id`**, porque é o padrão de mercado e resolve a grande maioria dos casos sem exigir coordenação com o cliente ([09:25] Diego, [09:26] Larissa).

O identificador é um UUID gerado quando o evento entra na outbox e é único por evento. O cliente é responsável por deduplicar pelo `event_id` do lado dele ([09:25] Diego).

## Alternativas Consideradas

### Opção 1: At-least-once com X-Event-Id (escolhida)

- Prós: padrão de mercado, usado por Stripe e GitHub ([09:25] Diego).
- Prós: não exige estado compartilhado nem protocolo extra com o cliente.
- Prós: combina naturalmente com retry e replay de DLQ.
- Contras: o cliente pode receber o mesmo evento duas vezes ([09:24] Diego).
- Contras: a deduplicação vira responsabilidade do cliente ([09:25] Sofia).

### Opção 2: Exactly-once

- Prós: o cliente nunca veria duplicatas.
- Contras: exige coordenação dos dois lados ([09:25] Diego).
- Contras: fica muito mais complexo de construir e operar ([09:25] Diego).
- Motivo do descarte: complexidade desproporcional, já que at-least-once com `event_id` resolve 99% dos casos ([09:25] Diego).

## Consequências

**Positivas**
- O contrato de entrega fica simples e explícito para os clientes.
- Retry e replay manual não precisam de tratamento especial para evitar duplicatas.

**Negativas**
- Parte da responsabilidade vai para o cliente, que precisa estar preparado para duplicatas ([09:25] Sofia, [09:24] Diego).
- A garantia depende de documentação clara. O PM assumiu documentar esse ponto com destaque no portal de desenvolvedor ([09:26] Marcos).

**Trade-off explícito:** ganhamos simplicidade e aderência ao padrão de mercado. Aceitamos que duplicatas podem acontecer e que o tratamento delas fica do lado do cliente.

## Referências

- Transcrição: [09:24] Diego, [09:25] Diego, [09:25] Sofia, [09:26] Marcos, [09:26] Larissa
- Documentos: [RFC](../RFC.md), [FDD](../FDD.md)
