### PRD: OMS Sistema de Webhooks de Notificação de Pedidos

Versão: 0.1 (rascunho para revisão)
Data: 2026-11-17
Responsável: Marcos (Product Manager)

Documentos relacionados: [RFC](./RFC.md), [FDD](./FDD.md), [ADRs](./adrs/), [Tracker](./TRACKER.md)

---

### Resumo e contexto da feature

Clientes B2B do OMS querem saber na hora quando o status de um pedido deles muda. Hoje a única forma é consultar a API repetidamente. Esta feature faz o OMS avisar o cliente: a cada mudança de status, o OMS envia uma requisição HTTP (webhook) para um endereço cadastrado pelo cliente ([09:00] Marcos).

O fluxo é só de saída. O OMS envia e o cliente recebe ([09:02] Marcos).

Onde essa feature será implantada
- No OMS existente: API Node.js com TypeScript, Express, Prisma e MySQL 8 (`package.json`, `prisma/schema.prisma`).
- A mudança de status que dispara a notificação já existe, com máquina de estados em `src/modules/orders/order.status.ts`.
- A entrega é feita por API. Não há tela nesta fase ([09:40] Larissa).

---

### Problema e motivação

Problemas priorizados
- **Integração lenta e cara para o cliente (prioridade alta).** Os clientes ficam consultando `GET /orders` de tempos em tempos para descobrir se algo mudou ([09:00] Marcos).
- **Risco de perda de cliente (prioridade alta).** Três clientes B2B fizeram pedido formal: Atlas Comercial, MaxDistribuição e Nova Cargo. A Atlas sinalizou que pode migrar para o concorrente se a entrega não sair até o fim do trimestre ([09:00] Marcos).
- **Informação atrasada (prioridade média).** Os clientes não querem que a mudança "fique pendurada" nem ter que atualizar manualmente ([09:02] Marcos).

---

### Público-alvo e cenários de uso

Público-alvo
- Clientes B2B que integram seus sistemas com o OMS, começando por Atlas Comercial, MaxDistribuição e Nova Cargo ([09:00] Marcos).
- Usuários do OMS que representam o cliente e configuram os webhooks pela API ([09:32] Marcos).
- Administradores do OMS, que reprocessam entregas com falha permanente ([09:36] Sofia).

Cenários de uso chave
- O cliente cadastra um endereço e passa a receber um aviso sempre que um pedido dele muda de status ([09:31] Marcos).
- O cliente escolhe receber só alguns status, por exemplo apenas `SHIPPED` e `DELIVERED` ([09:33] Marcos).
- O sistema do cliente fica fora do ar em uma manutenção planejada e, quando volta, recebe os avisos que ficaram pendentes ([09:16] Diego).
- O cliente consulta as últimas entregas para investigar um problema de integração ([09:34] Marcos).
- O cliente troca a secret depois de um vazamento, sem parar a integração ([09:21] Sofia, [09:22] Diego).

---

### Objetivos e métricas de sucesso

| ID | Objetivo | Métrica | Meta |
|---|---|---|---|
| PRD-OBJ-01 | Notificar o cliente em "tempo real" | Tempo entre a mudança de status e o envio do webhook | Abaixo de 10 segundos ([09:02] Marcos) |
| PRD-OBJ-02 | Atender os clientes que fizeram o pedido formal | Clientes solicitantes com webhook em uso | Os 3 clientes: Atlas Comercial, MaxDistribuição e Nova Cargo ([09:00] Marcos) |
| PRD-OBJ-03 | Entregar dentro do prazo combinado com a Atlas | Prazo de entrega | 3 sprints, com a revisão de segurança incluída ([09:47] Larissa). A Atlas pediu para o fim de novembro ([09:45] Marcos) |
| PRD-OBJ-04 | Tolerar indisponibilidade do cliente sem perder aviso | Janela coberta por retentativas automáticas | Quase 15 horas entre a primeira falha e a última tentativa ([09:17] Diego) |

---

