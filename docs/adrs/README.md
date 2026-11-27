# Architectural Decision Records

Este diretório armazena os ADRs (Architectural Decision Records) do projeto.
Cada decisão arquitetural relevante deve ser registrada aqui em arquivos individuais,
nomeados sequencialmente (por exemplo `0001-titulo-da-decisao.md`).

## Índice

| ADR | Decisão | Status |
| --- | --- | --- |
| [ADR-001](ADR-001-outbox-no-mysql.md) | Padrão Outbox transacional no MySQL existente | Aceito |
| [ADR-002](ADR-002-worker-separado-com-polling-de-2s.md) | Worker em processo separado com polling de 2 segundos | Aceito |
| [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) | Retry com backoff exponencial e DLQ em tabela separada | Aceito |
| [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md) | HMAC-SHA256 com secret por endpoint e rotação de 24h | Aceito |
| [ADR-005](ADR-005-at-least-once-com-x-event-id.md) | At-least-once com deduplicação via `X-Event-Id` | Aceito |
| [ADR-006](ADR-006-reuso-dos-padroes-existentes-do-projeto.md) | Reuso máximo dos padrões existentes do projeto | Aceito |
| [ADR-007](ADR-007-publicacao-transacional-com-snapshot-e-filtro.md) | Publicação transacional com `publishWebhookEvent`, snapshot e filtro na inserção | Aceito |

Convenção: caminhos marcados como "a criar" são arquivos novos da feature; os demais existem no repositório.
