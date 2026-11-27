# Design Docs com IA: Sistema de Webhooks de Notificação de Pedidos

## Sobre o desafio

O desafio teve como objetivo gerar uma documentação técnica a partir da transcrição de uma reunião realizada entre a equipe técnica e também do código existente da aplicação para a construção de uma nova feature para um Sistema de Webhooks de Notificação de Pedidos. Os Design Docs foram gerados com o apoio de IA.

## Ferramentas de IA utilizadas

- **Claude**, a ferramenta principal. É a minha preferida pela precisão e por alucinar pouco. Mesmo assim, não confiei às cegas: o Tracker existe justamente para conferir cada item contra a transcrição e o código, e esse cruzamento chegou a apontar imprecisões (ver [Iterações e ajustes](#iterações-e-ajustes)).
- **GitHub Copilot**, usado para ter uma "segunda opinião".

## Workflow adotado

1. **Contexto e análise inicial.** Antes de gerar qualquer documento, passei à IA o contexto do desafio e pedi uma primeira análise do que estava sendo pedido, junto com uma análise do projeto: estrutura, padrões do código e o que a feature precisava endereçar.
2. **Um documento por vez, na ordem sugerida pelo desafio:** ADRs, RFC, FDD, PRD, Tracker e README. A ordem faz sentido porque cada documento se apoia nos anteriores: o PRD, por ser o mais alto nível, virou praticamente uma consolidação do RFC, do FDD e dos ADRs, e o Tracker só pôde ser feito com tudo pronto.
3. **Prompt dirigido para cada documento.** A cada pedido, informei como aquele documento deveria ser feito, qual o seu conteúdo e quais os seus critérios de aceite, em vez de pedir algo genérico.
4. **Revisão cruzada entre documentos.** Isso funcionou bem: durante a elaboração do FDD, por exemplo, a IA percebeu que precisava revisar e ajustar alguns ADRs e o RFC, e voltou a eles.
5. **Separação entre o que foi dito e o que foi proposto.** Os documentos distinguem o que veio da reunião (`[hh:mm] Nome`) ou do código (caminho de arquivo) do que foi **Proposta**, isto é, detalhe de implementação que a reunião não definiu. As propostas aparecem como questões em aberto no RFC (seção 5.2) e como propostas pendentes de validação no FDD (seção 14), e não como decisões.

## Prompts customizados

Dois dos prompts que mais pesaram no resultado. Em ambos, o que os torna dirigidos é: apontam os documentos que já existem como base, definem o formato esperado e trazem os critérios de aceite. Os trechos entre `[...]` foram omitidos aqui por serem texto do enunciado.

**1. PRD como consolidação dos documentos anteriores**

```text
Agora produza o PRD por último entre os grandes documentos. Como ele é mais
alto nível, com RFC, FDD e ADRs em mãos vira praticamente uma consolidação.

Produza o arquivo docs/PRD.md cobrindo a feature de Sistema de Webhooks de
Notificação de Pedidos. O PRD deve seguir o formato apresentado no curso e
incluir, no mínimo, as seguintes seções:

Resumo e contexto da feature
Problema e motivação
Público-alvo e cenários de uso
Objetivos e métricas de sucesso
Escopo (incluso e fora de escopo)
Requisitos funcionais
Requisitos não funcionais
Decisões e trade-offs principais
Dependências
Riscos e mitigação
Critérios de aceitação
Estratégia de testes e validação

A seção "Fora de escopo" deve listar explicitamente pelo menos 2 itens
descartados ou adiados durante a reunião.

Critérios de aceite:
☐ Arquivo existe e está em Markdown
☐ Contém todas as seções obrigatórias
☐ Identifica no mínimo 8 requisitos funcionais discutidos na reunião
☐ Inclui pelo menos 1 objetivo com métrica e meta quantitativa
☐ "Fora de escopo" lista pelo menos 2 itens explicitamente descartados ou
  adiados na reunião
☐ "Riscos" inclui pelo menos 2 riscos com probabilidade, impacto e mitigação
```

**2. Tracker de rastreabilidade contra alucinações**

```text
Perfeito, agora produza o Tracker: varre os documentos prontos

Produza o arquivo TRACKER.md, uma tabela markdown que mapeia cada item
registrado nos seus documentos à origem na transcrição ou no código. [...]
O tracker ajuda a manter a integridade da documentação contra alucinações
da IA.

Formato obrigatório da tabela:
ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização

Fonte: TRANSCRICAO ou CODIGO.
Localização: para TRANSCRICAO, timestamp + nome do falante (ex: [09:17] Diego).
Para CODIGO, caminho do arquivo (ex: src/modules/orders/order.service.ts).
Cobertura mínima: pelo menos 80% dos itens identificáveis nos seus documentos
devem ter linha correspondente no tracker.

Critérios de aceite:
☐ Arquivo existe e segue o formato de tabela definido
☐ Pelo menos 80% dos itens identificáveis dos documentos têm linha
☐ Pelo menos 70% das linhas têm Fonte = TRANSCRICAO com timestamp válido
  no formato [hh:mm] Nome
☐ Pelo menos 5 linhas têm Fonte = CODIGO com caminho de arquivo real
```

## Iterações e ajustes

O Claude trabalhou bem ao longo da sequência e, conforme avançamos para os documentos seguintes, ele mesmo voltou aos anteriores para corrigi-los. Foram doze rodadas principais de geração, uma por documento, e em pelo menos duas delas (FDD e Tracker) houve retorno a documentos já entregues. Os principais momentos:

- **Do ADR ao RFC: lacunas que a reunião não cobriu.** Ao escrever os ADRs e consolidar o RFC, apareceram pontos que a reunião não tratou e que a IA não deveria "decidir" sozinha, como a interação entre retry e ordering, se a criação de pedido gera evento, se o `X-Timestamp` entra na assinatura e como armazenar a secret. Em vez de assumir respostas, eles foram registrados como **questões em aberto** (RFC, seção 5.2, Q7 a Q14) e como "observação do redator" nos ADRs.
- **FDD: ajustes nos ADRs e no RFC.** Ao detalhar a implementação e cruzá-la com o código, a IA percebeu que precisava ajustar ADRs e RFC. Um exemplo documentado é o `NotFoundError`, que fixa o código `NOT_FOUND` e por isso não serve para emitir `WEBHOOK_NOT_FOUND`; foi preciso prever uma subclasse de `AppError` (ADR-006 e FDD, seção 7).
- **PRD: não inventar metas.** A reunião só define metas quantitativas para latência (menos de 10 s), janela de retry (5 retentativas em 14h36) e prazo (3 sprints). Para o que a reunião não definiu, como taxa de sucesso, o PRD marca **Proposta** ou deixa como pendência, em vez de fabricar um número.
- **Tracker: dois achados.** (1) A primeira versão cobria os itens, mas só cerca de 69,6% das linhas vinham da transcrição, abaixo do mínimo de 70%. Em vez de reclassificar propostas como se fossem falas da reunião, acrescentei linhas de itens que realmente têm origem na transcrição (contexto dos ADRs, público-alvo, cenários e componentes do RFC), chegando a 72%. (2) O cruzamento com o código mostrou que o FDD afirma que `OrderService.delete` apaga pedidos `CANCELLED`, mas o código (`src/modules/orders/order.service.ts`) também permite `PENDING`. A conclusão do FDD (sem FK de `orderId`) continua válida, mas a frase está imprecisa.
- **Itens sem origem na reunião ficam rotulados.** O Tracker usa um terceiro valor de Fonte, `PROPOSTA`, para os itens que não vieram da transcrição nem do código. Isso evita que uma sugestão da IA passe por decisão da reunião.

## Como navegar a entrega

### Ordem sugerida de leitura

| # | Arquivo | Pergunta que responde | Quando ler |
| --- | --- | --- | --- |
| 1 | [`TRANSCRICAO.md`](TRANSCRICAO.md) | O que foi realmente dito na reunião? | Para ter a fonte primária em mente |
| 2 | [`docs/PRD.md`](docs/PRD.md) | Por que e o quê? | Primeiro documento: problema, escopo, requisitos e métricas |
| 3 | [`docs/RFC.md`](docs/RFC.md) | Como pretendemos resolver, e o que está em aberto? | Proposta técnica, alternativas e questões para revisão |
| 4 | [`docs/adrs/`](docs/adrs/README.md) | Por que decidimos exatamente assim? | Uma decisão por arquivo, com contexto e consequências |
| 5 | [`docs/FDD.md`](docs/FDD.md) | Como construir, em detalhe? | Modelo de dados, fluxos, contratos, erros e integração com o código |
| 6 | [`docs/TRACKER.md`](docs/TRACKER.md) | De onde veio cada coisa? | Para conferir a origem de qualquer item |

Quem tem pouco tempo pode ler só o PRD e o RFC; quem vai implementar deve ir direto ao FDD, consultando os ADRs para entender o motivo de cada decisão.

### Convenções dos documentos

- `[hh:mm] Nome` indica uma fala da reunião; um caminho de arquivo indica um fato do código.
- **Proposta** marca o que a reunião não definiu e foi preenchido na redação. É sugestão a validar, não decisão.
- Caminhos marcados como *(a criar)* são arquivos novos da feature; os demais já existem no repositório.
- Os documentos se referenciam por ID (`PRD-RF-01`, `FDD-CONTRATO-03`, `ADR-002`, `RFC Q7`), todos listados no Tracker.
