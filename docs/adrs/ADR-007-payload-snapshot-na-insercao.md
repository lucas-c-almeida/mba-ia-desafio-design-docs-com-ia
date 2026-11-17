# ADR-007: Payload do evento gravado como snapshot na inserção da outbox

## Status

Aceito

Data: 2026-11-17
ADRs relacionados: [ADR-001](./ADR-001-outbox-no-mysql.md), [ADR-003](./ADR-003-retry-com-backoff-e-dlq.md)

## Contexto

O evento é inserido na outbox no momento da mudança de status, mas pode ser enviado bem depois: segundos mais tarde no caso normal, horas mais tarde em caso de retry, ou dias depois em um replay de DLQ (ver [ADR-003](./ADR-003-retry-com-backoff-e-dlq.md)). Nesse intervalo o pedido pode mudar de novo.

No fim da reunião surgiu a dúvida: a outbox guarda o payload já renderizado, ou guarda só o `order_id` e renderiza na hora do envio ([09:51] Bruno).

### Direcionadores da decisão

- O evento deve refletir o estado do pedido no momento em que o status mudou ([09:52] Larissa).
- Evitar casos estranhos quando o pedido muda entre a inserção e o envio ([09:52] Larissa).
- Manter o payload enxuto ([09:44] Bruno).
- Retry e replay devem reenviar exatamente o mesmo conteúdo.

## Decisão

Opção escolhida: **gravar o payload já renderizado na inserção da outbox, como snapshot**, porque assim o evento continua refletindo o estado de quando o status mudou, mesmo que o pedido seja alterado depois ([09:52] Larissa, [09:52] Diego, [09:52] Bruno).

O payload é enxuto: traz identificação do evento e do pedido, os status de origem e destino e campos básicos do pedido, sem os itens. O cliente que quiser detalhes consulta `GET /orders/:id` depois ([09:43] Diego).

## Alternativas Consideradas

### Opção 1: Snapshot renderizado na inserção (escolhida)

- Prós: o evento é fiel ao momento da mudança de status ([09:52] Larissa).
- Prós: retry e replay reenviam conteúdo idêntico.
- Prós: o worker não precisa consultar o pedido para montar o envio.
- Contras: a outbox guarda mais dados por linha.
- Contras: mudanças futuras no formato do payload não se aplicam a eventos já gravados.

### Opção 2: Guardar só o order_id e renderizar no envio

- Prós: linhas menores na outbox.
- Prós: o payload sempre sai no formato mais recente.
- Contras: se o pedido mudar depois, o evento deixa de refletir o estado de quando o status mudou ([09:52] Larissa).
- Motivo do descarte: "Senão tem caso esquisito" ([09:52] Larissa).

## Consequências

**Positivas**
- Cada evento é um registro imutável do que aconteceu.
- A DLQ guarda o mesmo payload, o que ajuda no debug e no replay ([09:18] Diego).

**Negativas**
- O custo de montar o payload acontece dentro da transação de mudança de status.
- O tamanho do payload precisa de controle. O limite acordado é 64KB, com erro se for ultrapassado, tratado como requisito não funcional e não como decisão separada ([09:24] Diego, [09:24] Larissa).

**Trade-off explícito:** ganhamos fidelidade histórica e reenvio determinístico. Aceitamos linhas maiores na outbox e um pouco mais de trabalho dentro da transação do pedido.

## Referências

- Transcrição: [09:43] Diego, [09:51] Bruno, [09:52] Larissa, [09:52] Diego, [09:52] Bruno
- Código: `prisma/schema.prisma` (model `Order`, campos como `orderNumber` e `totalCents`)
- Documentos: [RFC](../RFC.md), [FDD](../FDD.md)
