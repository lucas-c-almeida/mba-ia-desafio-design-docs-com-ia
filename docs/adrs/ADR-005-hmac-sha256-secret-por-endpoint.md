# ADR-005: Assinatura HMAC-SHA256 com secret por endpoint e rotação com grace period

## Status

Aceito

Data: 2026-11-17
ADRs relacionados: [ADR-004](./ADR-004-at-least-once-com-x-event-id.md), [ADR-006](./ADR-006-reuso-dos-padroes-do-projeto.md)

## Contexto

Os webhooks são de saída: saem do OMS para endpoints dos clientes, nunca o contrário ([09:02] Marcos, [09:03] Sofia). Isso significa expor eventos com dados de pedidos para fora da nossa infraestrutura. O cliente precisa conseguir validar que a requisição veio realmente do OMS e que ninguém adulterou o payload no caminho ([09:19] Sofia).

Já houve um caso de cliente que vazou secret em log da aplicação dele ([09:22] Diego). Qualquer esquema baseado em segredo compartilhado precisa limitar o estrago de um vazamento e permitir troca sem parar a integração.

### Direcionadores da decisão

- Autenticidade: o cliente confirma que o remetente é o OMS ([09:19] Sofia).
- Integridade: o payload não pode ser adulterado no caminho ([09:19] Sofia).
- Um vazamento de secret não pode comprometer todos os clientes ([09:21] Sofia).
- A troca de secret não pode derrubar a integração do cliente ([09:21] Sofia).
- O mecanismo precisa ser fácil de implementar do lado do cliente ([09:20] Sofia).

## Decisão

Opção escolhida: **assinatura HMAC-SHA256 sobre o corpo do request, com secret única por endpoint de webhook e rotação com grace period de 24 horas**, porque HMAC-SHA256 é o padrão de mercado, com biblioteca disponível para qualquer cliente, e a secret por endpoint contém o impacto de um vazamento ([09:20] Sofia, [09:21] Sofia, [09:22] Sofia).

A assinatura segue em um header do tipo `X-Signature`, e o cliente verifica do lado dele ([09:20] Sofia). A secret é gerada pelo OMS e devolvida na criação do endpoint ([09:31] Marcos). O cliente pode pedir uma nova secret pela API. Depois da rotação, a secret antiga continua válida por 24 horas em paralelo e então deixa de valer ([09:21] Sofia).

Como complemento, a URL do endpoint precisa ser `https`. Cadastro com `http` é recusado na validação. Isso foi tratado na reunião como validação de schema, não como decisão arquitetural separada ([09:23] Sofia).

## Alternativas Consideradas

### Opção 1: HMAC-SHA256 com secret por endpoint e rotação com grace period (escolhida)

- Prós: padrão de mercado, com suporte em bibliotecas dos clientes ([09:20] Sofia).
- Prós: cobre autenticidade e integridade do payload ([09:19] Sofia).
- Prós: um vazamento afeta apenas um endpoint ([09:21] Sofia).
- Prós: o grace period de 24 horas dá tempo para o cliente migrar ([09:21] Sofia).
- Contras: é preciso armazenar e gerenciar uma secret por endpoint.
- Contras: durante o grace period existem duas secrets válidas para o mesmo endpoint.

### Opção 2: Secret global da plataforma

- Prós: uma única secret para gerenciar.
- Contras: "se vaza uma, vaza tudo" ([09:21] Sofia).
- Contras: rotacionar exigiria coordenar todos os clientes ao mesmo tempo.
- Motivo do descarte: o raio de impacto de um vazamento é inaceitável ([09:21] Sofia).

## Consequências

**Positivas**
- Clientes conseguem rejeitar requisições forjadas ou adulteradas.
- A rotação vira operação de rotina, sem janela de indisponibilidade para o cliente.
- O incidente de vazamento em log de cliente já vivido passa a ter resposta clara: rotacionar aquele endpoint ([09:22] Diego).

**Negativas**
- O OMS passa a guardar secrets de clientes, o que exige cuidado com armazenamento e com logs.
- A geração de secret e o cálculo do HMAC são código sensível. A Segurança pediu pelo menos dois dias úteis de revisão antes do deploy ([09:46] Sofia).
- A lógica de rotação precisa controlar a validade da secret antiga por 24 horas.

**Trade-off explícito:** ganhamos autenticidade, integridade e contenção de vazamento. Aceitamos a complexidade de gerenciar secrets por endpoint e o custo de uma revisão de segurança dedicada.

## Referências

- Transcrição: [09:19] Sofia, [09:20] Sofia, [09:21] Sofia, [09:22] Sofia, [09:22] Diego, [09:23] Sofia, [09:46] Sofia
- Código: `src/shared/logger/index.ts` (lista de campos redigidos nos logs)
- Documentos: [RFC](../RFC.md), [FDD](../FDD.md)
