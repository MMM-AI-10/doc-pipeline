# `{service-name}` — README

> **One line:** what the service does and why it exists.
>
> GOOD examples:
> ✅ "Notification Service — sends notifications via email, push, Telegram. Abstracts channel selection — other services don't know how a message is delivered."
> ✅ "Payment Service — processes payments through external providers (Stripe, YooKassa), stores transaction history, ensures operation idempotency."
> ✅ "Auth Service — issues and validates JWT tokens, manages sessions. The only service in the system that knows user passwords."
> ❌ "A service for working with data" — too abstract, doesn't explain why

**Criticality:** `critical` / `high` / `medium` / `low`
**Layer:** `control` / `execution` / `integration` / `data` / `infra`
**Owner:** `___`

---

## What and Why

**What it does:**

> GOOD examples — pick a style:
>
> Coordinator service:
> "Order Service manages the order lifecycle from creation to delivery. When a customer places an order — it checks stock, reserves warehouse inventory, initiates payment, coordinates delivery. None of these steps bypasses Order Service."
>
> Executor service:
> "Email Service receives send tasks and delivers via SMTP/SendGrid. Doesn't know business context — only: who, what, when. Stores history and handles bounce/unsubscribe from provider."
>
> Storage service:
> "User Profile Service stores extended user data: preferences, history, interface settings. Answers 'who is this user and what do they like' — Auth Service answers 'do they have the right to enter'."

`___`

**Key idea / principle:**

> GOOD examples:
> "Idempotency — repeat call with same Idempotency-Key gives same result. This is the guarantee for upstream services on retry."
>
> "Single source of truth — only this service writes to the orders table. Everyone else reads, but never writes directly."
>
> "Fire-and-forget — accepts task, responds 202 Accepted, executes async. Result via Kafka event."

`___`

**Why it exists as a separate service:**

> GOOD examples:
> "Isolates payment provider logic. When Stripe API changes — only Payment Service changes, the other 12 services don't know."
>
> "Separates responsibility: Auth knows secrets (passwords, keys), User Profile knows preferences. Compromising one doesn't expose the other."

`___`

---

## Place in the System

> GOOD examples of diagrams:
>
> Linear chain:
> [API Gateway] → [Order Service] → [Payment Service] → [Stripe API]
> ↓
> [Warehouse Service] → [Delivery Service]
>
> Star (coordinator):
> [Telegram Bot] [Web UI] [Mobile App]
> ↘ ↓ ↙
> [Orchestrator Service]
> ↙ ↑ ↘
> [Policy Engine] [Workflow Engine] [Dialog Tool]
>
> Event consumer:
> [Order Service] ──kafka:order.created──→
> [Payment Service] ──kafka:payment.done──→ [Notification Service] → [Email/Push/TG]
> [Delivery Service] ──kafka:shipped──────→

```
[___] → [{service-name}] → [___]
```

**Belongs to service group:** `___`

> GOOD examples:
> "Control Layer — management + action execution"
> "Integration Layer — adapters to external systems"
> "Foundation Layer — base infrastructure + observability + operations"

---

## Incoming Links — who calls me

> RULE: for every incoming link — include a link to THAT service's file,
> so the reader can see this same relationship from the other side.
>
> GOOD example for Notification Service:
> | `order-service` | Notify of status change | Kafka | `order.status.changed` | `{ order_id, user_id, status }` | [→ order-service/dataflow.md#produces](../order-service/dataflow.md#produces) |
> | `payment-service` | Notify of payment | Kafka | `payment.completed` | `{ payment_id, user_id, amount }` | [→ payment-service/dataflow.md#produces](../payment-service/dataflow.md#produces) |

| Service | Why they call | Protocol | Endpoint / Topic | What they send | Their file |
| --- | --- | --- | --- | --- | --- |
| `___` | `___` | `___` | `___` | `___` | [→ ](../___) |

---

## Outgoing Links — who I call

| Service | Why I call | Protocol | Endpoint / Topic | What I expect back | Their file |
| --- | --- | --- | --- | --- | --- |
| `___` | `___` | `___` | `___` | `___` | [→ ](../___) |

---

## Quick Navigation

| Question | File |
| --- | --- |
| Why this service, business rules, processes? | [business-logic.md](./business-logic.md) |
| Components and all interactions? | [service-logic.md](./service-logic.md) |
| Step-by-step scenarios? | [flows.md](./flows.md) |
| Entities and fields? | [entities.md](./entities.md) |
| REST endpoints? | [api-contracts.md](./api-contracts.md) |
| Kafka topics? | [dataflow.md](./dataflow.md) |
| DB and Redis? | [databases.md](./databases.md) |
| ENV? | [configs.md](./configs.md) |
| Roles and permissions? | [rbac.md](./rbac.md) |
| Error codes? | [errors.md](./errors.md) |
| Auth, PII? | [security.md](./security.md) |
| Logs, metrics? | [observability.md](./observability.md) |
| Docker, startup? | [infrastructure.md](./infrastructure.md) |
| External services? | [integrations.md](./integrations.md) |

---

## Key Scenarios

| Scenario | Link |
| --- | --- |
| Happy path | [flows.md#happy-path](./flows.md#happy-path) |
| Error handling | [flows.md#error-handling](./flows.md#error-handling) |
| `___` | |

---

## Critical Dependencies

| Dependency | Critical | Behavior on failure |
| --- | --- | --- |
| `postgres` | ✅ | Service won't start |
| `redis` | | |
| `kafka` | | |
| `___` | | |

→ [errors.md#dependencies](./errors.md#dependencies) · [infrastructure.md#healthcheck](./infrastructure.md#healthcheck)