### Escopo

Incluso
- Notificação por webhook a cada mudança de status de pedido ([09:00] Marcos).
- Cadastro, edição, remoção e listagem de endpoints de webhook por customer ([09:31] Marcos, [09:33] Bruno).
- Escolha dos status que cada endpoint recebe ([09:33] Marcos).
- Assinatura das requisições e rotação de secret ([09:22] Sofia).
- Retentativa automática e fila de falhas permanentes com reprocessamento manual ([09:17] Larissa, [09:19] Larissa).
- Histórico de entregas por endpoint ([09:34] Marcos).
- Documentação de integração no portal de desenvolvedor ([09:40] Marcos).

Fora de escopo
- **PRD-OOS-01 Aviso por email quando o webhook do cliente falha.** Adiado para uma próxima fase, depois de medir o impacto ([09:37] Larissa).
- **PRD-OOS-02 Dashboard visual para o cliente.** Nesta fase só existem endpoints. Painel é projeto separado do time de frontend ([09:40] Larissa).
- **PRD-OOS-03 Rate limiting de envio para o cliente.** Fica como "observar e decidir depois" ([09:39] Larissa).
- **PRD-OOS-04 Webhooks de entrada.** O cliente não envia nada para o OMS ([09:02] Marcos).
- **PRD-OOS-05 Garantia de ordem global dos eventos.** A ordem vale por pedido e enquanto houver um único worker. Os clientes nunca pediram ordem global ([09:13] Larissa, [09:14] Marcos).
- **PRD-OOS-06 Garantia de entrega exatamente uma vez.** Descartada pela complexidade ([09:25] Diego).
- **PRD-OOS-07 Itens do pedido no payload.** O aviso é enxuto, e o cliente consulta o pedido se quiser detalhes ([09:43] Diego).
- **PRD-OOS-08 Arquivamento de eventos antigos.** Citado como necessário depois, fora do escopo desta feature ([09:08] Diego).
- **PRD-OOS-09 Permissões mais restritas para configurar webhooks.** Pode ser endurecido mais à frente ([09:37] Sofia).

---

### Requisitos funcionais

#### PRD-FR-01 Notificar mudança de status de pedido
O OMS envia um webhook para o cliente sempre que o status de um pedido dele muda ([09:00] Marcos).

**Fluxo principal**
- O status de um pedido muda no OMS.
- O OMS registra o evento e envia a notificação para os endpoints do customer que querem aquele status.
- O cliente recebe os dados básicos da mudança: pedido, status anterior, status novo e total ([09:43] Diego).

**Fluxos alternativos e exceções**
- O customer não tem endpoint cadastrado para aquele status: nada é enviado ([09:34] Bruno).

**Erros previstos**
- Endpoint do cliente indisponível: ver PRD-FR-10.

**Prioridade:** alta

---

#### PRD-FR-02 Cadastrar endpoint de webhook
Um usuário autenticado cadastra um endpoint para um customer, informando a URL e a lista de status de interesse ([09:31] Marcos).

**Fluxo principal**
- O usuário envia a URL, os status e o customer.
- O OMS gera a secret e a devolve na resposta da criação ([09:31] Marcos).

**Fluxos alternativos e exceções**
- O customer é informado na requisição. Ele não vem do JWT, que é de usuário do OMS ([09:32] Bruno, [09:32] Larissa).

**Erros previstos**
- URL que não é `https`: cadastro recusado ([09:23] Sofia).
- Usuário não autenticado.

**Prioridade:** alta

---

#### PRD-FR-03 Editar endpoint de webhook
Um usuário autenticado altera um endpoint existente ([09:33] Bruno).

**Fluxo principal**
- O usuário envia os campos a alterar e recebe o endpoint atualizado.

**Fluxos alternativos e exceções**
- Nova URL passa pela mesma regra de `https` ([09:23] Sofia).

**Erros previstos**
- Endpoint inexistente.

**Prioridade:** alta

---

