# ADR-005: Garantia de entrega at-least-once com deduplicação via X-Event-Id

## Status

Aceito — decidido na reunião técnica de webhooks (`[09:26] Larissa`). Registrado em 2026-11-24.

**Decisores:** Larissa, Diego. Sofia levantou o impacto de responsabilidade para o cliente (`[09:25] Sofia`). Marcos assumiu documentar no portal de desenvolvedores (`[09:26] Marcos`).

## Contexto

Com outbox e retry ([ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)), um evento pode ser entregue mais de uma vez, por exemplo se o cliente processou a chamada mas a resposta se perdeu, ou após replay da DLQ. É preciso definir a semântica de entrega prometida ao cliente e como ele distingue repetições.

## Decisão

- **Garantia at-least-once.** O cliente pode receber o mesmo evento duas vezes e "tem que estar preparado" (`[09:24] Diego`).
- Cada evento recebe um **UUID gerado quando entra na outbox**, enviado no header **`X-Event-Id`**. É único por evento, e o cliente deduplica por ele do lado dele (`[09:25] Diego`).
- Responsabilidade de idempotência é do **cliente**; Marcos documentará isso de forma destacada no portal (`[09:26] Marcos`).

## Alternativas Consideradas

1. **Exactly-once.** Descartada: exigiria coordenação dos dois lados e seria "muito mais complexo" (`[09:25] Diego`).
2. **At-most-once (sem retry).** Plausível, não discutida; incompatível com a política de retry decidida em [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) e com o objetivo de não perder notificações.

## Consequências

**Positivas**

- Simplicidade de implementação e alinhamento com o padrão de mercado (Stripe e GitHub, citados em `[09:25] Diego`).
- Nenhuma perda de evento por falha de rede ou indisponibilidade do cliente.
- O `X-Event-Id` permanece estável em retries e em replays da DLQ, já que é gerado na entrada na outbox (`[09:25] Diego`); a estabilidade em replay decorre do desenho, e a reunião não a afirma explicitamente.

**Negativas / trade-offs**

- Transfere responsabilidade ao cliente: quem não deduplica pode processar um evento duas vezes (`[09:25] Sofia`). Aceitamos em troca de evitar a complexidade de exactly-once, que Diego diz resolver "99% dos casos" (`[09:25] Diego`).
- Exige comunicação clara e destacada na documentação para integradores (`[09:26] Marcos`).

## Referências

- Transcrição: `[09:24]`–`[09:26]`, `[09:44] Diego` (demais headers).
- Código: sem arquivo existente envolvido; o UUID segue o padrão do projeto (`uuid` v11.0.3 em `package.json`, já usado em `src/middlewares/request-logger.middleware.ts`).
