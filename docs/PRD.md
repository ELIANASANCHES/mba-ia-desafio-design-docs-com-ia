# PRD: Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
| --- | --- |
| **Autor** | Marcos (Product Manager), com apoio de Larissa (Tech Lead) |
| **Status** | Em revisão (acompanha o [RFC](RFC.md) e o [FDD](FDD.md)) |
| **Data** | 2026-11-25 |
| **Revisores** | Larissa, Bruno, Diego, Sofia |
| **Documentos relacionados** | [RFC](RFC.md) · [FDD](FDD.md) · [ADRs](adrs/README.md) · [Tracker](TRACKER.md) |

**Como ler a coluna "Origem".** Cada item cita sua fonte: `[hh:mm] Nome` é uma fala da reunião técnica (`TRANSCRICAO.md`); um caminho de arquivo é um fato do código; **Proposta** é algo que a reunião não definiu e que foi preenchido para o documento ser acionável. Propostas são sugestões a validar na revisão, não decisões. Os IDs (`OBJ-`, `RF-`, `RNF-`, `RISCO-`, `AC-`) são usados pelo [Tracker](TRACKER.md).

---

## 1. Resumo e contexto da feature

O Order Management System (OMS) passará a **notificar clientes B2B por webhooks de saída** sempre que o status de um pedido deles mudar. Cada cliente cadastra uma URL `https`, escolhe os status que quer ouvir e recebe, em menos de 10 segundos, um evento assinado com HMAC-SHA256. Em caso de falha, o sistema reenvia automaticamente (5 tentativas com backoff exponencial) e, esgotadas, preserva o evento em uma fila de falhas (DLQ) que um administrador pode reprocessar.

O OMS é uma aplicação Node.js + TypeScript (Express, Prisma sobre MySQL) com módulos de autenticação, usuários, clientes, produtos e pedidos. Ele **não possui hoje nenhum mecanismo de notificação externa, evento, fila ou webhook**; esta é a primeira funcionalidade desse tipo.

A solução técnica já foi decidida na reunião e está detalhada no RFC, nos ADRs e no FDD. Este PRD responde **por que e o quê**; não repete o **como**.

## 2. Problema e motivação

Atlas Comercial, MaxDistribuição e Nova Cargo fizeram um pedido formal para serem notificados em tempo real quando o status de seus pedidos muda (`[09:00] Marcos`). Hoje eles consultam `GET /orders` de tempos em tempos para descobrir mudanças, o que torna a integração **lenta e cara** para eles (`[09:00] Marcos`).

- **Risco comercial.** A Atlas sugeriu que pode migrar para um concorrente se a funcionalidade não for entregue até o fim do trimestre (`[09:00] Marcos`), e pediu a entrega para o fim de novembro (`[09:45] Marcos`).
- **Definição de "tempo real".** Para os clientes, qualquer coisa abaixo de 10 segundos já é tempo real; o essencial é não precisarem atualizar manualmente (`[09:02] Marcos`).
- **Restrições que moldam a solução.** A mudança de status é uma transação já pesada (atualiza `orders`, insere em `order_status_history` e movimenta estoque, `src/modules/orders/order.service.ts`). Um cliente lento ou fora do ar não pode travar nem reverter essa mudança (`[09:04] Bruno`), e o evento não pode se perder nem sair para uma mudança revertida (`[09:40] Bruno`, `[09:41] Diego`).

## 3. Público-alvo e cenários de uso

### 3.1 Públicos

| Público | Papel na feature | Origem |
| --- | --- | --- |
| **Clientes B2B** (inicialmente Atlas Comercial, MaxDistribuição e Nova Cargo) | Recebem as notificações e integram seus sistemas | `[09:00] Marcos` |
| **Usuários do OMS que representam o cliente** | Cadastram e gerenciam os webhooks pela API, autenticados com JWT do nosso sistema | `[09:32] Marcos` |
| **Administradores (role `ADMIN`)** | Reprocessam eventos da DLQ | `[09:36] Sofia` |
| **Time de engenharia e Segurança** | Implementam, operam e revisam a feature | `[09:46] Larissa`, `[09:46] Sofia` |
| **Product Manager** | Documenta a integração no portal de desenvolvedores | `[09:26] Marcos`, `[09:40] Marcos` |

