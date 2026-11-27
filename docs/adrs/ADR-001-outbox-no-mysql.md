# ADR-001: Padrão Outbox transacional no MySQL existente

## Status

Aceito — decidido na reunião técnica de webhooks (`[09:08] Larissa`). Registrado em 2026-11-24.

**Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos). Marcos e Sofia presentes, sem objeção.

## Contexto

Três clientes B2B (Atlas Comercial, MaxDistribuição e Nova Cargo) pediram notificação quando o status de seus pedidos muda, hoje obtida por polling em `GET /orders` (`[09:00] Marcos`). Qualquer latência abaixo de 10 segundos é "tempo real" para eles (`[09:02] Marcos`).

O ponto de origem do evento é `OrderService.changeStatus` em `src/modules/orders/order.service.ts` (linhas 126-179), que executa em um único `prisma.$transaction`: debita ou repõe estoque, atualiza `orders` e insere em `order_status_history`. Duas restrições foram levantadas:

- Uma chamada HTTP dentro dessa transação, já "pesada", faria um cliente lento travar mudanças de status de outros pedidos (`[09:04] Bruno`).
- Se o cliente estiver fora do ar, não faz sentido dar rollback na mudança de status (`[09:04] Bruno`).

Era preciso garantir que, se a mudança de status foi confirmada, o evento foi registrado, e vice-versa (`[09:40] Bruno`, `[09:41] Diego`).

## Decisão

Adotar o **padrão Outbox no MySQL já existente**. Na mesma transação SQL que atualiza `orders` e `order_status_history`, insere-se uma linha na tabela `webhook_outbox` com o evento (`[09:06] Diego`). Um worker separado (ver [ADR-002](ADR-002-worker-separado-com-polling-de-2s.md)) lê essa tabela e dispara as chamadas HTTP.

Decisões de modelagem acordadas na reunião:

- A tabela tem índice no campo de status (`pendente`, `processando`, `falhou`, `entregue`) e em `created_at`; o worker lê apenas pendentes, em batch pequeno (`[09:08] Diego`).
- A chave primária é UUID, seguindo o padrão do projeto (`[09:51] Larissa`), coerente com `@id @default(uuid()) @db.Char(36)` em `prisma/schema.prisma`.
- Se a inserção na outbox falhar, a transação inteira sofre rollback: "não pode ter caso de status mudar e evento não sair" (`[09:40] Bruno`).

O arquivamento de linhas entregues (~30 dias) ficou explicitamente **fora do escopo** desta feature (`[09:08] Diego`).

## Alternativas Consideradas

1. **Disparo síncrono dentro do `changeStatus`.** Descartada: cliente lento ou indisponível bloquearia ou reverteria mudanças de status (`[09:04] Bruno`; `[09:06] Diego`: "síncrono está fora de questão").
2. **Fila externa (Redis Streams ou similar).** Descartada: exigiria subir infraestrutura nova, e para um time pequeno "Redis Cluster pra isso é overengineering" (`[09:07] Larissa`, `[09:07] Diego`).
3. **Trigger no banco para notificar o worker.** Descartada: o MySQL não tem listener nativo equivalente ao NOTIFY/LISTEN do Postgres, e a trigger só executa SQL (`[09:09] Diego`). Detalhada no [ADR-002](ADR-002-worker-separado-com-polling-de-2s.md).

## Consequências

**Positivas**

- Atomicidade: evento e mudança de status commitam ou revertem juntos, sem inconsistência possível (`[09:06] Diego`).
- Zero infraestrutura nova: reutiliza MySQL e Prisma já em produção.
- Falhas de entrega ao cliente ficam isoladas da transação de pedidos.

**Negativas / trade-offs**

- A transação de `changeStatus` ganha mais uma escrita, e uma falha na outbox passa a impedir a mudança de status. Aceitamos essa perda de disponibilidade em troca da garantia de consistência (`[09:40] Bruno`).
- A outbox cresce no mesmo banco transacional; o arquivamento fica para outra feature (`[09:08] Diego`), então o crescimento da tabela é um risco conhecido.
- Entrega passa a ser assíncrona, com latência mínima definida pelo intervalo de polling.
- Exige mudança em código crítico existente (`order.service.ts`); o mecanismo está em [ADR-007](ADR-007-publicacao-transacional-com-snapshot-e-filtro.md).

## Referências

- Transcrição: `[09:03]`–`[09:08]`, `[09:40]`–`[09:41]`, `[09:51] Larissa`.
- Código: `src/modules/orders/order.service.ts` (`changeStatus`), `prisma/schema.prisma`.
