# ADR-002: Worker em processo separado com polling de 2 segundos

## Status

Aceito — decidido na reunião técnica de webhooks (`[09:10] Larissa`, `[09:11] Larissa`). Registrado em 2026-11-24.

**Decisores:** Larissa, Diego, Bruno. Marcos aprovou a latência (`[09:10] Marcos`).

## Contexto

Com a outbox definida ([ADR-001](ADR-001-outbox-no-mysql.md)), é preciso decidir **quem lê a tabela e dispara as chamadas HTTP**, e **como** esse processo descobre que há eventos novos. O requisito de negócio é entrega em menos de 10 segundos (`[09:02] Marcos`).

Restrição levantada por Diego: o worker não pode viver na mesma instância da API, "senão se a API reinicia, perde o worker" (`[09:11] Diego`).

## Decisão

1. **Processo separado.** O worker é um entry-point novo, `src/worker.ts` (arquivo novo, a criar), ao lado de `src/server.ts`, iniciado por um script `npm run worker` (`[09:11] Larissa`). Usa o mesmo banco e a mesma stack, mas com uma instância própria de `PrismaClient`, pois o client é por processo: mesma `DATABASE_URL`, instância nova (`[09:30] Bruno`). A lógica de processamento fica em um arquivo do módulo, como `src/modules/webhooks/webhook.worker.ts` ou `webhook.processor.ts` (novos, a criar) (`[09:28] Bruno`).
2. **Polling em loop a cada 2 segundos**, buscando os eventos pendentes mais antigos, processando e marcando (`[09:09] Diego`). A latência mínima no pior caso é de 2 segundos, e isso foi aceito (`[09:10] Larissa`).
3. **Worker único (single-worker).** A ordem de envio segue o `created_at` da outbox. Isso dá ordering implícita **por `order_id`**, e não global (`[09:12] Diego`).

**Limitação conhecida (registrada na reunião):** se o worker for escalado para múltiplos processos em paralelo, a garantia de ordering se perde. Particionar por `order_id` ou usar lock pessimista seriam caminhos, mas "isso é problema do futuro" (`[09:13] Diego`, `[09:13] Larissa`). Os clientes nunca pediram ordering global (`[09:14] Marcos`).

## Alternativas Consideradas

1. **Trigger do MySQL para acionar o worker de forma reativa.** Descartada: o MySQL não oferece listener nativo tipo NOTIFY/LISTEN; a trigger só executa SQL e não notifica processo externo, o que exigiria improvisos como escrever em arquivo ou chamar um endpoint (`[09:09] Diego`).
2. **Worker embutido no processo da API.** Descartada: um restart da API derrubaria o worker (`[09:11] Diego`).
3. **Múltiplos workers em paralelo desde já.** Não adotada: perderia a garantia de ordering por `order_id` (`[09:12] Diego`). Adiada como evolução futura (`[09:13] Diego`).

## Consequências

**Positivas**

- API e worker têm ciclos de vida independentes: reiniciar um não afeta o outro.
- O polling de 2s atende "abaixo de 10 segundos" com folga (`[09:09] Diego`, `[09:10] Marcos`).
- Reuso de stack e banco; o novo entry-point espelha `src/server.ts`, que já faz bootstrap e shutdown gracioso (`SIGINT`/`SIGTERM`).

**Negativas / trade-offs**

- Latência mínima de até 2s e consultas periódicas ao banco, mesmo sem eventos: trocamos reatividade por simplicidade.
- Um novo processo para implantar e monitorar (script `npm run worker`).
- Ordering só vale por `order_id` e com worker único; escalar horizontalmente exigirá nova decisão.
- *Observação do redator (não discutida na reunião):* a interação entre ordering e retry com backoff (um evento em retry de 1 minuto a 12 horas, enquanto eventos posteriores do mesmo pedido seguem) não foi tratada. Deve ser levada ao RFC como questão em aberto, e não assumida aqui. Ver [ADR-003](ADR-003-retry-com-backoff-e-dlq.md).
- O tamanho do batch ("pequeno", `[09:08] Diego`) não foi definido; fica para o FDD.

## Referências

- Transcrição: `[09:09]`–`[09:14]`, `[09:28] Bruno`, `[09:30] Bruno`.
- Código: `src/server.ts`, `src/config/database.ts` (fábrica do `PrismaClient`), `package.json` (scripts).