### 3.2 Cenários de uso

1. **Acompanhar entregas sem consultar a API.** A Atlas cadastra um webhook e passa a receber um evento a cada mudança de status dos seus pedidos, em vez de chamar `GET /orders` periodicamente (`[09:00] Marcos`).
2. **Receber só o que interessa.** Um cliente configura o webhook para ouvir apenas `SHIPPED` e `DELIVERED` (`[09:33] Marcos`).
3. **Cliente temporariamente indisponível.** O sistema do cliente fica fora do ar por uma manutenção de duas horas; as retentativas cobrem a janela e o cliente recebe os eventos quando voltar (`[09:16] Diego`).
4. **Suspeita de vazamento de secret.** O cliente rotaciona a secret pela API e tem 24 horas para migrar seus sistemas, enquanto a antiga continua válida (`[09:21] Sofia`, `[09:22] Diego`).
5. **Auditoria do que foi enviado.** O cliente consulta os últimos 100 envios, com sucesso ou falha, payload, resposta e tempo de resposta (`[09:34] Marcos`).
6. **Evento que esgotou as tentativas.** Um administrador reprocessa o item da DLQ depois que o cliente corrige o problema; a ação fica registrada para auditoria (`[09:18] Diego`, `[09:36] Sofia`).

## 4. Objetivos e métricas de sucesso

| ID | Objetivo | Métrica | Meta | Origem |
| --- | --- | --- | --- | --- |
| OBJ-01 | Notificar em tempo real | Tempo entre o commit da mudança de status e a chegada ao endpoint, em condições normais | **Menos de 10 segundos** (polling de 2 s no pior caso, somado ao tempo de entrega) | `[09:02] Marcos`, `[09:10] Larissa` |
| OBJ-02 | Nunca perder nem inventar evento | Mudanças de status confirmadas sem evento correspondente (para endpoints interessados), e eventos de mudanças revertidas | **0** (garantia por construção: mesma transação) | `[09:40] Bruno`, `[09:41] Diego` |
| OBJ-03 | Isolar o pedido da falha do cliente | Mudanças de status atrasadas ou revertidas por lentidão ou queda de endpoint de cliente | **0** | `[09:04] Bruno` |
| OBJ-04 | Cobrir indisponibilidades prolongadas do cliente | Janela de retentativas antes de a falha ser considerada permanente | **5 retentativas em até 14 h 36 min** (1 min, 5 min, 30 min, 2 h, 12 h); nenhum evento descartado, só movido para a DLQ | `[09:15] Diego`, `[09:17] Larissa` |
| OBJ-05 | Reter os clientes que pediram a feature | Clientes com webhook em produção até o fim de novembro | **3 de 3** (Atlas, MaxDistribuição, Nova Cargo) | `[09:00] Marcos`, `[09:45] Marcos` |
| OBJ-06 | Entregar no prazo acordado | Duração da implementação, incluindo a revisão de segurança | **3 sprints**, com 2 dias úteis finais para a revisão de Segurança | `[09:46] Larissa`, `[09:46] Sofia` |
| OBJ-07 | Não degradar o OMS existente | Testes existentes (`tests/orders.test.ts`, `tests/auth.test.ts`) passando sem mudança de comportamento | **100%** | `src/modules/orders/order.service.ts`; Proposta (meta) |

**Metas de operação sugeridas (Proposta, a calibrar com Produto e Plataforma).** A reunião não definiu metas de taxa de sucesso nem de alertas. Sugerimos acompanhar a idade do evento mais pendente (alerta acima de 30 s, já que o requisito é inferior a 10 s), a quantidade de itens abertos na DLQ e a ausência de atividade do worker por mais de 1 minuto (FDD, seção 9.2). Uma meta de percentual de entregas no primeiro envio não foi discutida e fica como pendência de Produto.

## 5. Escopo

### 5.1 Dentro do escopo

