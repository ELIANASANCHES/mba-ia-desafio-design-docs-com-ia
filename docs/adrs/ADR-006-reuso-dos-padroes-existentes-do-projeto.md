# ADR-006: Reuso máximo dos padrões existentes do projeto

## Status

Aceito — decidido na reunião técnica de webhooks (`[09:30] Larissa`). Registrado em 2026-11-24.

**Decisores:** Larissa, Bruno, Diego.

## Contexto

O OMS tem convenções consolidadas e a feature deve se encaixar nelas, sem introduzir novas bibliotecas ou estilos (`[09:29] Bruno`, `[09:30] Larissa`). Esta decisão ancora o módulo de webhooks no código existente.

## Decisão

O módulo de webhooks reutiliza o que já existe:

1. **Estrutura modular.** Nova pasta `src/modules/webhooks/` (nova, a criar) com controller, service, repository, routes e schemas, como `src/modules/orders/` e `src/modules/customers/` (`[09:27] Bruno`). O worker fica em `src/worker.ts` (novo, a criar) com a lógica no módulo (`[09:28] Bruno`; ver [ADR-002](ADR-002-worker-separado-com-polling-de-2s.md)).
2. **Erros.** Classes que estendem `AppError` (`src/shared/errors/app-error.ts`), com códigos no padrão existente (`INSUFFICIENT_STOCK`, `INVALID_STATUS_TRANSITION`), agora com prefixo **`WEBHOOK_`**: `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED`, "etc." (`[09:28] Bruno`, `[09:29] Larissa`).
3. **Error middleware centralizado** (`src/middlewares/error.middleware.ts`): já trata `AppError`, Zod e Prisma e pega os novos erros sem alteração (`[09:29] Bruno`).
4. **Logger Pino** (`src/shared/logger/index.ts`), sem nova dependência (`[09:29] Bruno`).
5. **Schemas Zod** e o middleware `validate` (`src/middlewares/validate.middleware.ts`) (`[09:30] Larissa`; `[09:23] Sofia` para a validação `https`).
6. **Autorização.** `authenticate` e `requireRole` de `src/middlewares/auth.middleware.ts`: o replay da DLQ exige `ADMIN` e o resto do CRUD aceita qualquer role autenticada, "por enquanto" (`[09:36] Larissa`, `[09:37] Sofia`). O `customer_id` vem no body ou no path, **não** do JWT, pois o JWT identifica um usuário operador (`[09:32] Bruno`, `[09:32] Larissa`).
7. **IDs UUID** e `PrismaClient`: tabelas novas seguem o padrão de `prisma/schema.prisma` (`[09:51] Larissa`); o worker abre um `PrismaClient` próprio sobre a mesma `DATABASE_URL` (`[09:30] Bruno`).

## Alternativas Consideradas

1. **Novos frameworks de fila e novas bibliotecas de logging ou erros.** Rejeitado implicitamente: a decisão é "reuso máximo" e "não vamos botar nada novo" (`[09:29] Bruno`, `[09:30] Larissa`).
2. **Módulo fora de `src/modules`, como serviço isolado.** Plausível, não discutida; contraria o padrão e a decisão de "módulo igual aos outros" (`[09:30] Larissa`).

## Consequências

**Positivas**

- Curva de aprendizado e custo de revisão mínimos: o time reconhece o formato.
- O error middleware, a validação e a autorização já testados são herdados.
- Respostas de erro consistentes com o restante da API (`{ error: { code, message, details } }`).

**Negativas / trade-offs**

- Herdamos limitações atuais. Exemplo concreto: `NotFoundError` (`src/shared/errors/http-errors.ts`) fixa o código `NOT_FOUND` no construtor e não aceita código customizado; para emitir `WEBHOOK_NOT_FOUND`, será preciso uma subclasse de `AppError` com status 404, e não o simples reuso de `NotFoundError`. Já `ConflictError`, `BadRequestError` e `UnprocessableEntityError` aceitam `code`.
- A fiação continua manual em `buildControllers` (`src/app.ts`) e no tipo `Controllers` (`src/routes/index.ts`): o módulo exige editar esses dois arquivos existentes.
- O logger precisa de ajuste de redact para `secret` (ver [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md)).

## Referências

- Transcrição: `[09:27]`–`[09:30]`, `[09:32]`, `[09:36]`–`[09:37]`, `[09:51] Larissa`.
- Código: `src/modules/orders/`, `src/modules/customers/`, `src/shared/errors/app-error.ts`, `src/shared/errors/http-errors.ts`, `src/shared/errors/index.ts`, `src/middlewares/error.middleware.ts`, `src/middlewares/auth.middleware.ts`, `src/middlewares/validate.middleware.ts`, `src/shared/logger/index.ts`, `src/app.ts`, `src/routes/index.ts`, `prisma/schema.prisma`.
