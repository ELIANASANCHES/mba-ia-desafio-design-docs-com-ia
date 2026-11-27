# ADR-003: Retry com backoff exponencial e DLQ em tabela separada

## Status

Aceito — decidido na reunião técnica de webhooks (`[09:17] Larissa`; replay admin em `[09:19] Larissa` e `[09:36] Larissa`). Registrado em 2026-11-24.

**Decisores:** Larissa, Diego, Bruno, Sofia (replay restrito a ADMIN). Marcos considerou o prazo aceitável (`[09:17] Marcos`).

## Contexto

Endpoints de clientes podem estar offline ou lentos. É preciso definir quando e quantas vezes reenviar, o que fazer ao esgotar as tentativas e como recuperar eventos falhos. Já houve cliente com indisponibilidade de duas horas em manutenção planejada (`[09:16] Diego`).

## Decisão

1. **Backoff exponencial com 5 tentativas**, nos intervalos **1 min / 5 min / 30 min / 2 h / 12 h** (`[09:17] Larissa`, consolidado em `[09:48] Larissa`). Diego estima "quase 15 horas" entre a primeira falha e a última tentativa (`[09:17] Diego`).
2. **Timeout de 10 segundos** na chamada HTTP do worker; sem resposta em 10s, a tentativa é tratada como falha e marcada para retry (`[09:42] Diego`).
3. **DLQ em tabela separada**, `webhook_dead_letter`, com payload, motivo da falha e timestamp. Esgotadas as tentativas, o evento é movido para ela como falha permanente (`[09:15] Diego`, `[09:18] Diego`).
4. **Reprocessamento manual** via `POST /admin/webhooks/dead-letter/:id/replay`, que recoloca o evento na outbox como pendente (`[09:18] Diego`, `[09:35] Diego`). O endpoint exige **role `ADMIN`**, reaproveitando o `requireRole` existente, e **registra quem executou o replay** para auditoria (`[09:36] Sofia`, `[09:36] Larissa`).

**Ambiguidade tratada no FDD:** "5 tentativas" convive com 5 intervalos listados. A soma dos intervalos (1m+5m+30m+2h+12h = 14h36m) é compatível com o "quase 15 horas entre primeira falha e última tentativa" dito por Diego se a primeira falha for o ponto de partida, o que leva à leitura de **1 envio inicial + 5 retries** (até 6 chamadas). A transcrição não diz explicitamente se o envio inicial conta entre as 5; o FDD adota essa leitura como proposta (ver Q9 no RFC).

## Alternativas Consideradas

1. **Retry indefinido com backoff.** Descartada: o evento ficaria "pendurado pra sempre" se o cliente sumiu (`[09:15] Diego`).
2. **3 tentativas (mais agressivo).** Descartada: seriam três retries em ~30 minutos, insuficiente para indisponibilidades como a de duas horas em manutenção (`[09:16] Bruno`, `[09:16] Diego`).
3. **Marcar como `failed` na própria outbox, sem tabela separada.** Descartada: a tabela separada deixa a leitura da outbox principal mais limpa e serve de evidência para debug e reprocessamento (`[09:17] Larissa`, `[09:18] Diego`).
4. **Reprocessamento automático da DLQ.** Não discutida; a decisão foi por replay manual (`[09:18] Diego`).

## Consequências

**Positivas**

- Cobre janelas de indisponibilidade de até ~15h, com teto claro para a vida de um evento.
- A DLQ preserva payload e motivo para diagnóstico, e o replay é auditável.
- A outbox principal permanece enxuta: só eventos em fluxo normal.

**Negativas / trade-offs**

- Cliente fora do ar por mais de ~15h perde a entrega automática e depende de replay manual por um ADMIN; aceito pelo PM (`[09:17] Marcos`).
- Não há alerta ao cliente (e-mail foi adiado, `[09:37] Larissa`), então falhas só são vistas via histórico de entregas ou DLQ.
- Replay recoloca o evento na outbox e, com at-least-once ([ADR-005](ADR-005-at-least-once-com-x-event-id.md)), o cliente pode receber duplicata.
- Há mais uma tabela e um endpoint administrativo para manter.

## Referências

- Transcrição: `[09:14]`–`[09:19]`, `[09:35]`–`[09:36]`, `[09:42] Diego`, `[09:48] Larissa`.
- Código: `src/middlewares/auth.middleware.ts` (`requireRole`, `AuthUser.role` com `ADMIN`/`OPERATOR`), `prisma/schema.prisma` (enum `UserRole`).