- Webhooks **de saída**: nós enviamos, o cliente recebe (`[09:02] Marcos`).
- Evento único `order.status_changed`, disparado pela mudança de status do pedido (`[09:43] Diego`).
- Cadastro, edição, remoção e listagem de webhooks por cliente, com filtro de status por endpoint (`[09:31] Marcos`, `[09:33] Bruno`).
- Entrega assíncrona por worker em processo separado, com retry, DLQ e replay administrativo (`[09:11] Diego`, `[09:17] Larissa`, `[09:18] Diego`).
- Assinatura HMAC-SHA256, secret por endpoint e rotação com 24 h de convivência (`[09:22] Sofia`).
- Histórico de entregas consultável pelo cliente (`[09:34] Marcos`).
- Documentação de integração no portal de desenvolvedores, a cargo de Produto (`[09:26] Marcos`, `[09:40] Marcos`).

### 5.2 Fora de escopo

Itens explicitamente descartados ou adiados na reunião:

| Item | Situação | Origem |
| --- | --- | --- |
| **Aviso por e-mail** ao cliente quando o webhook falha repetidamente | Adiado para a próxima fase, depois de medir o impacto | `[09:37] Larissa`, `[09:38] Marcos` |
| **Rate limiting de envio por cliente** | "Observar e decidir depois"; registrado como ponto em aberto | `[09:38] Diego`, `[09:39] Larissa` |
| **Dashboard visual** para o cliente acompanhar seus webhooks | Descartado nesta fase; é projeto separado do time de frontend. Só endpoints | `[09:39] Marcos`, `[09:40] Larissa` |
| **Múltiplos workers em paralelo** e ordering global | Adiado; vale a limitação de worker único e ordering por `order_id` | `[09:12] Diego`, `[09:13] Larissa` |
| **Arquivamento das linhas entregues** da outbox (cerca de 30 dias) | Fora do escopo desta feature | `[09:08] Diego` |
| **Webhooks de entrada** (cliente para nós) | Não solicitado | `[09:02] Marcos` |
| **Itens do pedido no payload** | Excluídos para manter o payload enxuto; o cliente consulta `GET /orders/:id` | `[09:43] Diego` |
| **Endurecer permissões** do CRUD de configuração (hoje, qualquer role autenticada) | Mantido "por enquanto"; pode ser revisto | `[09:37] Sofia` |

Não discutidos na reunião e, por isso, fora desta especificação (Proposta): listagem de itens da DLQ via API, evento de criação de pedido (questão Q8 do RFC) e alertas automáticos.

## 6. Requisitos funcionais

