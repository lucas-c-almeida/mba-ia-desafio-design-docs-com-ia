# Architectural Decision Records

Este diretório guarda os ADRs do Sistema de Webhooks de Notificação de Pedidos.
Cada decisão arquitetural fica em um arquivo próprio, nomeado como `ADR-NNN-titulo-em-kebab-case.md`.

| ADR | Decisão | Status |
|---|---|---|
| [ADR-001](./ADR-001-outbox-no-mysql.md) | Padrão Outbox no MySQL | Aceito |
| [ADR-002](./ADR-002-worker-separado-com-polling.md) | Worker em processo separado com polling de 2 segundos | Aceito |
| [ADR-003](./ADR-003-retry-com-backoff-e-dlq.md) | Retry com backoff exponencial e DLQ em tabela separada | Aceito |
| [ADR-004](./ADR-004-at-least-once-com-x-event-id.md) | Garantia at-least-once com `X-Event-Id` | Aceito |
| [ADR-005](./ADR-005-hmac-sha256-secret-por-endpoint.md) | HMAC-SHA256 com secret por endpoint e rotação | Aceito |
| [ADR-006](./ADR-006-reuso-dos-padroes-do-projeto.md) | Reuso dos padrões existentes do projeto | Aceito |
| [ADR-007](./ADR-007-payload-snapshot-na-insercao.md) | Payload gravado como snapshot na inserção | Aceito |
