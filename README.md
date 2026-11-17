# Da Reunião ao Documento: Design Docs do Sistema de Webhooks de Notificação de Pedidos

Entrega do desafio "Design Docs Gerados por IA" (MBA Engenharia de Software com IA, Full Cycle). O enunciado original está em [DESAFIO.md](./DESAFIO.md).

## 1. Sobre o desafio

Uma empresa opera um Order Management System (OMS) em produção e decidiu, em uma reunião técnica de cerca de 55 minutos, construir um sistema de webhooks para avisar clientes B2B quando o status de um pedido muda. O único registro dessa decisão é a transcrição da call, em [TRANSCRICAO.md](./TRANSCRICAO.md).

O trabalho aqui foi transformar essa transcrição, junto com o código existente do OMS, em um pacote de design docs que o time de engenharia consiga usar para começar a implementar: PRD, RFC, FDD, ADRs e um Tracker de rastreabilidade. A entrega é só documental. Nenhum arquivo de `src/`, `prisma/` ou `tests/` foi alterado. A regra central foi não inventar: cada requisito, decisão ou restrição aponta para uma fala da reunião ou para um arquivo real do repositório.

## 2. Ferramentas de IA utilizadas

- **Claude Code**: leitura da transcrição e do código, extração e classificação dos itens da reunião, geração dos rascunhos de todos os documentos e verificação automática de citações e caminhos.
- **Documento de contexto para o agente**: um prompt consolidado, escrito antes da geração, com o enunciado, as regras de rastreabilidade, a fronteira entre os documentos e os templates de saída (baseados nos prompts de PRD e FDD do curso e nas regras do plugin `adrs-management`).
- **`CLAUDE.md`**: instruções fixas do repositório para o agente, com as restrições do desafio e os pontos de arquitetura do OMS.
- `[PREENCHER: outras ferramentas efetivamente usadas, como os plugins project-analizer ou adrs-management, e o papel de cada uma. Remover esta linha se não houver.]`

## 3. Workflow adotado

1. **Contextualização.** O agente leu a transcrição inteira e os arquivos de código que a feature toca: `order.service.ts`, `order.status.ts`, `schema.prisma`, erros, middlewares, logger, `app.ts`, `server.ts`, `routes/index.ts`, `env.ts` e os testes.
2. **Extração dirigida.** Cada item da reunião foi classificado como decisão fechada, requisito, restrição, descartado, adiado ou em aberto, sempre com a fala que fechou o assunto. Itens descartados e adiados foram separados para nunca virarem requisito.
3. **ADRs primeiro.** As seis decisões principais da reunião, mais a decisão de snapshot do payload, viraram sete ADRs.
4. **RFC.** Consolidou a proposta em nível de arquitetura, com alternativas descartadas e questões em aberto, apontando para os ADRs.
5. **FDD.** Detalhou fluxos, contratos, matriz de erros, observabilidade e a integração com o código existente.
6. **PRD.** Feito por último entre os grandes, consolidando a visão de produto sobre o que já estava decidido.
7. **Tracker.** Montado por varredura dos IDs escritos nos documentos.
8. **Verificação automática.** Um script conferiu cada `[hh:mm] Nome` contra a transcrição, cada caminho de arquivo contra o repositório, os links entre documentos e a cobertura do Tracker.
9. **Revisão humana.** `[PREENCHER: como foi a revisão, quantos ciclos de geração e revisão aconteceram e o que foi pedido em cada um.]`

Para evitar duplicação, cada documento ficou em uma altura: o PRD fala de produto, o RFC propõe e abre para revisão, os ADRs registram cada decisão fechada e o FDD detalha como construir. Onde um assunto aparece em mais de um lugar, o documento de nível mais alto resume em uma linha e aponta para o outro.

## 4. Prompts customizados

Prompt de extração e classificação da transcrição, rodado antes de escrever qualquer documento:

```text
Leia TRANSCRICAO.md inteira. Classifique cada item técnico ou de produto mencionado em
exatamente uma categoria:
DECISAO_FECHADA | REQUISITO_FUNCIONAL | REQUISITO_NAO_FUNCIONAL | RESTRICAO |
DESCARTADO | ADIADO | EM_ABERTO | GANCHO_COM_CODIGO | DETALHE_SECUNDARIO

Para cada item, devolva uma linha de tabela:
| categoria | resumo em 1 linha | [hh:mm] Nome (fala de origem) | trecho literal curto | observação |

Regras:
- Use o timestamp e o falante exatos da transcrição.
- Se um item foi proposto e depois descartado ou adiado, registre a fala do descarte/adiamento como origem.
- Se o item mudou ao longo da reunião, registre só o estado final e cite a fala que o fechou.
- Não infira nada que não esteja dito. Não complete números ausentes.
```

Prompt para isolar o que não pode virar requisito:

```text
Para cada item DESCARTADO ou ADIADO da tabela anterior, liste:
| item | descartado ou adiado | motivo/trade-off citado | [hh:mm] Nome |
Esses itens NUNCA podem aparecer como requisito. Indique em qual documento cada um deve
aparecer: PRD (Fora de escopo), RFC (Alternativas consideradas ou Questões em aberto),
FDD (Excluído) ou ADR (Alternativas consideradas).
```

Prompt para confirmar os ganchos com o código antes de citar qualquer arquivo:

```text
Para cada GANCHO_COM_CODIGO, abra o arquivo correspondente no repositório e confirme:
caminho real, nome do método/classe, comportamento atual relevante (ex.: transação, auditoria,
classes de erro). Devolva | gancho | caminho | símbolo confirmado | como a feature se integra |.
Se o arquivo ou símbolo não existir, marque NAO_ENCONTRADO e não use nos documentos.
```

Regra de rastreabilidade aplicada a todos os documentos:

```text
Todo requisito, decisão, restrição, métrica, risco ou contrato precisa ter origem identificável
na transcrição ([hh:mm] Nome) ou no código (caminho de arquivo real). Se não houver origem,
não escreva. Se for indispensável, marque explicitamente como [HIPÓTESE] e liste para revisão humana.
```

## 5. Iterações e ajustes

Número de iterações principais: `[PREENCHER: quantidade de ciclos de geração e revisão]`

Pontos em que a primeira geração precisou de atenção e que já estão sinalizados nos documentos. São candidatos naturais para os ciclos de revisão:

- **Contagem das tentativas de retry.** A reunião fala em "5 tentativas" e lista cinco intervalos (1m, 5m, 30m, 2h, 12h), com quase 15 horas "entre primeira falha e última tentativa". Isso admite duas leituras: cinco envios ao todo, ou um envio inicial mais cinco retentativas. A IA adotou a segunda como hipótese (S-01 no FDD) e registrou a dúvida como RFC-OQ-05, em vez de escolher em silêncio.
- **Código de erro da URL inválida.** A reunião diz que exigir `https` é "só uma validação no schema Zod", mas também pede o código `WEBHOOK_INVALID_URL`. No código atual, falha de schema Zod sai como `VALIDATION_ERROR`. As duas falas não fecham entre si, e o FDD (seção 6.1) deixa o conflito explícito para decisão.
- **`customer_id` no cadastro.** O PM disse primeiro que o `customer_id` viria implícito do JWT. A fala foi corrigida logo depois, porque o JWT é de usuário do OMS. Os documentos usam só o estado final, com o customer no body ou na rota.
- **Itens sem origem.** Rotas não ditas na reunião, nomes de duas tabelas, nomes de métricas e de spans, e alguns comportamentos (qual secret assina durante o grace period, o que acontece com eventos de endpoint removido) não têm fonte. Em vez de inventar, ficaram marcados como `[HIPÓTESE]` ou "em aberto" na seção 3.1 do FDD.

`[PREENCHER: pelo menos 2 momentos concretos da sua revisão em que a IA errou ou foi superficial, o que você pediu e como ficou depois. Exemplo de formato: "Na primeira versão do RFC, X estava detalhado demais. Pedi para mover para o FDD."]`

## 6. Como navegar a entrega

Ordem sugerida de leitura:

1. [docs/PRD.md](./docs/PRD.md): o problema, para quem, o que entra e o que fica fora, e como saber que deu certo.
2. [docs/RFC.md](./docs/RFC.md): a proposta técnica em nível de arquitetura, alternativas descartadas e questões em aberto.
3. [docs/adrs/](./docs/adrs/): uma decisão por arquivo.
   - [ADR-001](./docs/adrs/ADR-001-outbox-no-mysql.md): Outbox no MySQL
   - [ADR-002](./docs/adrs/ADR-002-worker-separado-com-polling.md): Worker separado com polling
   - [ADR-003](./docs/adrs/ADR-003-retry-com-backoff-e-dlq.md): Retry com backoff e DLQ
   - [ADR-004](./docs/adrs/ADR-004-at-least-once-com-x-event-id.md): At-least-once com `X-Event-Id`
   - [ADR-005](./docs/adrs/ADR-005-hmac-sha256-secret-por-endpoint.md): HMAC-SHA256 com secret por endpoint
   - [ADR-006](./docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md): Reuso dos padrões do projeto
   - [ADR-007](./docs/adrs/ADR-007-payload-snapshot-na-insercao.md): Payload como snapshot na inserção
4. [docs/FDD.md](./docs/FDD.md): fluxos, contratos, erros, observabilidade e integração com o código. A seção 3.1 lista as hipóteses e os pontos a confirmar.
5. [docs/TRACKER.md](./docs/TRACKER.md): de onde veio cada item, com fala da transcrição ou arquivo do repositório.

Material de apoio: [TRANSCRICAO.md](./TRANSCRICAO.md) (fonte, não alterada) e [DESAFIO.md](./DESAFIO.md) (enunciado).
