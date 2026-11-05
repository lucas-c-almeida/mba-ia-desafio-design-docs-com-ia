# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a **documentation-only challenge** (MBA Full Cycle, "Design Docs com IA"), written in Portuguese. The deliverable is a design-doc package for a new feature — an **Order Webhook Notification System** — derived from `TRANSCRICAO.md` (a recorded tech meeting) and from the existing OMS code. The full brief and acceptance criteria are in `DESAFIO.md` (`README.md` currently holds the same brief and must eventually be replaced with a description of the production process).

**Hard constraint: never modify `src/`, `prisma/`, `tests/`, or config files, and never edit `TRANSCRICAO.md`.** The code is reference material only. All work happens in `docs/` and `README.md`. Write the docs in Portuguese, in Markdown.

## Deliverables (in `docs/`, currently placeholders)

| File | Altitude | Key mandatory content |
| --- | --- | --- |
| `ADRs` in `docs/adrs/ADR-NNN-titulo-em-kebab-case.md` (5–8 files) | Single decision | MADR sections: Status, Contexto, Decisão, Alternativas Consideradas, Consequências. Cover ≥5 of: Outbox in MySQL; retry with backoff + DLQ; HMAC-SHA256 with per-endpoint secret; at-least-once with `X-Event-Id`; separate polling worker process; reuse of existing project patterns. ≥1 ADR must cite real code files. |
| `RFC.md` | Architecture, 2–4 pages | Metadata (meeting participants as reviewers), TL;DR, ≥2 alternatives discarded in the meeting with trade-off, ≥2 open questions, links to ≥2 ADRs. No FDD-level detail. |
| `FDD.md` | Implementation | Flows (outbox write, worker, retry, DLQ), ≥4 HTTP endpoints with request/response examples and status codes, error matrix with `WEBHOOK_*` codes, resilience, observability (metrics, logs, tracing), and a section **"Integração com o sistema existente"** naming ≥4 real file paths. |
| `PRD.md` | Product | ≥8 functional requirements, ≥1 quantitative metric, "Fora de escopo" with ≥2 items discarded/deferred in the meeting, ≥2 risks with probability/impact/mitigation. |
| `TRACKER.md` | Cross-cutting | Table `ID \| Documento \| Tipo \| Conteúdo (resumo) \| Fonte \| Localização`. Fonte is `TRANSCRICAO` (Localização `[hh:mm] Nome`) or `CODIGO` (real file path). ≥80% of doc items covered, ≥70% of rows from TRANSCRICAO, ≥5 rows from CODIGO. |

Suggested order from the brief: ADRs → RFC → FDD → PRD → Tracker → README. The documents must not duplicate each other — each lives at its own altitude.

**Traceability rule:** every requirement, decision, or constraint must trace to a timestamp/speaker in `TRANSCRICAO.md` or to a real file in the repo. Do not invent requirements. The meeting explicitly discards or defers some ideas — those belong in "Fora de escopo" / "Alternativas" / "Questões em aberto", never as requirements. Verify every cited file path exists and every cited `[hh:mm] Nome` exists in the transcript.

Transcript format: `[hh:mm] Nome: fala`. Participants: Larissa (Tech Lead), Marcos (PM), Bruno (Eng. Pleno, Pedidos), Diego (Eng. Sênior, Plataforma), Sofia (Segurança).

## Existing application (reference for the docs)

Node.js ≥20 + TypeScript (ESM, `.js` import suffixes), Express 4, Prisma 5 on MySQL 8, Zod, Pino, Vitest + Supertest. There is currently **no** eventing, queue, or webhook mechanism — that gap is what the feature fills.

Architecture points the docs will typically need to reference:

- **Modular layout** `src/modules/<name>/` with `*.routes.ts` → `*.controller.ts` → `*.service.ts` → `*.repository.ts` and `*.schemas.ts` (Zod). Manual DI wiring in `buildControllers()` in `src/app.ts`; routers mounted under `/api/v1` in `src/routes/index.ts`.
- **Order state machine** in `src/modules/orders/order.status.ts` (`PENDING → PAID → PROCESSING → SHIPPED → DELIVERED`, `CANCELLED` from the first three; stock debit on `PENDING→PAID`, replenish on cancel from `PAID`/`PROCESSING`).
- **`OrderService.changeStatus`** in `src/modules/orders/order.service.ts` runs inside `prisma.$transaction`: validates the transition, moves stock, updates the order, and writes `OrderStatusHistory`. This transaction is the natural hook for writing outbox events atomically.
- **Errors**: `AppError(message, statusCode, errorCode, details)` in `src/shared/errors/app-error.ts`, with subclasses in `src/shared/errors/http-errors.ts` (codes like `INVALID_STATUS_TRANSITION`, `INSUFFICIENT_STOCK`). `src/middlewares/error.middleware.ts` maps `AppError`, `ZodError`, and Prisma `P2002`/`P2025` to `{ error: { code, message, details? } }`.
- **Auth**: `authenticate` and `requireRole(...roles)` in `src/middlewares/auth.middleware.ts` (roles `ADMIN` | `OPERATOR`; `requireRole` is currently used only in `src/modules/users/user.routes.ts`).
- **Logging**: Pino logger in `src/shared/logger/index.ts` (redacts auth headers, passwords, tokens); `src/middlewares/request-logger.middleware.ts` propagates/creates `X-Request-Id`.
- **Pagination helper**: `src/shared/http/response.ts`. Env validation via Zod in `src/config/env.ts`.
- Schema: `prisma/schema.prisma` (models `User`, `Customer`, `Product`, `Order`, `OrderItem`, `OrderStatusHistory`, `OrderNumberSequence`).

## Commands (only needed to inspect/run the reference app)

```bash
docker compose up -d          # MySQL 8 on :3306 (copy .env.example to .env first)
npm run db:migrate            # prisma migrate dev
npm run db:seed
npm run dev                   # tsx watch, loads .env
npm test                      # vitest run (needs MySQL; tests run serially in one fork, DB wiped in tests/setup.ts)
npx vitest run tests/orders.test.ts   # single test file
npx vitest run -t "name"              # single test by name
npm run lint
npm run build
```