#### PRD-FR-04 Remover endpoint de webhook
Um usuário autenticado remove um endpoint ([09:33] Bruno).

**Fluxo principal**
- O usuário pede a remoção e o endpoint deixa de receber notificações.

**Fluxos alternativos e exceções**
- Nenhum discutido na reunião.

**Erros previstos**
- Endpoint inexistente.

**Prioridade:** alta

---

#### PRD-FR-05 Listar endpoints de um customer
Um usuário autenticado lista os webhooks cadastrados de um customer ([09:33] Bruno).

**Fluxo principal**
- O usuário informa o customer e recebe a lista de endpoints.

**Fluxos alternativos e exceções**
- Customer sem endpoint: lista vazia.

**Erros previstos**
- Usuário não autenticado.

**Prioridade:** média

---

#### PRD-FR-06 Escolher quais status cada endpoint recebe
Cada endpoint tem uma lista dos status que quer ouvir ([09:33] Bruno, [09:33] Marcos).

**Fluxo principal**
- O cliente define, por exemplo, que só quer `SHIPPED` e `DELIVERED`.
- Mudanças para outros status não geram notificação para aquele endpoint.

**Fluxos alternativos e exceções**
- Se nenhum endpoint do customer quer o status, o evento nem é registrado ([09:34] Bruno).

**Erros previstos**
- Status inválido na lista.

**Prioridade:** alta

---

#### PRD-FR-07 Rotacionar a secret do endpoint
O cliente consegue pedir uma nova secret pela API ([09:21] Sofia).

**Fluxo principal**
- O usuário pede a rotação e recebe a nova secret.
- A secret antiga continua válida por 24 horas, para o cliente migrar os sistemas dele ([09:21] Sofia).

**Fluxos alternativos e exceções**
- Depois de 24 horas, a secret antiga deixa de valer ([09:21] Sofia).

**Erros previstos**
- Endpoint inexistente.

**Prioridade:** alta

---

#### PRD-FR-08 Consultar histórico de entregas
O cliente vê as últimas entregas feitas para um endpoint ([09:34] Marcos).

**Fluxo principal**
- O usuário consulta o endpoint e recebe as últimas 100 entregas, com sucesso ou falha, payload, response e tempo de resposta ([09:34] Marcos).

**Fluxos alternativos e exceções**
- Endpoint sem entregas: lista vazia.

**Erros previstos**
- Endpoint inexistente.

**Prioridade:** média

---

#### PRD-FR-09 Permitir que o cliente valide a origem da notificação
Toda notificação leva uma assinatura que o cliente confere para saber que ela veio do OMS e não foi adulterada ([09:19] Sofia).

**Fluxo principal**
- O OMS assina o corpo da requisição com a secret do endpoint e envia a assinatura em um header ([09:20] Sofia).
- O cliente refaz o cálculo e compara.

**Fluxos alternativos e exceções**
- Durante a rotação, a secret antiga ainda é aceita por 24 horas ([09:21] Sofia).

**Erros previstos**
- Não se aplica do lado do OMS. A validação é do cliente.

**Prioridade:** alta

---

#### PRD-FR-10 Tentar de novo quando a entrega falha
Se o cliente estiver fora do ar ou lento, o OMS tenta de novo automaticamente ([09:14] Larissa, [09:15] Diego).

**Fluxo principal**
- A entrega falha ou o cliente não responde em 10 segundos ([09:42] Diego).
- O OMS faz 5 tentativas, com intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas ([09:17] Larissa).

**Fluxos alternativos e exceções**
- Todas as tentativas falham: ver PRD-FR-11.

**Erros previstos**
- Cliente indisponível por mais tempo que a janela de tentativas.

**Prioridade:** alta

---

#### PRD-FR-11 Guardar falhas permanentes e permitir reprocessamento
Notificações que esgotam as tentativas ficam guardadas e podem ser reenviadas por um administrador ([09:18] Diego).

