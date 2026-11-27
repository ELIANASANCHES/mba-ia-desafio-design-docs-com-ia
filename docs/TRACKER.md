# Tracker

Rastreabilidade dos documentos desta feature: cada item registrado no [PRD](PRD.md), no [RFC](RFC.md), no [FDD](FDD.md) e nos [ADRs](adrs/README.md) aponta para a origem, que é a transcrição da reunião (`TRANSCRICAO.md`) ou o código do repositório.

## Como ler

- **ID:** identificador do item. Para PRD, RFC e ADRs o prefixo é o documento (`PRD-RF-01`, `RFC-ALT-02`, `ADR-002`). Para o FDD, o ID é o do próprio documento (`FDD-CONTRATO-03`). Sufixos como `-ALT`, `-TRADE` e `-COD` indicam alternativa descartada, trade-off e fato do código associado a um ADR ou a um item do FDD.
- **Fonte `TRANSCRICAO`:** a localização é `[hh:mm] Nome`, conferido contra `TRANSCRICAO.md` (o horário e o falante existem). Havendo mais de uma fala, elas são separadas por `;`.
- **Fonte `CODIGO`:** a localização é o caminho de um arquivo existente no repositório (conferido).
- **Fonte `PROPOSTA`:** item que a reunião **não** discutiu e que foi preenchido na redação dos documentos para torná-los acionáveis. Não é decisão, e deve ser validado na revisão. Quando há uma fala relacionada, a localização a cita como `Relacionada a [hh:mm] Nome`, mas ela **não** é a origem do item. O uso deste valor, além de `TRANSCRICAO` e `CODIGO`, é deliberado: marcar uma sugestão como se tivesse vindo da reunião seria exatamente a alucinação que o tracker quer evitar.
- **Itens `(a criar)`** dos documentos (por exemplo `src/worker.ts`) não aparecem como fonte de código, pois ainda não existem.

## Resumo

| Indicador | Valor |
| --- | --- |
| Linhas no tracker | 354 |
| Itens identificáveis com linha (IDs do PRD e do FDD, ADRs, alternativas e questões do RFC) | 195 de 195 (100%) |
| Fonte TRANSCRICAO | 255 (72%) |
| Fonte CODIGO | 57 (16%), em 28 arquivos distintos |
| Fonte PROPOSTA (sem origem direta) | 42 (12%) |

