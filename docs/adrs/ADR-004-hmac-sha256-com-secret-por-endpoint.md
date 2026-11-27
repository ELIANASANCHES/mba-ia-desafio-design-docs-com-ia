# ADR-004: Autenticação HMAC-SHA256 com secret por endpoint e rotação com grace period de 24h

## Status

Aceito — decidido na reunião técnica de webhooks (`[09:22] Sofia`). Registrado em 2026-11-24.

**Decisores:** Sofia (Segurança), com concordância de Larissa, Diego e Bruno. A revisão de segurança do código (HMAC e geração de secret) está reservada em pelo menos dois dias úteis antes do deploy (`[09:46] Sofia`).

## Contexto

Os eventos carregam dados de pedidos e saem para endpoints fora da nossa infraestrutura. O cliente precisa validar que a requisição veio de nós e que o payload não foi adulterado (`[09:19] Sofia`). Já houve cliente que vazou uma secret em log da própria aplicação (`[09:22] Diego`).

## Decisão

- **HMAC-SHA256 sobre o corpo do request**, enviado no header `X-Signature`; o cliente verifica do seu lado (`[09:20] Sofia`, `[09:22] Sofia`).
- **Uma secret única por endpoint de webhook**, e não uma secret global da plataforma (`[09:21] Sofia`). A tabela de configuração guarda URL, secret, `customer_id` e estado ativo (`[09:21] Bruno`, `[09:21] Sofia`).
- **A secret é gerada por nós** e devolvida na criação do webhook (`[09:31] Marcos`).
- **Rotação pela API**: o cliente pede uma nova secret; a antiga continua válida por **24 horas** em paralelo e depois deixa de valer (`[09:21] Sofia`).

Itens correlatos que **não** são decisão arquitetural separada e ficam no FDD: validação de URL `https` no schema Zod (`[09:23] Sofia`) e limite de 64KB de payload (`[09:24] Diego`, `[09:24] Larissa`).

## Alternativas Consideradas

1. **Secret global da plataforma.** Descartada: "se vaza uma, vaza tudo" (`[09:21] Sofia`).
2. **Enviar sem assinatura, confiando só em TLS.** Plausível, não discutida como opção formal; incompatível com o requisito de o cliente validar origem e integridade (`[09:19] Sofia`).
3. **Rotação sem período de convivência.** Descartada implicitamente: o grace period de 24h existe para o cliente migrar seus sistemas (`[09:21] Sofia`).

## Consequências

**Positivas**

- Vazamento de uma secret afeta um único endpoint.
- Padrão de mercado, com bibliotecas disponíveis nos clientes (`[09:20] Sofia`).
- A rotação com grace period evita janela de indisponibilidade para o cliente.

**Negativas / trade-offs**

- Durante o grace period, o sistema precisa manter e aceitar **duas secrets ativas** por endpoint (não detalhado na reunião; vai para o FDD).
- A secret precisa ser armazenada de forma que o servidor consiga reassinar eventos; o modelo de armazenamento não foi definido na reunião e fica para a revisão de segurança.
- O logger atual (`src/shared/logger/index.ts`) só faz redact de `password`, `passwordHash`, `token`, `accessToken` e dos headers `authorization`/`cookie`; **não cobre `secret`**. Como a reunião citou vazamento em log, o FDD deve tratar esse ponto.
- `X-Timestamp` é enviado para o cliente detectar replay (`[09:44] Diego`), mas a reunião não definiu se o timestamp entra na assinatura; fica como questão em aberto.
- Prazo: reserva de dois dias úteis para revisão de segurança antes do deploy (`[09:46] Sofia`).

## Referências

- Transcrição: `[09:19]`–`[09:23]`, `[09:31] Marcos`, `[09:44] Diego`, `[09:46] Sofia`.
- Código: `src/shared/logger/index.ts` (lista `redactPaths`), `src/middlewares/validate.middleware.ts` e padrão de schemas Zod por módulo.