**Fluxo principal**
- Esgotadas as tentativas, o evento vai para a DLQ com o payload e o motivo da falha ([09:18] Diego).
- Um usuário com role `ADMIN` pede o replay e o evento volta para a fila de envio ([09:18] Diego, [09:36] Sofia).
- O sistema registra quem fez o replay ([09:36] Sofia).

**Fluxos alternativos e exceções**
- Nenhum discutido na reunião.

**Erros previstos**
- Usuário sem role `ADMIN`: acesso negado ([09:36] Larissa).

**Prioridade:** alta

---

#### PRD-FR-12 Identificar cada evento para o cliente evitar duplicidade
Cada notificação leva um identificador único do evento, para o cliente reconhecer quando recebe a mesma mais de uma vez ([09:25] Diego).

**Fluxo principal**
- O OMS envia o identificador em um header a cada entrega.
- Se o mesmo evento chegar duas vezes, o cliente descarta a repetição pelo identificador ([09:25] Diego).

**Fluxos alternativos e exceções**
- O cliente com vários endpoints recebe também o identificador do endpoint que originou o envio ([09:44] Sofia).

**Erros previstos**
- Não se aplica.

**Prioridade:** alta

---

### Requisitos não funcionais

Performance
- **PRD-NFR-01** A notificação sai em menos de 10 segundos após a mudança de status ([09:02] Marcos). A verificação de eventos novos a cada 2 segundos atende esse limite ([09:10] Marcos).
- **PRD-NFR-02** A chamada ao cliente tem timeout de 10 segundos ([09:42] Diego).
- **PRD-NFR-03** O payload tem limite de 64KB. Acima disso o evento gera erro e não é enviado ([09:24] Larissa).

Disponibilidade
- **PRD-NFR-04** Cliente lento ou fora do ar não pode travar nem desfazer a mudança de status de pedidos ([09:04] Bruno).
- **PRD-NFR-05** O processamento das notificações não pode parar quando a API reinicia ([09:11] Diego).

Segurança e autorização
- **PRD-NFR-06** Só são aceitas URLs `https` ([09:23] Sofia).
- **PRD-NFR-07** Cada endpoint tem secret própria. Não existe secret global ([09:21] Sofia).
- **PRD-NFR-08** A configuração de webhooks exige usuário autenticado, de qualquer role. O replay exige role `ADMIN` ([09:36] Larissa, [09:37] Sofia).
- **PRD-NFR-09** O código de assinatura e de geração de secret passa por revisão de segurança antes do deploy ([09:46] Sofia).

Observabilidade
- **PRD-NFR-10** O replay registra em log quem o executou, para auditoria ([09:36] Sofia).
- **PRD-NFR-11** Os logs usam o logger já existente no projeto ([09:29] Bruno).

Confiabilidade e integridade de dados
- **PRD-NFR-12** Não pode existir mudança de status sem o evento correspondente registrado ([09:40] Bruno).
- **PRD-NFR-13** A entrega é at-least-once. O mesmo evento pode chegar mais de uma vez ([09:24] Diego).
- **PRD-NFR-14** Os eventos de um mesmo pedido chegam em ordem, enquanto houver um único worker ([09:13] Larissa).
- **PRD-NFR-15** O evento reflete o pedido como estava no momento da mudança de status ([09:52] Larissa).

Compatibilidade e portabilidade
- **PRD-NFR-16** A feature usa o MySQL e a stack atuais, sem infraestrutura nova ([09:07] Diego).
- **PRD-NFR-17** O módulo segue os padrões de código do projeto ([09:30] Larissa).

---

### Decisões e trade-offs principais

#### Decisão: registrar o evento junto com a mudança de status
- **Justificativa:** garante que nenhuma mudança de status fique sem notificação registrada.
- **Trade-off:** uma escrita a mais em uma operação crítica.
- **Registro:** [ADR-001](./adrs/ADR-001-outbox-no-mysql.md)