| ID | Requisito | Prioridade | Origem |
| --- | --- | --- | --- |
| RF-01 | O cliente cadastra um webhook informando a URL e a lista de status que deseja receber; a secret é **gerada pelo sistema** e devolvida na criação | Must | `[09:31] Marcos` |
| RF-02 | O `customer_id` é informado no body (ou no path/query), e **não** extraído do JWT, pois o JWT identifica um usuário do nosso sistema | Must | `[09:32] Bruno`, `[09:32] Larissa` |
| RF-03 | O cliente edita (`PATCH`), remove (`DELETE`) e lista (`GET`) os webhooks de um customer | Must | `[09:33] Bruno` |
| RF-04 | Cada webhook escolhe quais status quer ouvir; o filtro é aplicado **na inserção** da outbox, e se nenhum webhook do customer quer o status, nada é inserido | Must | `[09:33] Marcos`, `[09:34] Bruno` |
| RF-05 | Toda mudança de status relevante registra o evento na outbox **na mesma transação** da mudança; se a gravação do evento falhar, a mudança de status é revertida | Must | `[09:03] Larissa`, `[09:06] Diego`, `[09:40] Bruno` |
| RF-06 | O evento guarda um **snapshot** do payload no momento da mudança, e não apenas o `order_id` | Must | `[09:51] Bruno`, `[09:52] Larissa` |
| RF-07 | Um worker em **processo separado** lê a outbox em polling de 2 segundos e entrega o evento por HTTPS (`src/worker.ts` *(a criar)*, `npm run worker`) | Must | `[09:09] Diego`, `[09:11] Diego`, `[09:11] Larissa` |
| RF-08 | A entrega usa **timeout de 10 segundos**; sem resposta, é falha e entra em retry | Must | `[09:42] Diego` |
| RF-09 | Em caso de falha, o sistema reenvia com **backoff exponencial** de 1 min, 5 min, 30 min, 2 h e 12 h (5 tentativas) | Must | `[09:15] Diego`, `[09:17] Larissa` |
| RF-10 | Esgotadas as tentativas, o evento vai para a **DLQ**, em tabela separada, com payload, motivo da falha e timestamp | Must | `[09:17] Larissa`, `[09:18] Diego` |
| RF-11 | Um administrador reprocessa um item da DLQ por `POST /admin/webhooks/dead-letter/:id/replay`, que recoloca o evento na outbox como pendente. Exige role `ADMIN` e **registra quem executou** | Must | `[09:18] Diego`, `[09:35] Diego`, `[09:36] Sofia` |
| RF-12 | Cada envio é assinado com **HMAC-SHA256 sobre o corpo**, enviado em `X-Signature`, com **secret única por endpoint** | Must | `[09:20] Sofia`, `[09:21] Sofia`, `[09:22] Sofia` |
| RF-13 | O cliente **rotaciona a secret pela API**; a secret antiga continua válida por **24 horas** em paralelo e depois deixa de valer | Must | `[09:21] Sofia`, `[09:22] Sofia` |
| RF-14 | O cliente consulta o **histórico de entregas** (`GET /webhooks/:id/deliveries`): últimos 100 envios, com sucesso ou falha, payload, resposta e tempo de resposta | Must | `[09:34] Marcos` |
| RF-15 | A entrega é **at-least-once**; cada evento leva um `X-Event-Id` (UUID, gerado na entrada da outbox) para o cliente deduplicar | Must | `[09:24] Diego`, `[09:25] Diego`, `[09:26] Larissa` |
| RF-16 | Cada requisição leva os headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type: application/json` | Must | `[09:44] Diego`, `[09:44] Sofia`, `[09:45] Diego` |
| RF-17 | O payload é JSON com `event_id`, `event_type` (`order.status_changed`), `timestamp` ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos básicos do pedido (como `total_cents`), **sem itens** | Must | `[09:43] Diego` |
| RF-18 | A URL cadastrada deve ser `https`; `http` é recusado com erro de validação | Must | `[09:23] Sofia` |
| RF-19 | Eventos com corpo acima de **64 KB** não são enviados e geram erro | Must | `[09:23] Sofia`, `[09:24] Diego`, `[09:24] Larissa` |
| RF-20 | Os erros do módulo seguem o padrão do projeto: classes sobre `AppError`, com códigos prefixados por `WEBHOOK_` (por exemplo `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED`) | Should | `[09:28] Bruno`, `[09:29] Larissa` |

Os contratos, os campos e a matriz completa de erros estão no [FDD](FDD.md), seções 6 e 7.

## 7. Requisitos não funcionais

| ID | Requisito | Categoria | Origem |
| --- | --- | --- | --- |
| RNF-01 | **Latência:** evento entregue em menos de 10 s após o commit, em condições normais | Desempenho | `[09:02] Marcos`, `[09:10] Larissa` |
| RNF-02 | **Consistência:** evento e mudança de status são atômicos (mesma transação SQL) | Confiabilidade | `[09:06] Diego`, `[09:40] Bruno` |
| RNF-03 | **Isolamento:** nenhuma chamada HTTP ocorre dentro da transação de pedidos; a API e o worker são processos independentes | Disponibilidade | `[09:04] Bruno`, `[09:11] Diego` |
| RNF-04 | **Garantia de entrega:** at-least-once; o cliente é responsável pela deduplicação | Confiabilidade | `[09:24] Diego`, `[09:26] Larissa` |
| RNF-05 | **Ordenação:** apenas por `order_id` e enquanto houver um único worker; sem garantia de ordering global (limitação conhecida) | Consistência | `[09:12] Diego`, `[09:13] Larissa`, `[09:14] Marcos` |
| RNF-06 | **Autenticidade e integridade:** HMAC-SHA256 com secret por endpoint e rotação com convivência de 24 h | Segurança | `[09:19] Sofia`, `[09:22] Sofia` |
| RNF-07 | **Transporte:** TLS obrigatório (URL `https`) | Segurança | `[09:23] Sofia` |
| RNF-08 | **Controle de acesso:** CRUD de configuração para qualquer role autenticada, por enquanto; replay de DLQ somente `ADMIN`, com `requireRole` existente | Segurança | `[09:36] Larissa`, `[09:37] Sofia` |
| RNF-09 | **Auditoria:** o replay registra quem o executou | Segurança | `[09:36] Sofia` |
| RNF-10 | **Limite de payload:** 64 KB | Robustez | `[09:24] Larissa` |
| RNF-11 | **Reuso de padrões:** módulo em `src/modules/webhooks` *(a criar)* (controller, service, repository, routes, schemas), Pino, error middleware, Zod, UUID como chave primária | Manutenibilidade | `[09:27] Bruno`, `[09:30] Larissa`, `[09:51] Larissa` |
| RNF-12 | **Sem infraestrutura nova:** outbox no MySQL existente, sem Redis ou fila externa; o worker usa o mesmo banco, com instância própria do `PrismaClient` | Simplicidade operacional | `[09:07] Diego`, `[09:30] Bruno` |
| RNF-13 | **Revisão de segurança antes do deploy:** mínimo de 2 dias úteis para HMAC e geração de secret | Segurança | `[09:46] Sofia`, `[09:49] Sofia` |
| RNF-14 | **Observabilidade:** logs em Pino; métricas por SQL e logs, sem dependência nova | Operação | `[09:29] Bruno`; métricas, Proposta (FDD, seção 9) |

## 8. Decisões e trade-offs principais

Cada decisão está registrada em um ADR; o RFC apresenta as alternativas. Aqui ficam o resumo e o custo aceito do ponto de vista do produto.

| Decisão | Alternativa descartada | Trade-off aceito | ADR |
| --- | --- | --- | --- |
| **Outbox no MySQL**, na mesma transação do status | Disparo síncrono; fila externa (Redis Streams) | Evento nunca se perde e não há infraestrutura nova, ao custo de a falha na outbox derrubar a mudança de status | [ADR-001](adrs/ADR-001-outbox-no-mysql.md) |
| **Worker separado, polling de 2 s**, worker único | Trigger do banco; worker dentro da API | Latência mínima de até 2 s e ordering só por `order_id`; ganha simplicidade e resiliência a restart da API | [ADR-002](adrs/ADR-002-worker-separado-com-polling-de-2s.md) |
| **5 tentativas com backoff** e **DLQ em tabela separada**, com replay manual | Retry indefinido; apenas 3 tentativas | Falha permanente só após cerca de 15 h e depende de ação manual; evita evento pendurado para sempre sem encerrar cedo demais | [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md) |
| **HMAC-SHA256 com secret por endpoint** e rotação de 24 h | Secret global da plataforma | Mais gestão de secrets; um vazamento não compromete os demais clientes | [ADR-004](adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md) |
| **At-least-once** com `X-Event-Id` | Exactly-once | O cliente precisa deduplicar (documentado no portal); evita coordenação complexa entre os dois lados | [ADR-005](adrs/ADR-005-at-least-once-com-x-event-id.md) |
| **Reuso máximo** dos padrões do projeto | Estrutura própria para o módulo | Segue convenções existentes; menor curva para o time | [ADR-006](adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md) |
| **Publicação transacional com snapshot** e filtro na inserção | Guardar só `order_id` e renderizar no envio | O evento reflete o estado do momento da mudança; o payload não acompanha alterações posteriores do pedido | [ADR-007](adrs/ADR-007-publicacao-transacional-com-snapshot-e-filtro.md) |

## 9. Dependências

| Dependência | Tipo | Detalhe | Origem |
| --- | --- | --- | --- |
| `OrderService.changeStatus` | Código existente | Único ponto de integração com a lógica de negócio atual; passa a chamar `publishWebhookEvent(tx, order, fromStatus, toStatus)` dentro da transação | `[09:40] Bruno`, `[09:41] Bruno` |
| MySQL e Prisma (novas tabelas via migration) | Infraestrutura existente | Outbox, DLQ, configuração e histórico de entregas, com UUID como PK | `[09:06] Diego`, `[09:51] Larissa` |
| Padrões do projeto (`AppError`, Pino, error middleware, `requireRole`, Zod) | Código existente | Reutilizados sem mudança | `[09:29] Bruno`, `[09:36] Larissa` |
| Segundo processo (`npm run worker`) | Operação | Precisa ser implantado, monitorado e reiniciado; **como será empacotado em produção não está definido** (não há Dockerfile no repositório) | `[09:11] Larissa`; pendência, FDD seção 11 |
| Revisão de segurança da Sofia | Pessoas | Mínimo de 2 dias úteis ao fim, antes do deploy | `[09:46] Sofia` |
| Portal de desenvolvedores | Produto | Documentar integração, verificação de assinatura e deduplicação por `X-Event-Id` | `[09:26] Marcos`, `[09:40] Marcos` |
| Comunicação com os clientes | Produto | Marcos confirma o prazo com a Atlas e atualiza os clientes | `[09:47] Marcos`, `[09:49] Marcos` |
| Clientes B2B | Externa | Precisam implementar verificação de assinatura e deduplicação | `[09:25] Diego`, `[09:26] Marcos` |
| Dependências npm novas | Código | Nenhuma prevista (`fetch`, `AbortController` e `node:crypto` nativos no Node 20) | `[09:29] Bruno`; confirmação, Proposta (FDD seção 11) |

## 10. Riscos e mitigação

| ID | Risco | Prob. | Impacto | Mitigação | Origem |
| --- | --- | --- | --- | --- | --- |
| RISCO-01 | **Prazo:** 3 sprints (incluindo a revisão de segurança) frente ao pedido da Atlas para o fim de novembro; a duração do sprint e o calendário não foram registrados na reunião | Média | Alto | Confirmar o calendário com Larissa e Marcos; seguir a sequência de implementação do FDD; reservar 2 dias úteis para a Sofia; alinhar com a Atlas o que for possível entregar até o fim do mês | `[09:45] Marcos`, `[09:46] Larissa`, `[09:46] Sofia` |
| RISCO-02 | **Falha na outbox bloqueia a mudança de status** (consequência da atomicidade) | Baixa | Alto | Aceito por decisão; testes ponta a ponta de `changeStatus`; medir a duração da transação antes do deploy | `[09:40] Bruno`, `[09:46] Larissa`; FDD-RISCO-01 |
| RISCO-03 | **Vazamento de secret de cliente** (já ocorreu antes, em log de aplicação do cliente) | Média | Alto | Secret por endpoint, rotação com 24 h de convivência, redact de `secret` nos logs, revisão de segurança | `[09:22] Diego`, `[09:21] Sofia` |
| RISCO-04 | **Cliente não deduplica** e processa o mesmo evento duas vezes | Média | Médio | Documentação destacada no portal; `X-Event-Id` estável entre reenvios e replays | `[09:25] Diego`, `[09:26] Marcos` |
| RISCO-05 | **Cliente lento atrasa os demais:** o envio é sequencial e cada chamada pode levar até 10 s, o que pode romper o requisito de menos de 10 s | Média | Alto | Monitorar a idade do evento mais antigo; reduzir o batch; avaliar concorrência limitada preservando a ordem por `order_id` | Proposta (decorre de `[09:12] Diego`, `[09:42] Diego`); RFC Q12 |
| RISCO-06 | **Crescimento da outbox** e do histórico de entregas, sem arquivamento nesta feature | Média | Médio | Índices em status e `created_at`; tratar o arquivamento em feature posterior; definir retenção do histórico | `[09:08] Diego`; FDD-RISCO-05 |
| RISCO-07 | **Worker parado sem ninguém perceber:** os eventos acumulam e a promessa de menos de 10 s deixa de valer | Média | Alto | Alertas de idade do evento mais antigo e de ausência de atividade do worker; definir empacotamento com a Plataforma | Proposta; FDD-RISCO-08 |
| RISCO-08 | **SSRF:** a URL cadastrada pode apontar para rede interna; a reunião exigiu só `https` | Média | Alto | Revisão de segurança; bloquear loopback, IPs privados e link-local no cadastro e no envio; não seguir redirects | Não discutido na reunião; FDD-RISCO-03 |
| RISCO-09 | **Perda de ordering** se o worker for escalado, ou se um evento ficar em retry enquanto outros do mesmo pedido seguem | Baixa (escala); Média (retry) | Médio | Limitação documentada; payload com `timestamp`, `from_status` e `to_status` para o cliente reordenar; decidir a questão Q7 antes de escalar | `[09:13] Larissa`; RFC Q7; FDD-RISCO-07 |
| RISCO-10 | **Permissões amplas no CRUD:** qualquer role autenticada gerencia webhooks de qualquer customer | Média | Alto | Aceito "por enquanto"; revisar na revisão de segurança e endurecer em fase posterior | `[09:37] Sofia`; FDD-RISCO-04 |

## 11. Critérios de aceitação

A feature está pronta quando todos os critérios abaixo forem atendidos. A verificação técnica detalhada está nos FDD-AC-01 a FDD-AC-18 (FDD, seção 12).

| ID | Critério | Requisitos | Verificação |
| --- | --- | --- | --- |
| AC-01 | Dado um webhook ativo que escuta `SHIPPED`, quando um pedido do customer muda para `SHIPPED`, o evento chega ao endpoint em menos de 10 s, com os headers e o payload previstos | RF-05, RF-07, RF-16, RF-17, RNF-01 | Integração com servidor HTTP local e medição |
| AC-02 | Se a gravação na outbox falhar, a mudança de status, o histórico e o estoque permanecem como estavam | RF-05, RNF-02 | Integração: forçar a falha e conferir o rollback |
| AC-03 | Se nenhum webhook do customer escuta o novo status, nenhuma linha é inserida na outbox | RF-04 | Integração |
| AC-04 | O payload gravado não muda se o pedido for alterado depois | RF-06 | Integração |
| AC-05 | URL `http` é recusada no cadastro e na edição, com `WEBHOOK_INVALID_URL` | RF-18, RF-20 | Integração |
| AC-06 | O `X-Signature` é verificável por implementação independente de HMAC-SHA256; após a rotação valem as duas secrets por 24 h e depois só a nova | RF-12, RF-13 | Unitário com relógio injetável |
| AC-07 | Sem resposta em 10 s a tentativa é falha e reagendada; as falhas seguintes reagendam em 1 min, 5 min, 30 min, 2 h e 12 h | RF-08, RF-09 | Integração com servidor lento; unitário com relógio injetável |
| AC-08 | Esgotadas as tentativas, o evento sai da outbox e vai para a DLQ com payload, motivo e timestamp | RF-10 | Unitário |
| AC-09 | O replay retorna 403 para role não `ADMIN`; para `ADMIN`, o evento volta como pendente com o mesmo `event_id` e o log registra quem executou | RF-11, RNF-08, RNF-09 | Integração |
| AC-10 | O worker roda por `npm run worker`, em processo separado; reiniciar a API não o interrompe; interromper o worker durante um envio resulta em reenvio com o mesmo `X-Event-Id` | RF-07, RF-15, RNF-03 | Manual e integração |
| AC-11 | Corpo acima de 64 KB não é enviado e o evento vai para a DLQ com erro específico | RF-19, RNF-10 | Unitário |
| AC-12 | O cliente cadastra, edita, lista e remove webhooks; a secret só aparece na criação e na rotação, nunca em listagens, edições, histórico ou logs | RF-01, RF-03, RNF-06 | Integração e inspeção de logs |
| AC-13 | O histórico devolve no máximo 100 entregas, da mais recente para a mais antiga, com resultado, payload, resposta e tempo | RF-14 | Integração |
| AC-14 | Eventos do mesmo pedido, sem falhas, chegam na ordem em que ocorreram | RNF-05 | Integração |
| AC-15 | Os testes existentes de pedidos e autenticação continuam passando sem alteração de comportamento | OBJ-07, RNF-11 | Execução da suíte |
| AC-16 | A revisão de segurança (HMAC, geração e armazenamento de secret) foi concluída antes do deploy | RNF-13 | Parecer da Sofia |
| AC-17 | O portal de desenvolvedores documenta a integração, a verificação da assinatura e a deduplicação por `X-Event-Id` | RNF-04, RISCO-04 | Revisão por Produto |

## 12. Estratégia de testes e validação

A reunião definiu apenas que haverá **testes ponta a ponta da integração em `order.service.ts`** (`[09:46] Larissa`) e a **revisão de segurança** (`[09:46] Sofia`). Os demais níveis abaixo são Proposta, alinhada às ferramentas que o repositório já usa (Vitest e Supertest, em `tests/`).

| Nível | O que cobre | Ferramenta | Origem |
| --- | --- | --- | --- |
| **Unitário** | HMAC e geração de secret, tabela de backoff e transições da outbox, rotação com relógio injetável, limite de 64 KB, validação de URL | Vitest | Proposta (FDD seção 12) |
| **Integração (API)** | CRUD de webhooks, rotação de secret, histórico, replay com e sem `ADMIN`, códigos `WEBHOOK_*` | Vitest + Supertest, MySQL de teste | `tests/` existentes; Proposta |
| **Ponta a ponta** | `changeStatus` com publicação transacional (sucesso e rollback), worker entregando a um servidor HTTP local, timeout, retry, DLQ e restart do worker | Vitest com servidor HTTP local | `[09:46] Larissa`; cenários, Proposta |
| **Regressão** | Suíte atual de pedidos e autenticação sem alteração | Vitest | `tests/orders.test.ts`, `tests/auth.test.ts` |
| **Revisão de segurança** | HMAC, geração e armazenamento de secret, SSRF, formato do `X-Signature` com duas assinaturas e inclusão do `X-Timestamp` na assinatura (RFC Q10, Q11, Q13) | Revisão manual da Sofia, mínimo de 2 dias úteis | `[09:46] Sofia`, `[09:49] Sofia` |
| **Validação com clientes** | Integração real de ao menos um cliente (por exemplo, a Atlas) em ambiente de homologação, antes de liberar aos três | Acompanhamento de Produto | Proposta |
| **Validação pós-deploy** | Acompanhar a idade do evento mais antigo, a DLQ aberta e a atividade do worker | Consultas SQL e logs | Proposta (FDD seção 9) |

**Ambiente de teste.** O `tests/setup.ts` precisa conhecer as novas tabelas para a limpeza entre testes (FDD, seção 15, passo 1).

## 13. Questões em aberto

Decisões que a reunião não tomou e que afetam o produto. A lista completa, com revisor sugerido, está na seção 5 do [RFC](RFC.md).

| Questão | Por que importa para o produto | Quem decide |
| --- | --- | --- |
| **Criação de pedido gera evento?** (RFC Q8) O código grava `PENDING` na criação, e a reunião só tratou de "mudança de status" | Define se o cliente é avisado de pedidos novos | Marcos, Bruno |
| **Retry versus ordering** (RFC Q7) | Define se o cliente pode receber `SHIPPED` antes de um `PROCESSING` que ficou em retry | Diego |
| **Contagem das "5 tentativas"** (RFC Q9) | O FDD adota 1 envio inicial + 5 retries (até 14 h 36 min); precisa ser confirmado | Diego |
| **Meta de taxa de sucesso e alertas** | Não foi discutida; necessária para operar o SLA de menos de 10 s | Marcos, Diego |
| **Rate limiting e e-mail** | Fora desta fase; reavaliar após medir o uso | Larissa, Marcos |