## Tabela de rastreabilidade

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-OBJ-01 | docs/PRD.md | Objetivo / Métrica | Entregar o evento em menos de 10 s após o commit | TRANSCRICAO | [09:02] Marcos; [09:10] Larissa |
| PRD-OBJ-02 | docs/PRD.md | Objetivo / Métrica | Zero mudança de status sem evento e zero evento de mudança revertida | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| PRD-OBJ-03 | docs/PRD.md | Objetivo / Métrica | Zero mudança de status atrasada ou revertida por falha de endpoint de cliente | TRANSCRICAO | [09:04] Bruno |
| PRD-OBJ-04 | docs/PRD.md | Objetivo / Métrica | 5 retentativas em até 14h36 (1m/5m/30m/2h/12h), sem descartar evento | TRANSCRICAO | [09:15] Diego; [09:17] Larissa |
| PRD-OBJ-05 | docs/PRD.md | Objetivo / Métrica | 3 de 3 clientes com webhook em produção até o fim de novembro | TRANSCRICAO | [09:00] Marcos; [09:45] Marcos |
| PRD-OBJ-06 | docs/PRD.md | Objetivo / Métrica | Entregar em 3 sprints, com 2 dias úteis finais de revisão de segurança | TRANSCRICAO | [09:46] Larissa; [09:46] Sofia |
| PRD-OBJ-07 | docs/PRD.md | Objetivo / Métrica | Testes existentes de pedidos e autenticação 100% verdes (meta é Proposta) | CODIGO | tests/orders.test.ts; tests/auth.test.ts |
| PRD-RF-01 | docs/PRD.md | Requisito Funcional | Cadastrar webhook com URL e lista de status; secret gerada pelo sistema e devolvida na criação | TRANSCRICAO | [09:31] Marcos |
| PRD-RF-02 | docs/PRD.md | Requisito Funcional | customer_id no body/path/query, não do JWT | TRANSCRICAO | [09:32] Bruno; [09:32] Larissa |
| PRD-RF-03 | docs/PRD.md | Requisito Funcional | Editar (PATCH), remover (DELETE) e listar webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-04 | docs/PRD.md | Requisito Funcional | Filtro de status por endpoint aplicado na inserção da outbox | TRANSCRICAO | [09:33] Marcos; [09:34] Bruno |
| PRD-RF-05 | docs/PRD.md | Requisito Funcional | Evento gravado na outbox na mesma transação; falha na outbox reverte o status | TRANSCRICAO | [09:03] Larissa; [09:06] Diego; [09:40] Bruno |
| PRD-RF-06 | docs/PRD.md | Requisito Funcional | Snapshot do payload gravado no momento da mudança | TRANSCRICAO | [09:51] Bruno; [09:52] Larissa |
| PRD-RF-07 | docs/PRD.md | Requisito Funcional | Worker em processo separado, polling de 2 s, entrega por HTTPS (src/worker.ts, npm run worker) | TRANSCRICAO | [09:09] Diego; [09:11] Diego; [09:11] Larissa |
| PRD-RF-08 | docs/PRD.md | Requisito Funcional | Timeout de 10 s na entrega; sem resposta é falha com retry | TRANSCRICAO | [09:42] Diego |
| PRD-RF-09 | docs/PRD.md | Requisito Funcional | Backoff exponencial 1m/5m/30m/2h/12h (5 tentativas) | TRANSCRICAO | [09:15] Diego; [09:17] Larissa |
| PRD-RF-10 | docs/PRD.md | Requisito Funcional | Esgotadas as tentativas, evento vai à DLQ em tabela separada com payload, motivo e timestamp | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| PRD-RF-11 | docs/PRD.md | Requisito Funcional | Replay de DLQ por POST /admin/webhooks/dead-letter/:id/replay, ADMIN, com registro de quem executou | TRANSCRICAO | [09:18] Diego; [09:35] Diego; [09:36] Sofia |
| PRD-RF-12 | docs/PRD.md | Requisito Funcional | HMAC-SHA256 sobre o corpo em X-Signature, secret única por endpoint | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| PRD-RF-13 | docs/PRD.md | Requisito Funcional | Rotação de secret pela API, antiga válida por 24 h | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| PRD-RF-14 | docs/PRD.md | Requisito Funcional | Histórico de entregas (últimos 100) em GET /webhooks/:id/deliveries | TRANSCRICAO | [09:34] Marcos |
| PRD-RF-15 | docs/PRD.md | Requisito Funcional | At-least-once com X-Event-Id (UUID gerado na entrada da outbox) | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa |
| PRD-RF-16 | docs/PRD.md | Requisito Funcional | Headers X-Event-Id, X-Signature, X-Timestamp, X-Webhook-Id e Content-Type application/json | TRANSCRICAO | [09:44] Diego; [09:44] Sofia; [09:45] Diego |
| PRD-RF-17 | docs/PRD.md | Requisito Funcional | Payload JSON com event_id, event_type, timestamp, order_id, order_number, from/to_status, customer_id, total_cents, sem itens | TRANSCRICAO | [09:43] Diego |
| PRD-RF-18 | docs/PRD.md | Requisito Funcional | URL do webhook obrigatoriamente https; http recusado | TRANSCRICAO | [09:23] Sofia |
| PRD-RF-19 | docs/PRD.md | Requisito Funcional | Corpo acima de 64 KB não é enviado e gera erro | TRANSCRICAO | [09:23] Sofia; [09:24] Diego; [09:24] Larissa |
| PRD-RF-20 | docs/PRD.md | Requisito Funcional | Erros do módulo com AppError e códigos prefixados por WEBHOOK_ | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa |
| PRD-RNF-01 | docs/PRD.md | Requisito Não Funcional | Latência abaixo de 10 s | TRANSCRICAO | [09:02] Marcos; [09:10] Larissa |
| PRD-RNF-02 | docs/PRD.md | Requisito Não Funcional | Atomicidade evento + status (mesma transação SQL) | TRANSCRICAO | [09:06] Diego; [09:40] Bruno |
| PRD-RNF-03 | docs/PRD.md | Requisito Não Funcional | Isolamento: sem HTTP na transação de pedidos; API e worker independentes | TRANSCRICAO | [09:04] Bruno; [09:11] Diego |
| PRD-RNF-04 | docs/PRD.md | Requisito Não Funcional | At-least-once; deduplicação a cargo do cliente | TRANSCRICAO | [09:24] Diego; [09:26] Larissa |
| PRD-RNF-05 | docs/PRD.md | Requisito Não Funcional | Ordering só por order_id e com worker único | TRANSCRICAO | [09:12] Diego; [09:13] Larissa; [09:14] Marcos |
| PRD-RNF-06 | docs/PRD.md | Requisito Não Funcional | HMAC-SHA256, secret por endpoint, rotação com 24 h | TRANSCRICAO | [09:19] Sofia; [09:22] Sofia |
| PRD-RNF-07 | docs/PRD.md | Requisito Não Funcional | TLS obrigatório (https) | TRANSCRICAO | [09:23] Sofia |
| PRD-RNF-08 | docs/PRD.md | Requisito Não Funcional | CRUD para qualquer role autenticada; replay só ADMIN via requireRole | TRANSCRICAO | [09:36] Larissa; [09:37] Sofia |
| PRD-RNF-09 | docs/PRD.md | Requisito Não Funcional | Auditoria: replay registra quem executou | TRANSCRICAO | [09:36] Sofia |
| PRD-RNF-10 | docs/PRD.md | Requisito Não Funcional | Limite de payload de 64 KB | TRANSCRICAO | [09:24] Larissa |
| PRD-RNF-11 | docs/PRD.md | Requisito Não Funcional | Reuso de padrões: módulo, Pino, error middleware, Zod, UUID | TRANSCRICAO | [09:27] Bruno; [09:30] Larissa; [09:51] Larissa |
| PRD-RNF-12 | docs/PRD.md | Requisito Não Funcional | Sem infraestrutura nova; worker com PrismaClient próprio na mesma DATABASE_URL | TRANSCRICAO | [09:07] Diego; [09:30] Bruno |
| PRD-RNF-13 | docs/PRD.md | Requisito Não Funcional | Revisão de segurança de no mínimo 2 dias úteis antes do deploy | TRANSCRICAO | [09:46] Sofia; [09:49] Sofia |
| PRD-RNF-14 | docs/PRD.md | Requisito Não Funcional | Observabilidade: logs em Pino; métricas por SQL e logs, sem dependência nova | TRANSCRICAO | [09:29] Bruno |
| PRD-ESC-01 | docs/PRD.md | Fora de escopo | E-mail ao cliente quando o webhook falha | TRANSCRICAO | [09:37] Larissa; [09:38] Marcos |
| PRD-ESC-02 | docs/PRD.md | Fora de escopo | Rate limiting de envio por cliente | TRANSCRICAO | [09:38] Diego; [09:39] Larissa |
| PRD-ESC-03 | docs/PRD.md | Fora de escopo | Dashboard visual para o cliente | TRANSCRICAO | [09:39] Marcos; [09:40] Larissa |
| PRD-ESC-04 | docs/PRD.md | Fora de escopo | Múltiplos workers em paralelo e ordering global | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| PRD-ESC-05 | docs/PRD.md | Fora de escopo | Arquivamento das linhas entregues da outbox | TRANSCRICAO | [09:08] Diego |
| PRD-ESC-06 | docs/PRD.md | Fora de escopo | Webhooks de entrada (cliente para nós) | TRANSCRICAO | [09:02] Marcos |
| PRD-ESC-07 | docs/PRD.md | Fora de escopo | Itens do pedido no payload | TRANSCRICAO | [09:43] Diego |
| PRD-ESC-08 | docs/PRD.md | Fora de escopo | Endurecer permissões do CRUD de configuração | TRANSCRICAO | [09:37] Sofia |
| PRD-DEC-01 | docs/PRD.md | Decisão | Outbox no MySQL na mesma transação (vs. síncrono e Redis) | TRANSCRICAO | [09:06] Diego; [09:07] Larissa; [09:08] Larissa |
| PRD-DEC-02 | docs/PRD.md | Decisão | Worker separado em polling de 2 s, worker único | TRANSCRICAO | [09:09] Diego; [09:10] Larissa; [09:11] Diego |
| PRD-DEC-03 | docs/PRD.md | Decisão | 5 tentativas com backoff e DLQ em tabela separada com replay | TRANSCRICAO | [09:15] Diego; [09:17] Larissa; [09:18] Diego |
| PRD-DEC-04 | docs/PRD.md | Decisão | HMAC-SHA256 com secret por endpoint e rotação de 24 h | TRANSCRICAO | [09:22] Sofia |
| PRD-DEC-05 | docs/PRD.md | Decisão | At-least-once com X-Event-Id | TRANSCRICAO | [09:26] Larissa |
| PRD-DEC-06 | docs/PRD.md | Decisão | Reuso máximo dos padrões existentes | TRANSCRICAO | [09:30] Larissa |
| PRD-DEC-07 | docs/PRD.md | Decisão | Publicação transacional com snapshot e filtro na inserção | TRANSCRICAO | [09:41] Bruno; [09:52] Larissa |
| PRD-DEP-01 | docs/PRD.md | Dependência | OrderService.changeStatus passa a chamar publishWebhookEvent dentro da transação | CODIGO | src/modules/orders/order.service.ts |
| PRD-DEP-02 | docs/PRD.md | Dependência | MySQL e Prisma: novas tabelas via migration, UUID como PK | CODIGO | prisma/schema.prisma |
| PRD-DEP-03 | docs/PRD.md | Dependência | Padrões reutilizados sem mudança: AppError, error middleware, requireRole | CODIGO | src/shared/errors/app-error.ts; src/middlewares/error.middleware.ts; src/middlewares/auth.middleware.ts |
| PRD-DEP-04 | docs/PRD.md | Dependência | Segundo processo (npm run worker) a implantar e monitorar; empacotamento indefinido (sem Dockerfile, docker-compose só com MySQL) | CODIGO | docker-compose.yml |
| PRD-DEP-05 | docs/PRD.md | Dependência | Revisão de segurança da Sofia, mínimo de 2 dias úteis ao fim | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-06 | docs/PRD.md | Dependência | Portal de desenvolvedores documentando integração, assinatura e deduplicação | TRANSCRICAO | [09:26] Marcos; [09:40] Marcos |
| PRD-DEP-07 | docs/PRD.md | Dependência | Marcos confirma o prazo com a Atlas e atualiza os clientes | TRANSCRICAO | [09:47] Marcos; [09:49] Marcos |
| PRD-DEP-08 | docs/PRD.md | Dependência | Clientes precisam verificar assinatura e deduplicar por X-Event-Id | TRANSCRICAO | [09:25] Diego; [09:26] Marcos |
| PRD-RISCO-01 | docs/PRD.md | Risco | Prazo de 3 sprints frente ao fim de novembro | TRANSCRICAO | [09:45] Marcos; [09:46] Larissa; [09:46] Sofia |
| PRD-RISCO-02 | docs/PRD.md | Risco | Falha na outbox bloqueia a mudança de status | TRANSCRICAO | [09:40] Bruno; [09:46] Larissa |
| PRD-RISCO-03 | docs/PRD.md | Risco | Vazamento de secret de cliente | TRANSCRICAO | [09:22] Diego; [09:21] Sofia |
| PRD-RISCO-04 | docs/PRD.md | Risco | Cliente não deduplica | TRANSCRICAO | [09:25] Diego; [09:26] Marcos |
| PRD-RISCO-05 | docs/PRD.md | Risco | Cliente lento atrasa os demais (envio sequencial, até 10 s por chamada) | PROPOSTA | Relacionada a [09:12] Diego; [09:42] Diego |
| PRD-RISCO-06 | docs/PRD.md | Risco | Crescimento da outbox e do histórico sem arquivamento | TRANSCRICAO | [09:08] Diego |
| PRD-RISCO-07 | docs/PRD.md | Risco | Worker parado sem ninguém perceber | PROPOSTA | Sem origem direta na reunião |
| PRD-RISCO-08 | docs/PRD.md | Risco | SSRF: URL cadastrada pode apontar para rede interna | PROPOSTA | Não discutido na reunião |
| PRD-RISCO-09 | docs/PRD.md | Risco | Perda de ordering com worker escalado ou evento em retry | TRANSCRICAO | [09:13] Larissa |
| PRD-RISCO-10 | docs/PRD.md | Risco | Permissões amplas no CRUD de configuração | TRANSCRICAO | [09:37] Sofia |
| PRD-AC-01 | docs/PRD.md | Critério de Aceite | Evento chega em menos de 10 s com headers e payload previstos | TRANSCRICAO | [09:02] Marcos; [09:44] Diego |
| PRD-AC-02 | docs/PRD.md | Critério de Aceite | Falha na outbox mantém status, histórico e estoque como antes | TRANSCRICAO | [09:40] Bruno |
| PRD-AC-03 | docs/PRD.md | Critério de Aceite | Sem webhook interessado no status, nada é inserido na outbox | TRANSCRICAO | [09:34] Bruno |
| PRD-AC-04 | docs/PRD.md | Critério de Aceite | Payload gravado não muda se o pedido for alterado depois | TRANSCRICAO | [09:52] Larissa |
| PRD-AC-05 | docs/PRD.md | Critério de Aceite | URL http recusada com WEBHOOK_INVALID_URL | TRANSCRICAO | [09:23] Sofia; [09:28] Bruno |
| PRD-AC-06 | docs/PRD.md | Critério de Aceite | X-Signature verificável; rotação com 24 h de convivência | TRANSCRICAO | [09:22] Sofia |
| PRD-AC-07 | docs/PRD.md | Critério de Aceite | Timeout de 10 s e reagendamento 1m/5m/30m/2h/12h | TRANSCRICAO | [09:42] Diego; [09:17] Larissa |
| PRD-AC-08 | docs/PRD.md | Critério de Aceite | Esgotadas as tentativas, evento vai à DLQ com payload, motivo e timestamp | TRANSCRICAO | [09:18] Diego |
| PRD-AC-09 | docs/PRD.md | Critério de Aceite | Replay: 403 para não ADMIN; ADMIN reenfileira e é logado | TRANSCRICAO | [09:36] Sofia; [09:36] Larissa |
| PRD-AC-10 | docs/PRD.md | Critério de Aceite | Worker por npm run worker, separado da API; reenvio após restart com mesmo X-Event-Id | TRANSCRICAO | [09:11] Diego; [09:25] Diego |
| PRD-AC-11 | docs/PRD.md | Critério de Aceite | Corpo acima de 64 KB vai à DLQ com erro específico | TRANSCRICAO | [09:24] Larissa |
| PRD-AC-12 | docs/PRD.md | Critério de Aceite | CRUD de webhooks; secret só na criação e na rotação | TRANSCRICAO | [09:31] Marcos; [09:33] Bruno |
| PRD-AC-13 | docs/PRD.md | Critério de Aceite | Histórico devolve no máximo 100 entregas, mais recente primeiro | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-14 | docs/PRD.md | Critério de Aceite | Eventos do mesmo pedido chegam em ordem quando não há falhas | TRANSCRICAO | [09:12] Diego |
| PRD-AC-15 | docs/PRD.md | Critério de Aceite | Testes existentes de pedidos e autenticação passam sem mudança de comportamento | CODIGO | tests/orders.test.ts; tests/auth.test.ts |
| PRD-AC-16 | docs/PRD.md | Critério de Aceite | Revisão de segurança concluída antes do deploy | TRANSCRICAO | [09:46] Sofia; [09:49] Sofia |
| PRD-AC-17 | docs/PRD.md | Critério de Aceite | Portal de desenvolvedores documenta integração, assinatura e deduplicação | TRANSCRICAO | [09:26] Marcos |
| PRD-TEST-01 | docs/PRD.md | Estratégia de teste | Testes ponta a ponta da integração em order.service.ts | TRANSCRICAO | [09:46] Larissa |
| PRD-TEST-02 | docs/PRD.md | Estratégia de teste | Níveis unitário, integração e regressão com Vitest e Supertest (Proposta alinhada às ferramentas do repositório) | CODIGO | tests/orders.test.ts; vitest.config.ts |
| PRD-TEST-03 | docs/PRD.md | Estratégia de teste | Revisão de segurança manual (HMAC, geração de secret) antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-CTX-01 | docs/PRD.md | Contexto | Pedido formal de Atlas, MaxDistribuição e Nova Cargo; Atlas pode migrar para concorrente sem entrega até o fim do trimestre | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-02 | docs/PRD.md | Contexto | Cenário: cliente com indisponibilidade de 2 h em manutenção planejada | TRANSCRICAO | [09:16] Diego |
| PRD-CTX-03 | docs/PRD.md | Contexto | Cenário: cliente que vazou secret em log de aplicação | TRANSCRICAO | [09:22] Diego |
| PRD-CTX-04 | docs/PRD.md | Restrição | Os usuários que representam o cliente se autenticam pela API com JWT do nosso sistema | TRANSCRICAO | [09:32] Marcos |
| PRD-CTX-05 | docs/PRD.md | Restrição | O OMS não tem hoje notificação externa, evento, fila ou webhook | CODIGO | src/app.ts |
| PRD-CTX-06 | docs/PRD.md | Restrição | Papel ADMIN exigido no replay via requireRole existente | CODIGO | src/middlewares/auth.middleware.ts |
| PRD-OPEN-01 | docs/PRD.md | Questão em aberto | Criação de pedido gera evento? (RFC Q8) | CODIGO | src/modules/orders/order.service.ts |
| PRD-OPEN-02 | docs/PRD.md | Questão em aberto | Retry × ordering (RFC Q7) | PROPOSTA | Relacionada a [09:12] Diego |
| PRD-OPEN-03 | docs/PRD.md | Questão em aberto | Contagem das 5 tentativas (RFC Q9) | PROPOSTA | Relacionada a [09:17] Diego |
| PRD-OPEN-04 | docs/PRD.md | Questão em aberto | Meta de taxa de sucesso e alertas não definida | PROPOSTA | Sem origem na reunião |
| PRD-OPEN-05 | docs/PRD.md | Fora de escopo / adiado | Rate limiting e e-mail reavaliados após medir o uso | TRANSCRICAO | [09:37] Larissa; [09:39] Larissa |
| PRD-PUB-01 | docs/PRD.md | Público-alvo | Público: clientes B2B (Atlas, MaxDistribuição, Nova Cargo) | TRANSCRICAO | [09:00] Marcos |
| PRD-PUB-02 | docs/PRD.md | Público-alvo | Público: usuários do OMS que representam o cliente, autenticados por JWT | TRANSCRICAO | [09:32] Marcos |
| PRD-PUB-03 | docs/PRD.md | Público-alvo | Público: administradores (role ADMIN) que reprocessam a DLQ | TRANSCRICAO | [09:36] Sofia |
| PRD-PUB-04 | docs/PRD.md | Público-alvo | Público: engenharia e Segurança que implementam, operam e revisam | TRANSCRICAO | [09:46] Larissa; [09:46] Sofia |
| PRD-PUB-05 | docs/PRD.md | Público-alvo | Público: Product Manager que documenta no portal de desenvolvedores | TRANSCRICAO | [09:26] Marcos |
| PRD-CEN-01 | docs/PRD.md | Cenário de uso | Cenário: acompanhar entregas sem consultar GET /orders | TRANSCRICAO | [09:00] Marcos |
| PRD-CEN-02 | docs/PRD.md | Cenário de uso | Cenário: receber só SHIPPED e DELIVERED | TRANSCRICAO | [09:33] Marcos |
| PRD-CEN-03 | docs/PRD.md | Cenário de uso | Cenário: cliente indisponível por manutenção de 2 h recebe depois | TRANSCRICAO | [09:16] Diego |
| PRD-CEN-04 | docs/PRD.md | Cenário de uso | Cenário: rotação de secret com 24 h para migrar | TRANSCRICAO | [09:21] Sofia; [09:22] Diego |
| PRD-CEN-05 | docs/PRD.md | Cenário de uso | Cenário: cliente consulta os últimos 100 envios | TRANSCRICAO | [09:34] Marcos |
| PRD-CEN-06 | docs/PRD.md | Cenário de uso | Cenário: administrador reprocessa item da DLQ com auditoria | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada | Disparo síncrono em changeStatus | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada | Fila externa (Redis Streams ou similar) | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa descartada | Trigger do MySQL para acionar o worker | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | docs/RFC.md | Alternativa descartada | Worker dentro da API | TRANSCRICAO | [09:11] Diego |
| RFC-ALT-05 | docs/RFC.md | Alternativa descartada | Retry indefinido com backoff | TRANSCRICAO | [09:15] Diego |
| RFC-ALT-06 | docs/RFC.md | Alternativa descartada | Apenas 3 tentativas | TRANSCRICAO | [09:16] Diego |
| RFC-ALT-07 | docs/RFC.md | Alternativa descartada | Secret global da plataforma | TRANSCRICAO | [09:21] Sofia |
| RFC-ALT-08 | docs/RFC.md | Alternativa descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| RFC-ALT-09 | docs/RFC.md | Alternativa descartada | Guardar só order_id e renderizar no envio | TRANSCRICAO | [09:52] Larissa |
| RFC-CTX-01 | docs/RFC.md | Contexto | Três clientes B2B pedem notificação em tempo real; hoje consultam GET /orders periodicamente | TRANSCRICAO | [09:00] Marcos |
| RFC-CTX-02 | docs/RFC.md | Restrição | Tempo real significa menos de 10 s; fluxo apenas de saída | TRANSCRICAO | [09:02] Marcos |
| RFC-CTX-03 | docs/RFC.md | Restrição | Cliente lento ou fora do ar não pode travar nem reverter a mudança de status | TRANSCRICAO | [09:04] Bruno |
| RFC-CTX-04 | docs/RFC.md | Restrição | Se o status mudou o evento tem que existir; se foi revertido o evento não sai | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| RFC-CTX-05 | docs/RFC.md | Restrição | OrderService.changeStatus atualiza orders, order_status_history e estoque numa transação; é o ponto de origem do evento | CODIGO | src/modules/orders/order.service.ts |
| RFC-GAR-01 | docs/RFC.md | Restrição | Ordering só por order_id e com worker único; sem ordering global | TRANSCRICAO | [09:13] Larissa; [09:14] Marcos |
| RFC-GAR-02 | docs/RFC.md | Restrição | Falha permanente após ~15 h de tentativas vai à DLQ e depende de replay manual | TRANSCRICAO | [09:17] Marcos |
| RFC-Q-01 | docs/RFC.md | Fora de escopo / adiado | Rate limiting de envio por cliente: observar e decidir depois | TRANSCRICAO | [09:38] Diego; [09:39] Larissa |
| RFC-Q-02 | docs/RFC.md | Fora de escopo / adiado | Aviso por e-mail ao cliente: adiado para próxima fase | TRANSCRICAO | [09:37] Larissa |
| RFC-Q-03 | docs/RFC.md | Fora de escopo / adiado | Escala para múltiplos workers (particionar por order_id ou lock pessimista): adiada | TRANSCRICAO | [09:13] Diego |
| RFC-Q-04 | docs/RFC.md | Fora de escopo / adiado | Arquivamento das linhas entregues da outbox (~30 dias): fora do escopo | TRANSCRICAO | [09:08] Diego |
| RFC-Q-05 | docs/RFC.md | Fora de escopo / adiado | Endurecer permissões do CRUD de configuração: mantido por enquanto | TRANSCRICAO | [09:37] Sofia |
| RFC-Q-06 | docs/RFC.md | Fora de escopo / adiado | Painel visual para o cliente: descartado nesta fase | TRANSCRICAO | [09:40] Larissa |
| RFC-Q-07 | docs/RFC.md | Questão em aberto | Q7 Retry × ordering: evento em backoff é ultrapassado por eventos posteriores do mesmo pedido? | PROPOSTA | Lacuna da consolidação; relacionada a [09:12] Diego |
| RFC-Q-08 | docs/RFC.md | Questão em aberto | Q8 Criação de pedido gera evento? | CODIGO | src/modules/orders/order.service.ts |
| RFC-Q-09 | docs/RFC.md | Questão em aberto | Q9 Contagem das 5 tentativas (1 envio + 5 retries = 14h36 vs. quase 15 h) | PROPOSTA | Lacuna da consolidação; relacionada a [09:17] Diego |
| RFC-Q-10 | docs/RFC.md | Questão em aberto | Q10 X-Timestamp entra na assinatura? | PROPOSTA | Lacuna da consolidação; relacionada a [09:44] Diego |
| RFC-Q-11 | docs/RFC.md | Questão em aberto | Q11 Armazenamento da secret e redact em logs | CODIGO | src/shared/logger/index.ts |
| RFC-Q-12 | docs/RFC.md | Questão em aberto | Q12 Bloqueio em cabeça de fila (envio sequencial, até 10 s por chamada) | PROPOSTA | Lacuna da consolidação; relacionada a [09:42] Diego |
| RFC-Q-13 | docs/RFC.md | Questão em aberto | Q13 Formato de X-Signature com duas assinaturas durante a rotação | PROPOSTA | Lacuna da consolidação; relacionada a [09:21] Sofia |
| RFC-Q-14 | docs/RFC.md | Questão em aberto | Q14 Exportação de métricas e tracing (Prometheus ou OpenTelemetry) | CODIGO | package.json |
| RFC-RISCO-01 | docs/RFC.md | Risco | Falha na outbox bloqueia a mudança de status | TRANSCRICAO | [09:40] Bruno; [09:46] Larissa |
| RFC-RISCO-02 | docs/RFC.md | Risco | Crescimento da outbox sem arquivamento | TRANSCRICAO | [09:08] Diego |
| RFC-RISCO-03 | docs/RFC.md | Risco | Cliente não deduplica e processa o evento duas vezes | TRANSCRICAO | [09:26] Marcos |
| RFC-RISCO-04 | docs/RFC.md | Risco | Vazamento de secret de cliente (já ocorreu antes) | TRANSCRICAO | [09:22] Diego |
| RFC-RISCO-05 | docs/RFC.md | Risco | Prazo de 3 sprints frente ao fim de novembro pedido pela Atlas | TRANSCRICAO | [09:45] Marcos; [09:46] Larissa |
| RFC-RISCO-06 | docs/RFC.md | Risco | Perda de ordering se o worker for escalado | TRANSCRICAO | [09:13] Larissa |
| RFC-IMP-01 | docs/RFC.md | Restrição | Novas tabelas por migration Prisma, com UUID como PK | TRANSCRICAO | [09:51] Larissa |
| RFC-IMP-02 | docs/RFC.md | Restrição | Fiação em src/app.ts e src/routes/index.ts; scripts em package.json; ajuste de redact no logger | CODIGO | src/app.ts; src/routes/index.ts; package.json; src/shared/logger/index.ts |
| RFC-SUP-01 | docs/RFC.md | Requisito Funcional | Superfície do cliente: CRUD de webhooks, rotação de secret, histórico de entregas e replay ADMIN com registro de quem executou | TRANSCRICAO | [09:31] Marcos; [09:33] Bruno; [09:34] Marcos; [09:36] Sofia |
| RFC-SUP-02 | docs/RFC.md | Restrição | customer_id vem no body ou no path, não do JWT | TRANSCRICAO | [09:32] Larissa |
| RFC-PRX-01 | docs/RFC.md | Decisão | Próximos passos: sessão de revisão com Bruno e Diego antes de codar; agendar revisão de segurança com Sofia | TRANSCRICAO | [09:50] Larissa; [09:49] Sofia |
| RFC-COMP-01 | docs/RFC.md | Decisão | Publicação do evento: função chamada dentro da transação de changeStatus, com snapshot e filtro | TRANSCRICAO | [09:41] Bruno; [09:52] Larissa |
| RFC-COMP-02 | docs/RFC.md | Decisão | Armazenamento: outbox no MySQL existente, atômica com a mudança de status | TRANSCRICAO | [09:06] Diego; [09:08] Larissa |
| RFC-COMP-03 | docs/RFC.md | Decisão | Entrega: worker em processo próprio, polling de 2 s, worker único | TRANSCRICAO | [09:09] Diego; [09:11] Diego; [09:12] Diego |
| RFC-COMP-04 | docs/RFC.md | Decisão | Resiliência: 5 tentativas com backoff, timeout de 10 s, DLQ separada e replay administrativo | TRANSCRICAO | [09:17] Larissa; [09:42] Diego; [09:18] Diego |
| RFC-COMP-05 | docs/RFC.md | Decisão | Segurança: HMAC-SHA256 sobre o corpo, secret por endpoint, rotação de 24 h | TRANSCRICAO | [09:22] Sofia |
| RFC-COMP-06 | docs/RFC.md | Decisão | Semântica de entrega: at-least-once, idempotência do cliente via X-Event-Id | TRANSCRICAO | [09:26] Larissa |
| RFC-COMP-07 | docs/RFC.md | Decisão | Integração com o código: módulo src/modules/webhooks (a criar), reuso de AppError, Pino, error middleware e requireRole, prefixo WEBHOOK_ | TRANSCRICAO | [09:30] Larissa; [09:36] Larissa |
| FDD-OT-01 | docs/FDD.md | Objetivo técnico | Registrar cada mudança de status relevante na outbox, na mesma transação | TRANSCRICAO | [09:40] Bruno |
| FDD-OT-02 | docs/FDD.md | Objetivo técnico | Entregar o evento em menos de 10 s após o commit | TRANSCRICAO | [09:02] Marcos; [09:10] Larissa |
| FDD-OT-03 | docs/FDD.md | Objetivo técnico | Entrega em processo separado da API (src/worker.ts (a criar), npm run worker) | TRANSCRICAO | [09:11] Diego; [09:11] Larissa |
| FDD-OT-04 | docs/FDD.md | Objetivo técnico | Backoff 1m/5m/30m/2h/12h e DLQ ao esgotar | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| FDD-OT-05 | docs/FDD.md | Objetivo técnico | HMAC-SHA256, secret única por endpoint, rotação com 24 h | TRANSCRICAO | [09:22] Sofia |
| FDD-OT-06 | docs/FDD.md | Objetivo técnico | At-least-once com deduplicação via X-Event-Id | TRANSCRICAO | [09:26] Larissa |
| FDD-OT-07 | docs/FDD.md | Objetivo técnico | CRUD, rotação, histórico e replay de DLQ | TRANSCRICAO | [09:31] Marcos; [09:33] Bruno; [09:34] Marcos; [09:35] Diego |
| FDD-OT-08 | docs/FDD.md | Objetivo técnico | Reuso dos padrões do projeto | TRANSCRICAO | [09:30] Larissa |
| FDD-OT-09 | docs/FDD.md | Objetivo técnico | Sem dependências npm nem infraestrutura novas (confirmação pendente no review) | TRANSCRICAO | [09:07] Diego; [09:29] Bruno |
| FDD-ESC-01 | docs/FDD.md | Fora de escopo | E-mail ao cliente quando o webhook falha: próxima fase | TRANSCRICAO | [09:37] Larissa |
| FDD-ESC-02 | docs/FDD.md | Fora de escopo | Rate limiting de envio: observar e decidir depois | TRANSCRICAO | [09:39] Larissa |
| FDD-ESC-03 | docs/FDD.md | Fora de escopo | Dashboard visual: projeto do frontend | TRANSCRICAO | [09:40] Larissa |
| FDD-ESC-04 | docs/FDD.md | Fora de escopo | Arquivamento de linhas entregues: fora da feature | TRANSCRICAO | [09:08] Diego |
| FDD-ESC-05 | docs/FDD.md | Fora de escopo | Múltiplos workers e ordering global: adiado | TRANSCRICAO | [09:13] Larissa |
| FDD-ESC-06 | docs/FDD.md | Fora de escopo | Webhooks de entrada: não solicitado | TRANSCRICAO | [09:02] Marcos |
| FDD-ESC-07 | docs/FDD.md | Fora de escopo | Itens do pedido no payload: excluídos | TRANSCRICAO | [09:43] Diego |
| FDD-ESC-08 | docs/FDD.md | Fora de escopo | Endurecer permissões do CRUD: por enquanto qualquer role | TRANSCRICAO | [09:37] Sofia |
| FDD-ESC-09 | docs/FDD.md | Fora de escopo | Listagem da DLQ via API, evento de criação de pedido e alertas automáticos ficam fora da especificação | PROPOSTA | Não discutido na reunião |
| FDD-DADOS-01 | docs/FDD.md | Decisão de modelagem | Quatro status do evento: pendente, processando, falhou, entregue | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-02 | docs/FDD.md | Decisão de modelagem | Índices em status e created_at na outbox | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-03 | docs/FDD.md | Decisão de modelagem | DLQ em tabela separada com payload, motivo e timestamp | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-04 | docs/FDD.md | Decisão de modelagem | Todas as PKs UUID | TRANSCRICAO | [09:51] Larissa |
| FDD-DADOS-05 | docs/FDD.md | Decisão de modelagem | Configuração guarda URL, secret, customer e estado ativo | TRANSCRICAO | [09:21] Bruno |
| FDD-DADOS-06 | docs/FDD.md | Decisão de modelagem | Uma linha de outbox por endpoint de destino, com id igual ao event_id | PROPOSTA | Deriva de [09:44] Sofia (X-Webhook-Id) |
| FDD-DADOS-07 | docs/FDD.md | Decisão de modelagem | Sem FK de orderId para orders, pois OrderService.delete remove pedidos PENDING ou CANCELLED | CODIGO | src/modules/orders/order.service.ts |
| FDD-DADOS-08 | docs/FDD.md | Decisão de modelagem | onDelete Cascade do endpoint para tabelas filhas, como em OrderItem e OrderStatusHistory | CODIGO | prisma/schema.prisma |
| FDD-DADOS-09 | docs/FDD.md | Decisão de modelagem | events como Json (lista de OrderStatus), no estilo de Customer.address | CODIGO | prisma/schema.prisma |
| FDD-DADOS-10 | docs/FDD.md | Decisão de modelagem | secret, previousSecret e previousSecretExpiresAt na linha do endpoint | PROPOSTA | Viabiliza [09:21] Sofia |
| FDD-FLUXO-01 | docs/FDD.md | Fluxo | Criação do evento: publishWebhookEvent(tx, ...) dentro do $transaction; filtra endpoints; createMany com snapshot; erro vira rollback | TRANSCRICAO | [09:34] Bruno; [09:40] Bruno; [09:41] Bruno; [09:52] Larissa |
| FDD-FLUXO-02 | docs/FDD.md | Fluxo | Processamento do worker: polling de 2 s em ordem de created_at, timeout de 10 s, PrismaClient próprio | TRANSCRICAO | [09:09] Diego; [09:11] Diego; [09:30] Bruno; [09:42] Diego |
| FDD-FLUXO-03 | docs/FDD.md | Fluxo | Retry: tabela fixa 1m/5m/30m/2h/12h; 5 tentativas | TRANSCRICAO | [09:17] Larissa; [09:48] Larissa |
| FDD-FLUXO-04 | docs/FDD.md | Fluxo | DLQ e replay: move para webhook_dead_letter; replay ADMIN recoloca na outbox como pendente e loga o executor | TRANSCRICAO | [09:18] Diego; [09:36] Larissa; [09:36] Sofia |
| FDD-FLUXO-05 | docs/FDD.md | Fluxo | Rotação de secret: antiga válida por 24 h | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-01-COD | docs/FDD.md | Restrição | Ponto de inserção: após tx.order.update e tx.orderStatusHistory.create, antes da releitura refreshed | CODIGO | src/modules/orders/order.service.ts |
| FDD-FLUXO-02-COD | docs/FDD.md | Restrição | src/worker.ts (a criar) espelha o bootstrap e o shutdown de src/server.ts | CODIGO | src/server.ts |
| FDD-FLUXO-03-Q9 | docs/FDD.md | Proposta | Leitura adotada: 1 envio inicial + 5 retries, até 6 chamadas, 14h36 de atraso total | PROPOSTA | Relacionada a [09:17] Diego |
| FDD-FLUXO-03-Q7 | docs/FDD.md | Proposta | Evento em backoff não bloqueia os seguintes do mesmo pedido | PROPOSTA | Relacionada a [09:12] Diego |
| FDD-FLUXO-03-JIT | docs/FDD.md | Proposta | Sem jitter: progressão exata da reunião | TRANSCRICAO | [09:17] Larissa |
| FDD-FLUXO-05-GEN | docs/FDD.md | Proposta | Geração da secret com crypto.randomBytes(32), sem dependência nova | PROPOSTA | Sem origem direta; relacionada a [09:21] Sofia |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | POST /webhooks: cadastrar, https obrigatório, secret devolvida na criação | TRANSCRICAO | [09:31] Marcos; [09:23] Sofia |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato | GET /webhooks?customerId=: listar webhooks de um customer | TRANSCRICAO | [09:33] Bruno; [09:32] Larissa |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | PATCH /webhooks/:id: editar | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04 | docs/FDD.md | Contrato | DELETE /webhooks/:id: remover (204) | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-05 | docs/FDD.md | Contrato | POST /webhooks/:id/secret/rotate: rotacionar pela API (nome da rota é Proposta) | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | GET /webhooks/:id/deliveries: últimos 100 envios | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | POST /admin/webhooks/dead-letter/:id/replay: replay ADMIN (202 é Proposta) | TRANSCRICAO | [09:18] Diego; [09:35] Diego; [09:36] Sofia |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | Requisição de saída: headers X-Event-Id, X-Signature, X-Timestamp, X-Webhook-Id; payload snake_case sem itens | TRANSCRICAO | [09:43] Diego; [09:44] Diego; [09:44] Sofia; [09:45] Diego |
| FDD-CONV-01 | docs/FDD.md | Restrição | Router montado em /api/v1; erros no formato { error: { code, message, details } } | CODIGO | src/app.ts; src/middlewares/error.middleware.ts |
| FDD-CONV-02 | docs/FDD.md | Restrição | Secret nunca aparece em listagens, PATCH ou histórico; só na criação e na rotação | TRANSCRICAO | [09:31] Marcos |
| FDD-CONV-03 | docs/FDD.md | Restrição | Validação Zod no padrão de customer.schemas.ts; events com z.nativeEnum(OrderStatus) | CODIGO | src/modules/customers/customer.schemas.ts |
| FDD-ERRO-01 | docs/FDD.md | Erro previsto | WEBHOOK_NOT_FOUND (404) | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-02 | docs/FDD.md | Erro previsto | WEBHOOK_INVALID_URL (400): URL não https | TRANSCRICAO | [09:28] Bruno; [09:23] Sofia |
| FDD-ERRO-03 | docs/FDD.md | Erro previsto | WEBHOOK_DEAD_LETTER_NOT_FOUND (404) | PROPOSTA | Sem origem direta na reunião |
| FDD-ERRO-04 | docs/FDD.md | Erro previsto | WEBHOOK_DEAD_LETTER_ALREADY_REPLAYED (409) | PROPOSTA | Sem origem direta na reunião |
| FDD-ERRO-05 | docs/FDD.md | Erro previsto | WEBHOOK_EVENT_PUBLISH_FAILED (500): falha na outbox reverte a transação | TRANSCRICAO | [09:40] Bruno |
| FDD-ERRO-06 | docs/FDD.md | Erro previsto | WEBHOOK_DELIVERY_TIMEOUT: sem resposta em 10 s | TRANSCRICAO | [09:42] Diego |
| FDD-ERRO-07 | docs/FDD.md | Erro previsto | WEBHOOK_DELIVERY_HTTP_ERROR: resposta não 2xx | PROPOSTA | Sem origem direta na reunião |
| FDD-ERRO-08 | docs/FDD.md | Erro previsto | WEBHOOK_DELIVERY_NETWORK_ERROR: DNS, conexão ou TLS | PROPOSTA | Sem origem direta na reunião |
| FDD-ERRO-09 | docs/FDD.md | Erro previsto | WEBHOOK_PAYLOAD_TOO_LARGE: corpo acima de 64 KB, vai direto à DLQ | TRANSCRICAO | [09:24] Diego; [09:24] Larissa |
| FDD-ERRO-10 | docs/FDD.md | Erro previsto | WEBHOOK_ENDPOINT_INACTIVE: endpoint desativado no envio, vai à DLQ | PROPOSTA | Sem origem direta na reunião |
| FDD-ERRO-11 | docs/FDD.md | Erro previsto | WEBHOOK_SECRET_REQUIRED: endpoint sem secret utilizável | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-COD | docs/FDD.md | Restrição | WebhookNotFoundError estende AppError diretamente, pois NotFoundError fixa NOT_FOUND | CODIGO | src/shared/errors/http-errors.ts |
| FDD-RES-01 | docs/FDD.md | Resiliência | Timeout HTTP de 10 s com AbortController | TRANSCRICAO | [09:42] Diego |
| FDD-RES-02 | docs/FDD.md | Resiliência | Retry com backoff 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Larissa |
| FDD-RES-03 | docs/FDD.md | Resiliência | DLQ com payload, motivo e timestamp | TRANSCRICAO | [09:18] Diego |
| FDD-RES-04 | docs/FDD.md | Resiliência | Replay manual ADMIN | TRANSCRICAO | [09:18] Diego; [09:36] Larissa |
| FDD-RES-05 | docs/FDD.md | Resiliência | Isolamento da transação de pedidos: sem HTTP na transação | TRANSCRICAO | [09:04] Bruno |
| FDD-RES-06 | docs/FDD.md | Resiliência | Atomicidade evento + status | TRANSCRICAO | [09:40] Bruno |
| FDD-RES-07 | docs/FDD.md | Resiliência | At-least-once: reenvio em qualquer dúvida | TRANSCRICAO | [09:26] Larissa |
| FDD-RES-08 | docs/FDD.md | Resiliência | Processo independente da API | TRANSCRICAO | [09:11] Diego |
| FDD-RES-09 | docs/FDD.md | Resiliência | Recuperação de eventos em voo no start (PROCESSING volta a PENDING) | PROPOSTA | Relacionada a [09:25] Diego (at-least-once) |
| FDD-RES-10 | docs/FDD.md | Resiliência | Reivindicação com guarda (updateMany com checagem de count) | PROPOSTA | Sem origem direta na reunião |
| FDD-RES-11 | docs/FDD.md | Resiliência | Rodadas sem sobreposição (setTimeout encadeado) | PROPOSTA | Relacionada a [09:09] Diego |
| FDD-RES-12 | docs/FDD.md | Resiliência | Redirects não seguidos; 3xx conta como falha | PROPOSTA | Sem origem direta na reunião |
| FDD-RES-13 | docs/FDD.md | Resiliência | Resposta do cliente limitada a ~2 KB | PROPOSTA | Sem origem direta na reunião |
| FDD-RES-14 | docs/FDD.md | Resiliência | Tolerância a indisponibilidade do banco no laço | PROPOSTA | Sem origem direta na reunião |
| FDD-RES-15 | docs/FDD.md | Resiliência | Shutdown gracioso com SIGINT e SIGTERM | CODIGO | src/server.ts |
| FDD-RES-LIM | docs/FDD.md | Restrição | Limites conhecidos: ordering só por order_id com worker único; sem rate limiting de saída | TRANSCRICAO | [09:13] Larissa; [09:39] Larissa |
| FDD-OBS-01 | docs/FDD.md | Observabilidade | Logs em Pino com eventos webhook_*; replay loga userId para auditoria; nunca logar secret, assinatura ou payload completo | TRANSCRICAO | [09:29] Bruno; [09:36] Sofia; [09:22] Diego |
| FDD-OBS-01-COD | docs/FDD.md | Restrição | redactPaths não cobre secret; acrescentar *.secret e *.previousSecret | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-02 | docs/FDD.md | Observabilidade | Métricas por SQL e logs (backlog, idade do mais antigo, DLQ aberta), sem prom-client; alerta acima de 30 s | PROPOSTA | Relacionada a [09:02] Marcos (requisito de 10 s) |
| FDD-OBS-03 | docs/FDD.md | Observabilidade | Tracing por correlação de eventId e webhookId; X-Request-Id não chega ao changeStatus | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-INT-01 | docs/FDD.md | Integração com o código | Único ponto crítico: chamar publishWebhookEvent em changeStatus dentro do $transaction | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Integração com o código | Instanciar repository, service e controller de webhooks em buildControllers | CODIGO | src/app.ts |
| FDD-INT-03 | docs/FDD.md | Integração com o código | Montar /webhooks e /admin/webhooks em buildApiRouter e no tipo Controllers | CODIGO | src/routes/index.ts |
| FDD-INT-04 | docs/FDD.md | Integração com o código | Reuso de authenticate e requireRole('ADMIN') sem alteração | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-INT-05 | docs/FDD.md | Integração com o código | Reuso de validate com schemas Zod no padrão de order.schemas.ts | CODIGO | src/middlewares/validate.middleware.ts |
| FDD-INT-06 | docs/FDD.md | Integração com o código | Classes de erro estendem AppError, BadRequestError e ConflictError sem alterar os arquivos | CODIGO | src/shared/errors/app-error.ts; src/shared/errors/http-errors.ts; src/shared/errors/index.ts |
| FDD-INT-07 | docs/FDD.md | Integração com o código | errorMiddleware já serializa AppError, cobrindo os códigos WEBHOOK_* | CODIGO | src/middlewares/error.middleware.ts |
| FDD-INT-08 | docs/FDD.md | Integração com o código | Logger compartilhado; ajuste único em redactPaths | CODIGO | src/shared/logger/index.ts |
| FDD-INT-09 | docs/FDD.md | Integração com o código | Variáveis de ambiente novas em env.ts; prisma client próprio no processo do worker; env.ts exige JWT_SECRET | CODIGO | src/config/env.ts; src/config/database.ts |
| FDD-INT-10 | docs/FDD.md | Integração com o código | src/worker.ts (a criar) segue o modelo de bootstrap e shutdown de server.ts | CODIGO | src/server.ts |
| FDD-INT-11 | docs/FDD.md | Integração com o código | Novos modelos e migration aditiva; Customer ganha webhookEndpoints | CODIGO | prisma/schema.prisma |
| FDD-INT-12 | docs/FDD.md | Integração com o código | Scripts worker e start:worker em package.json; tsconfig.build.json compila src/worker.ts (a criar) | CODIGO | package.json; tsconfig.build.json |
| FDD-INT-13 | docs/FDD.md | Integração com o código | Documentar as variáveis novas em .env.example | CODIGO | .env.example |
| FDD-INT-14 | docs/FDD.md | Integração com o código | paginated() para GET /webhooks | CODIGO | src/shared/http/response.ts |
| FDD-INT-15 | docs/FDD.md | Integração com o código | tests/setup.ts limpa tabelas de webhook antes de customer; helper createTestWebhook | CODIGO | tests/setup.ts; tests/helpers/factories.ts |
| FDD-INT-16 | docs/FDD.md | Integração com o código | OrderStatus define o domínio de events; canTransition decide transições válidas | CODIGO | src/modules/orders/order.status.ts |
| FDD-VAR-01 | docs/FDD.md | Configuração | OUTBOX_POLL_INTERVAL_MS = 2000 | TRANSCRICAO | [09:09] Diego |
| FDD-VAR-02 | docs/FDD.md | Configuração | DELIVERY_TIMEOUT_MS = 10000 | TRANSCRICAO | [09:42] Diego |
| FDD-VAR-03 | docs/FDD.md | Configuração | DELIVERY_MAX_PAYLOAD_BYTES = 65536 (64 x 1024) | TRANSCRICAO | [09:24] Diego |
| FDD-VAR-04 | docs/FDD.md | Configuração | OUTBOX_BATCH_SIZE = 10 (reunião disse só batch pequeno) | PROPOSTA | Relacionada a [09:08] Diego |
| FDD-DEP-01 | docs/FDD.md | Dependência | Node.js >= 20: fetch, AbortController e node:crypto nativos | CODIGO | package.json |
| FDD-DEP-02 | docs/FDD.md | Dependência | Prisma 5.22.0 e @prisma/client: modelos, Json, createMany, transações interativas | CODIGO | package.json |
| FDD-DEP-03 | docs/FDD.md | Dependência | MySQL 8.0 via docker-compose | CODIGO | docker-compose.yml |
| FDD-DEP-04 | docs/FDD.md | Dependência | uuid 11.0.3 para gerar o event_id | CODIGO | package.json |
| FDD-DEP-05 | docs/FDD.md | Dependência | zod, pino, express e vitest/supertest já presentes | CODIGO | package.json |
| FDD-DEP-06 | docs/FDD.md | Restrição | Sem dependências npm novas e sem infraestrutura nova | TRANSCRICAO | [09:07] Diego; [09:29] Bruno |
| FDD-COMPAT-01 | docs/FDD.md | Restrição | Migration aditiva; sem FK para orders preserva o DELETE /orders/:id existente | CODIGO | src/modules/orders/order.service.ts |
| FDD-AC-01 | docs/FDD.md | Critério de Aceite | Status e outbox commitam juntos; falha na outbox mantém o estado anterior | TRANSCRICAO | [09:40] Bruno |
| FDD-AC-02 | docs/FDD.md | Critério de Aceite | Sem endpoint interessado nada é inserido; N endpoints geram N linhas | TRANSCRICAO | [09:34] Bruno |
| FDD-AC-03 | docs/FDD.md | Critério de Aceite | Payload é snapshot | TRANSCRICAO | [09:52] Larissa |
| FDD-AC-04 | docs/FDD.md | Critério de Aceite | URL não https recusada com 400 WEBHOOK_INVALID_URL | TRANSCRICAO | [09:23] Sofia |
| FDD-AC-05 | docs/FDD.md | Critério de Aceite | X-Signature verificável; duas secrets durante a convivência de 24 h | TRANSCRICAO | [09:22] Sofia |
| FDD-AC-06 | docs/FDD.md | Critério de Aceite | Cinco headers presentes; corpo sem itens | TRANSCRICAO | [09:44] Diego; [09:43] Diego |
| FDD-AC-07 | docs/FDD.md | Critério de Aceite | Timeout de 10 s vira falha e é reagendado | TRANSCRICAO | [09:42] Diego |
| FDD-AC-08 | docs/FDD.md | Critério de Aceite | Reagendamento 1m/5m/30m/2h/12h e DLQ após o 6º envio | TRANSCRICAO | [09:17] Larissa |
| FDD-AC-09 | docs/FDD.md | Critério de Aceite | Replay: OPERATOR 403; ADMIN 202 com mesmo event_id e log do userId | TRANSCRICAO | [09:36] Sofia |
| FDD-AC-10 | docs/FDD.md | Critério de Aceite | Worker por npm run worker, separado; SIGTERM gracioso | TRANSCRICAO | [09:11] Diego |
| FDD-AC-11 | docs/FDD.md | Critério de Aceite | Evento chega em menos de 10 s após o commit | TRANSCRICAO | [09:02] Marcos |
| FDD-AC-12 | docs/FDD.md | Critério de Aceite | Reenvio com mesmo X-Event-Id após restart do worker | TRANSCRICAO | [09:25] Diego |
| FDD-AC-13 | docs/FDD.md | Critério de Aceite | Corpo acima de 64 KB vai à DLQ com WEBHOOK_PAYLOAD_TOO_LARGE | TRANSCRICAO | [09:24] Larissa |
| FDD-AC-14 | docs/FDD.md | Critério de Aceite | Secret ausente de respostas e logs | TRANSCRICAO | [09:31] Marcos |
| FDD-AC-15 | docs/FDD.md | Critério de Aceite | Histórico devolve no máximo 100 itens, mais recente primeiro | TRANSCRICAO | [09:34] Marcos |
| FDD-AC-16 | docs/FDD.md | Critério de Aceite | Eventos do mesmo pedido, sem falhas, na ordem de created_at | TRANSCRICAO | [09:12] Diego |
| FDD-AC-17 | docs/FDD.md | Critério de Aceite | Todo erro do módulo usa WEBHOOK_* ou código existente | TRANSCRICAO | [09:29] Larissa |
| FDD-AC-18 | docs/FDD.md | Critério de Aceite | tests/orders.test.ts e tests/auth.test.ts seguem passando | CODIGO | tests/orders.test.ts; tests/auth.test.ts |
| FDD-RISCO-01 | docs/FDD.md | Risco | Transação de changeStatus mais longa pode se aproximar do timeout do Prisma | TRANSCRICAO | [09:04] Bruno |
| FDD-RISCO-02 | docs/FDD.md | Risco | Bloqueio em cabeça de fila: cliente lento atrasa os demais | PROPOSTA | Relacionada a [09:12] Diego; [09:42] Diego |
| FDD-RISCO-03 | docs/FDD.md | Risco | SSRF: URL pode apontar para rede interna | PROPOSTA | Não discutido na reunião |
| FDD-RISCO-04 | docs/FDD.md | Risco | Secret em texto no banco; CRUD aberto a qualquer role | TRANSCRICAO | [09:37] Sofia; [09:22] Diego |
| FDD-RISCO-05 | docs/FDD.md | Risco | Crescimento de outbox e deliveries sem retenção | TRANSCRICAO | [09:08] Diego |
| FDD-RISCO-06 | docs/FDD.md | Risco | Duplicatas por queda entre o 2xx e DELIVERED, ou replay | TRANSCRICAO | [09:25] Diego; [09:26] Marcos |
| FDD-RISCO-07 | docs/FDD.md | Risco | Eventos do mesmo pedido fora de ordem durante retry | PROPOSTA | Relacionada a [09:12] Diego |
| FDD-RISCO-08 | docs/FDD.md | Risco | Worker parado sem ninguém perceber | PROPOSTA | Sem origem direta na reunião |
| FDD-RISCO-09 | docs/FDD.md | Risco | Cliente estranha X-Signature com duas assinaturas | PROPOSTA | Relacionada a [09:21] Sofia |
| FDD-RISCO-10 | docs/FDD.md | Risco | Worker exige JWT_SECRET por causa da validação de env.ts | CODIGO | src/config/env.ts |
| FDD-RISCO-11 | docs/FDD.md | Risco | Prazo de 3 sprints frente ao pedido da Atlas | TRANSCRICAO | [09:45] Marcos; [09:46] Larissa; [09:46] Sofia |
| FDD-PEND-07 | docs/FDD.md | Proposta pendente | Q7 retry × ordering: não bloqueia os seguintes do mesmo pedido | PROPOSTA | Relacionada a [09:12] Diego |
| FDD-PEND-08 | docs/FDD.md | Proposta pendente | Q8 criação de pedido não gera evento; OrderService.create intacto | CODIGO | src/modules/orders/order.service.ts |
| FDD-PEND-09 | docs/FDD.md | Proposta pendente | Q9 contagem de 5 tentativas: 1 + 5 retries | PROPOSTA | Relacionada a [09:17] Diego |
| FDD-PEND-10 | docs/FDD.md | Proposta pendente | Q10 X-Timestamp fora da assinatura | PROPOSTA | Relacionada a [09:44] Diego |
| FDD-PEND-11 | docs/FDD.md | Proposta pendente | Q11 secret legível pelo worker; criptografia em repouso a decidir | PROPOSTA | Relacionada a [09:22] Sofia |
| FDD-PEND-12 | docs/FDD.md | Proposta pendente | Q12 envio sequencial; concorrência limitada fica como evolução | PROPOSTA | Relacionada a [09:12] Diego |
| FDD-PEND-13 | docs/FDD.md | Proposta pendente | Q13 duas assinaturas no X-Signature na rotação | PROPOSTA | Relacionada a [09:21] Sofia |
| FDD-PEND-14 | docs/FDD.md | Proposta pendente | Q14 métricas por SQL e logs, sem prom-client nem OpenTelemetry | CODIGO | package.json |
| FDD-SEQ-01 | docs/FDD.md | Decisão | Estimativa de 3 sprints: outbox/DLQ 1, worker/retry 1, CRUD/entregas 0,5, integração e testes 0,5, HMAC e schemas | TRANSCRICAO | [09:46] Larissa |
| FDD-SEQ-02 | docs/FDD.md | Restrição | Revisão de segurança de 2 dias úteis ao fim da sequência | TRANSCRICAO | [09:46] Sofia |
| FDD-SEQ-03 | docs/FDD.md | Proposta | Ordem de PRs: migration, fundações, publicação, worker, API, observabilidade, revisão de segurança | PROPOSTA | Relacionada a [09:46] Larissa |
| FDD-ESTR-01 | docs/FDD.md | Decisão | Estrutura src/modules/webhooks (a criar: controller, service, repository, routes, schemas) mais worker.ts e processor | TRANSCRICAO | [09:27] Bruno; [09:28] Bruno |
| FDD-ESTR-02 | docs/FDD.md | Restrição | Padrão controller/service/repository/routes/schemas de src/modules/orders | CODIGO | src/modules/orders/order.routes.ts |
| FDD-CTX-01 | docs/FDD.md | Contexto | Origem do evento é uma transação pesada: orders, order_status_history e estoque | TRANSCRICAO | [09:04] Bruno |
| FDD-CTX-02 | docs/FDD.md | Contexto | Falha de cliente não pode travar nem reverter o status | TRANSCRICAO | [09:04] Bruno |
| FDD-CTX-03 | docs/FDD.md | Contexto | Evento e status precisam ser atômicos | TRANSCRICAO | [09:40] Bruno |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Outbox transacional no MySQL existente, na mesma transação da mudança de status | TRANSCRICAO | [09:06] Diego; [09:07] Diego; [09:08] Larissa |
| ADR-001-ALT-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa descartada | Disparo síncrono dentro do service de pedidos (travaria status e exigiria rollback) | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| ADR-001-ALT-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa descartada | Fila externa (Redis Streams): infraestrutura nova, overengineering para time pequeno | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| ADR-001-TRADE | docs/adrs/ADR-001-outbox-no-mysql.md | Trade-off | Falha na outbox impede a mudança de status; outbox cresce no mesmo banco (arquivamento fica para depois) | TRANSCRICAO | [09:40] Bruno; [09:08] Diego |
| ADR-002 | docs/adrs/ADR-002-worker-separado-com-polling-de-2s.md | Decisão | Worker em processo separado (src/worker.ts (a criar), npm run worker) com polling de 2 s sobre a outbox | TRANSCRICAO | [09:09] Diego; [09:10] Larissa; [09:11] Diego; [09:11] Larissa |
| ADR-002-ALT-01 | docs/adrs/ADR-002-worker-separado-com-polling-de-2s.md | Alternativa descartada | Trigger do MySQL para acionar o worker (sem listener nativo; só executa SQL) | TRANSCRICAO | [09:09] Diego |
| ADR-002-ALT-02 | docs/adrs/ADR-002-worker-separado-com-polling-de-2s.md | Alternativa descartada | Worker dentro da instância da API (restart da API perderia o worker) | TRANSCRICAO | [09:11] Diego |
| ADR-002-TRADE | docs/adrs/ADR-002-worker-separado-com-polling-de-2s.md | Trade-off | Latência mínima de até 2 s e ordering só por order_id com worker único; escalar exige nova decisão | TRANSCRICAO | [09:10] Larissa; [09:12] Diego; [09:13] Larissa |
| ADR-003 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | 5 tentativas com backoff 1m/5m/30m/2h/12h; esgotadas, DLQ em tabela separada com replay manual | TRANSCRICAO | [09:15] Diego; [09:17] Larissa; [09:18] Diego |
| ADR-003-ALT-01 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Alternativa descartada | Retry indefinido com backoff (evento pendurado se o cliente sumiu) | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Alternativa descartada | Apenas 3 tentativas (cobriria ~30 min; já houve indisponibilidade de 2 h) | TRANSCRICAO | [09:16] Diego |
| ADR-003-TRADE | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Trade-off | Cliente fora por mais de ~15 h depende de replay manual; sem aviso por e-mail | TRANSCRICAO | [09:17] Marcos; [09:37] Larissa |
| ADR-004 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | HMAC-SHA256 sobre o corpo, secret única por endpoint, rotação com grace period de 24 h | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| ADR-004-ALT | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Alternativa descartada | Secret global da plataforma (vazamento comprometeria todos os clientes) | TRANSCRICAO | [09:21] Sofia |
| ADR-004-TRADE | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Trade-off | Duas secrets ativas durante a rotação; X-Timestamp enviado mas sem decisão sobre entrar na assinatura; 2 dias úteis de revisão | TRANSCRICAO | [09:44] Diego; [09:46] Sofia |
| ADR-004-COD | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Restrição | Logger atual não faz redact de secret (cobre password, token, authorization, cookie) | CODIGO | src/shared/logger/index.ts |
| ADR-005 | docs/adrs/ADR-005-at-least-once-com-x-event-id.md | Decisão | Entrega at-least-once; deduplicação pelo cliente via X-Event-Id (UUID gerado na entrada da outbox) | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa |
| ADR-005-ALT | docs/adrs/ADR-005-at-least-once-com-x-event-id.md | Alternativa descartada | Exactly-once (coordenação dos dois lados, muito mais complexo) | TRANSCRICAO | [09:25] Diego |
| ADR-005-TRADE | docs/adrs/ADR-005-at-least-once-com-x-event-id.md | Trade-off | Responsabilidade de deduplicar passa ao cliente; exige documentação destacada | TRANSCRICAO | [09:25] Sofia; [09:26] Marcos |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Reuso máximo dos padrões: módulo em src/modules/webhooks (a criar), AppError, Pino, error middleware, Zod, prefixo WEBHOOK_ | TRANSCRICAO | [09:27] Bruno; [09:28] Bruno; [09:29] Larissa; [09:30] Larissa |
| ADR-006-ALT | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Alternativa descartada | Novos frameworks de fila, logging ou erros ("não vamos botar nada novo") | TRANSCRICAO | [09:29] Bruno; [09:30] Larissa |
| ADR-006-COD-01 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Restrição | NotFoundError fixa o código NOT_FOUND no construtor; WEBHOOK_NOT_FOUND exige subclasse de AppError | CODIGO | src/shared/errors/http-errors.ts |
| ADR-006-COD-02 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Restrição | Fiação manual: o módulo exige editar buildControllers e o tipo Controllers | CODIGO | src/app.ts; src/routes/index.ts |
| ADR-007 | docs/adrs/ADR-007-publicacao-transacional-com-snapshot-e-filtro.md | Decisão | publishWebhookEvent(tx, order, from, to) dentro da transação, com snapshot do payload e filtro por status na inserção | TRANSCRICAO | [09:34] Bruno; [09:40] Bruno; [09:41] Bruno; [09:41] Diego; [09:52] Larissa |
| ADR-007-ALT-01 | docs/adrs/ADR-007-publicacao-transacional-com-snapshot-e-filtro.md | Alternativa descartada | Guardar só order_id e renderizar no envio | TRANSCRICAO | [09:52] Larissa |
| ADR-007-ALT-02 | docs/adrs/ADR-007-publicacao-transacional-com-snapshot-e-filtro.md | Alternativa descartada | Injetar o repository de webhooks no OrderService | TRANSCRICAO | [09:41] Bruno; [09:41] Diego |
| ADR-007-ALT-03 | docs/adrs/ADR-007-publicacao-transacional-com-snapshot-e-filtro.md | Alternativa descartada | Filtrar só no momento do envio (gasta linhas da outbox) | TRANSCRICAO | [09:34] Bruno |
| ADR-007-TRADE | docs/adrs/ADR-007-publicacao-transacional-com-snapshot-e-filtro.md | Trade-off | Snapshot não reflete correções posteriores; consulta de webhooks amplia a transação de changeStatus | TRANSCRICAO | [09:52] Larissa; [09:40] Bruno |
| ADR-007-COD | docs/adrs/ADR-007-publicacao-transacional-com-snapshot-e-filtro.md | Restrição | OrderService.create grava histórico PENDING com fromStatus null; a reunião só tratou de mudança de status | CODIGO | src/modules/orders/order.service.ts |
| ADR-001-CTX | docs/adrs/ADR-001-outbox-no-mysql.md | Contexto | Contexto: clientes B2B consultam GET /orders hoje; abaixo de 10 s já é tempo real | TRANSCRICAO | [09:00] Marcos; [09:02] Marcos |
| ADR-002-CTX | docs/adrs/ADR-002-worker-separado-com-polling-de-2s.md | Contexto | Contexto: MySQL não tem listener nativo e trigger não notifica processo externo | TRANSCRICAO | [09:09] Diego |
| ADR-003-CTX | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Contexto | Contexto: já houve cliente com indisponibilidade de 2 h em manutenção planejada | TRANSCRICAO | [09:16] Diego |
| ADR-004-CTX | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Contexto | Contexto: dados de pedidos saem para endpoint fora da infra; cliente precisa validar origem e integridade | TRANSCRICAO | [09:19] Sofia |
| ADR-005-CTX | docs/adrs/ADR-005-at-least-once-com-x-event-id.md | Contexto | Contexto: cliente pode receber o mesmo evento duas vezes e precisa estar preparado | TRANSCRICAO | [09:24] Diego |
| ADR-006-CTX | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Contexto | Contexto: cada domínio é módulo com controller, service, repository, routes e schemas | TRANSCRICAO | [09:27] Bruno |
| ADR-007-CTX | docs/adrs/ADR-007-publicacao-transacional-com-snapshot-e-filtro.md | Contexto | Contexto: a alteração crítica é em changeStatus, na mesma transação | TRANSCRICAO | [09:40] Bruno |