#### Decisão: entregar por um processo separado, que verifica a cada 2 segundos
- **Justificativa:** atende o limite de 10 segundos e não depende da API estar no ar.
- **Trade-off:** até 2 segundos de espera e ordem garantida só por pedido.
- **Registro:** [ADR-002](./adrs/ADR-002-worker-separado-com-polling.md)

#### Decisão: 5 tentativas em quase 15 horas, depois fila de falhas
- **Justificativa:** cobre manutenções de horas do cliente sem deixar evento preso para sempre.
- **Trade-off:** depois disso, depende de ação manual de um administrador.
- **Registro:** [ADR-003](./adrs/ADR-003-retry-com-backoff-e-dlq.md)

#### Decisão: o cliente pode receber o mesmo evento mais de uma vez
- **Justificativa:** é o padrão de mercado e muito mais simples que garantir entrega única.
- **Trade-off:** o cliente precisa tratar duplicidade pelo identificador do evento.
- **Registro:** [ADR-004](./adrs/ADR-004-at-least-once-com-x-event-id.md)

#### Decisão: assinatura com secret por endpoint
- **Justificativa:** o cliente valida a origem, e um vazamento afeta só um endpoint.
- **Trade-off:** mais secrets para gerenciar e uma revisão de segurança dedicada.
- **Registro:** [ADR-005](./adrs/ADR-005-hmac-sha256-secret-por-endpoint.md)

#### Decisão: seguir os padrões que o projeto já usa
- **Justificativa:** entrega mais rápida e código consistente para um time pequeno.
- **Trade-off:** a feature se adapta aos padrões atuais.
- **Registro:** [ADR-006](./adrs/ADR-006-reuso-dos-padroes-do-projeto.md)

#### Decisão: o conteúdo do evento é fixado na hora da mudança
- **Justificativa:** o aviso mostra o pedido como estava quando o status mudou.
- **Trade-off:** mais dados guardados por evento.
- **Registro:** [ADR-007](./adrs/ADR-007-payload-snapshot-na-insercao.md)

---

### Dependências

#### Segurança: revisão antes do deploy
A Segurança precisa de pelo menos dois dias úteis para revisar o código de HMAC e de geração de secret antes de a feature subir. A revisão precisa ser agendada ([09:46] Sofia, [09:49] Sofia).

#### Produto: documentação no portal de desenvolvedor
O PM documenta a integração por API e destaca que o cliente pode receber eventos repetidos e deve tratar isso pelo identificador ([09:26] Marcos, [09:40] Marcos).

#### Produto: confirmação de prazo com os clientes
O PM confirma o prazo com a Atlas e atualiza os demais clientes ([09:47] Marcos, [09:49] Marcos).

#### Engenharia: revisão do design antes de codar
A Tech Lead abre o doc de design e revisa com Bruno e Diego antes do início da implementação ([09:50] Larissa).

#### Clientes: adaptação do lado deles
Cada cliente precisa expor um endpoint `https`, validar a assinatura e tratar eventos duplicados ([09:23] Sofia, [09:24] Diego).

---

### Riscos e mitigação

#### PRD-RISK-01 A entrega atrasa e a Atlas migra para o concorrente
- **Probabilidade:** média
- **Impacto:** alto. Perda de um cliente B2B que já sinalizou a saída ([09:00] Marcos).
- **Mitigação:**
  - Escopo enxuto, com email, dashboard e rate limiting fora desta fase ([09:37] Larissa, [09:39] Larissa, [09:40] Larissa).
  - Estimativa de 3 sprints já com a revisão de segurança ([09:47] Larissa).
  - Confirmação de prazo com a Atlas ([09:47] Marcos).
- **Plano de contingência:** `[HIPÓTESE]` renegociar o prazo com a Atlas com base no andamento das sprints. A reunião não definiu plano B.

#### PRD-RISK-02 Cliente fica indisponível por mais tempo que a janela de tentativas
- **Probabilidade:** baixa
- **Impacto:** médio. As notificações desse período não chegam sozinhas ([09:17] Marcos).
- **Mitigação:**
  - Janela de quase 15 horas de tentativas ([09:17] Diego).
  - Eventos guardados na DLQ, sem perda ([09:18] Diego).
