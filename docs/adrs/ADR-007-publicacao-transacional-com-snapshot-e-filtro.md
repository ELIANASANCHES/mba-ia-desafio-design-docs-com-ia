# ADR-007: Publicação do evento dentro da transação via publishWebhookEvent, com snapshot do payload e filtro na inserção

## Status

Aceito — decidido na reunião técnica de webhooks (`[09:41] Diego`, `[09:52] Bruno`). Registrado em 2026-11-24.

**Decisores:** Bruno, Diego, Larissa. Marcos definiu o filtro de eventos como requisito (`[09:33] Marcos`).

## Contexto

O [ADR-001](ADR-001-outbox-no-mysql.md) exige que o evento seja gravado na mesma transação da mudança de status. Falta decidir **como** o `OrderService` participa disso, **o que** a outbox guarda e **quando** o filtro de eventos por webhook é aplicado.

O trecho crítico é `OrderService.changeStatus` em `src/modules/orders/order.service.ts` (linhas 126-179). O `OrderService` hoje recebe apenas `OrderRepository` e `PrismaClient` e já usa o tipo `TxClient = Prisma.TransactionClient` internamente.

## Decisão

1. **Função `publishWebhookEvent(tx, order, fromStatus, toStatus)`**, chamada por `order.service.ts` dentro da transação existente, recebendo o client transacional `tx`. É uma função pura que recebe o `tx`, sem injetar um repository inteiro no `OrderService` (`[09:41] Bruno`, `[09:41] Diego`).
2. **Falha na outbox causa rollback** de toda a mudança de status (`[09:40] Bruno`).
3. **Snapshot do payload na inserção.** A outbox guarda o payload já renderizado; se o pedido mudar depois, o evento continua refletindo o estado do momento da mudança (`[09:52] Larissa`, `[09:52] Diego`, `[09:52] Bruno`).
4. **Filtro de eventos na inserção.** Cada webhook escolhe uma lista de status que quer ouvir; se nenhum webhook do customer quer aquele status, nem se insere linha na outbox (`[09:33] Marcos`, `[09:34] Bruno`, `[09:34] Diego`).
5. **Payload enxuto**: `event_id`, `event_type` (`order.status_changed`), timestamp ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos básicos como `total_cents`, **sem items**; detalhes via `GET /orders/:id` (`[09:43] Diego`). Contrato detalhado no FDD.

## Alternativas Consideradas

1. **Guardar só o `order_id` na outbox e renderizar o payload no envio.** Descartada: o evento refletiria o estado atual do pedido, e não o do momento da mudança, gerando "caso esquisito" (`[09:52] Larissa`).
2. **Injetar o repository de webhooks no `OrderService`.** Descartada em favor de função que recebe `tx` (`[09:41] Bruno`, `[09:41] Diego`).
3. **Filtrar apenas no momento do envio.** Descartada: gastaria linhas da outbox com eventos que ninguém quer (`[09:34] Bruno`).

## Consequências

**Positivas**

- Garantia forte: não existe mudança de status sem evento (quando há webhook interessado) nem evento sem mudança (`[09:41] Diego`).
- Acoplamento mínimo: o `OrderService` conhece uma função, e não o módulo de webhooks.
- O payload é imutável e auditável, e a outbox cresce menos graças ao filtro.

**Negativas / trade-offs**

- Mudança em código de produção crítico, com risco de regressão nos testes existentes (`tests/orders.test.ts` cobre transições e estoque). O `OrderService` passa a depender do módulo novo `src/modules/webhooks/` (a criar).
- O snapshot duplica dados e não reflete correções posteriores ao pedido; aceito em troca de consistência histórica (`[09:52] Larissa`).
- A consulta dos webhooks do customer ocorre dentro da transação de `changeStatus`, ampliando-a.
- *Observação do redator (não discutida na reunião):* `OrderService.create` também insere um registro de histórico para `PENDING` com `fromStatus: null`. A reunião falou em eventos de "mudança de status" e não define se a criação do pedido gera evento. Fica como questão em aberto para o RFC.

## Referências

- Transcrição: `[09:33]`–`[09:34]`, `[09:40]`–`[09:41]`, `[09:43]`, `[09:51]`–`[09:52]`.
- Código: `src/modules/orders/order.service.ts` (`changeStatus`, tipo `TxClient`), `src/modules/orders/order.repository.ts`, `tests/orders.test.ts`.
