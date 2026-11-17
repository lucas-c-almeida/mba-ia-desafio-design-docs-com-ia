# ADR-003: Retry com backoff exponencial e DLQ em tabela separada

## Status

Aceito

Data: 2026-11-17
ADRs relacionados: [ADR-001](./ADR-001-outbox-no-mysql.md), [ADR-002](./ADR-002-worker-separado-com-polling.md), [ADR-004](./ADR-004-at-least-once-com-x-event-id.md)

## Contexto

O destino do webhook é um endpoint do cliente, fora da nossa infraestrutura, e ele pode estar lento ou fora do ar. A reunião precisou definir o que fazer quando a entrega falha ([09:14] Larissa).

Já houve cliente com indisponibilidade de duas horas em manutenção planejada ([09:16] Diego). Ao mesmo tempo, um evento não pode ficar pendurado para sempre se o cliente sumiu ([09:15] Diego).

Também foi discutido onde guardar os eventos que esgotaram as tentativas: em uma tabela separada ou marcados como falha na própria outbox ([09:17] Larissa).

### Direcionadores da decisão

- Cobrir indisponibilidades de algumas horas do cliente ([09:16] Diego).
- Evitar evento pendurado para sempre ([09:15] Diego).
- Manter a leitura da outbox principal limpa ([09:18] Diego).
- Guardar evidência para debug e reprocessamento ([09:18] Diego).
- Controlar quem pode reprocessar, com auditoria ([09:36] Sofia).

## Decisão

Opção escolhida: **5 tentativas com backoff exponencial de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas, e depois DLQ em tabela separada**, porque essa janela cobre quase 15 horas entre a primeira falha e a última tentativa sem deixar o evento preso indefinidamente ([09:17] Diego, [09:17] Larissa).

Esgotadas as tentativas, o evento é considerado falha permanente e vai para uma tabela do tipo `webhook_dead_letter`, com payload, motivo da falha e timestamp ([09:15] Diego, [09:18] Diego). A opção de apenas marcar a linha como falha na própria outbox foi preterida em favor da tabela separada ([09:18] Diego).

O reprocessamento é manual, por um endpoint administrativo que recoloca o evento na outbox como pendente ([09:18] Diego). Esse endpoint exige role `ADMIN` e registra quem fez o replay ([09:36] Sofia, [09:36] Larissa). Uma chamada sem resposta em 10 segundos conta como falha e entra no retry ([09:42] Diego).

## Alternativas Consideradas

### Opção 1: 5 tentativas com backoff 1m/5m/30m/2h/12h e DLQ separada (escolhida)

- Prós: cobre quase 15 horas de indisponibilidade ([09:17] Diego).
- Prós: tem fim definido, sem evento preso para sempre ([09:15] Diego).
- Prós: a DLQ separada serve de evidência para debug e replay ([09:18] Diego).
- Contras: um cliente fora por mais de 15 horas depende de replay manual.
- Contras: mais uma tabela e um endpoint administrativo para manter.

### Opção 2: Retry indefinido com backoff

- Prós: nenhum evento é abandonado automaticamente.
- Contras: o evento fica pendurado para sempre se o cliente sumiu ([09:15] Diego).
- Motivo do descarte: risco de acúmulo sem fim na outbox ([09:15] Diego).

### Opção 3: 3 tentativas, política mais agressiva

- Prós: falhas permanentes são detectadas mais rápido ([09:16] Bruno).
- Contras: três tentativas em 30 minutos matariam o evento durante uma indisponibilidade de manhã ([09:16] Diego).
- Motivo do descarte: é pouco diante de manutenções planejadas de duas horas já vistas em clientes ([09:16] Diego).

## Consequências

**Positivas**
- Indisponibilidades comuns do cliente são absorvidas sem ação manual.
- A outbox principal fica só com eventos em andamento, e a DLQ concentra os casos que pedem investigação ([09:18] Diego).
- O replay é controlado e auditável ([09:36] Sofia).

**Negativas**
- Depois de quase 15 horas de falha, a entrega depende de alguém com role `ADMIN` acionar o replay. O PM considerou aceitável, pois um cliente fora por 15 horas já tem um problema sério do lado dele ([09:17] Marcos).
- Não existe aviso automático ao cliente quando o webhook dele falha. O envio de email foi deixado para uma próxima fase ([09:37] Larissa).
- Retentativas aumentam a chance de entrega duplicada, tratada em [ADR-004](./ADR-004-at-least-once-com-x-event-id.md).

**Trade-off explícito:** ganhamos resiliência a indisponibilidades de horas e um destino claro para falhas permanentes. Aceitamos intervenção manual após a quinta tentativa e a manutenção de uma tabela e de um endpoint a mais.

## Referências

- Transcrição: [09:15] Diego, [09:16] Diego, [09:17] Diego, [09:17] Larissa, [09:18] Diego, [09:36] Sofia, [09:42] Diego
- Código: `src/middlewares/auth.middleware.ts` (`requireRole`)
- Documentos: [RFC](../RFC.md), [FDD](../FDD.md)