- **Plano de contingência:** replay manual por um administrador ([09:18] Diego).

#### PRD-RISK-03 Vazamento da secret de um cliente
- **Probabilidade:** média. Já houve cliente que vazou secret em log ([09:22] Diego).
- **Impacto:** alto. Alguém poderia forjar notificações para aquele endpoint.
- **Mitigação:**
  - Secret única por endpoint ([09:21] Sofia).
  - Revisão de segurança antes do deploy ([09:46] Sofia).
- **Plano de contingência:** rotação da secret, com 24 horas para o cliente migrar ([09:21] Sofia).

#### PRD-RISK-04 Cliente não trata eventos duplicados
- **Probabilidade:** média
- **Impacto:** médio. O cliente pode processar a mesma mudança duas vezes ([09:25] Sofia).
- **Mitigação:**
  - Identificador único em cada evento ([09:25] Diego).
  - Documentação destacada no portal de desenvolvedor ([09:26] Marcos).
- **Plano de contingência:** `[HIPÓTESE]` suporte direto ao cliente na integração. A reunião não definiu plano B.

#### PRD-RISK-05 Cliente recebe rajadas de notificações
- **Probabilidade:** média
- **Impacto:** médio. Um cliente com 50 pedidos mudando em um minuto recebe 50 chamadas ([09:38] Diego).
- **Mitigação:**
  - Observar o comportamento em produção ([09:39] Diego).
- **Plano de contingência:** implementar rate limiting se virar problema ([09:39] Diego).

---

### Critérios de aceitação

- [ ] **PRD-AC-01** Um cliente com endpoint cadastrado recebe a notificação em menos de 10 segundos após a mudança de status ([09:02] Marcos).
- [ ] **PRD-AC-02** O cliente consegue cadastrar, editar, remover e listar endpoints pela API ([09:31] Marcos, [09:33] Bruno).
- [ ] **PRD-AC-03** A secret é devolvida na criação do endpoint ([09:31] Marcos).
- [ ] **PRD-AC-04** O cliente só recebe os status que escolheu ([09:33] Marcos).
- [ ] **PRD-AC-05** URL sem `https` é recusada ([09:23] Sofia).
- [ ] **PRD-AC-06** Toda notificação leva assinatura e identificador único do evento ([09:22] Sofia, [09:26] Larissa).
- [ ] **PRD-AC-07** Com o cliente fora do ar, o OMS tenta de novo nos intervalos definidos e, ao final, guarda o evento na DLQ ([09:17] Larissa).
- [ ] **PRD-AC-08** Só usuário `ADMIN` consegue fazer replay, e o replay fica registrado ([09:36] Sofia).
- [ ] **PRD-AC-09** O cliente consulta o histórico das últimas entregas de um endpoint ([09:34] Marcos).
- [ ] **PRD-AC-10** Depois da rotação, a secret antiga funciona por 24 horas ([09:21] Sofia).
- [ ] **PRD-AC-11** Um cliente fora do ar não afeta a mudança de status de nenhum pedido ([09:04] Bruno).
- [ ] **PRD-AC-12** A revisão de segurança foi concluída antes do deploy ([09:46] Sofia).

---

### Estratégia de testes e validação

Tipos de teste obrigatórios
- Testes de integração da API no padrão atual, com Vitest e Supertest, como em `tests/orders.test.ts`.
- Testes ponta a ponta da integração com a mudança de status ([09:46] Larissa).
- Revisão de segurança de HMAC e geração de secret ([09:46] Sofia).

Estratégia de validação
- Validar os critérios de aceitação acima em ambiente com o worker rodando.
- Medir o tempo entre mudança de status e envio, para confirmar a meta de PRD-OBJ-01.
- O detalhamento técnico dos testes e os critérios de aceite técnicos estão no [FDD](./FDD.md).
