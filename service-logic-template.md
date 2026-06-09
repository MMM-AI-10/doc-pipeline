# `{service-name}` — Service Logic

> Technical structure: components, architecture, full interaction table.
> Business meaning → [business-logic.md](./business-logic.md)
> Step-by-step scenarios → [flows.md](./flows.md)
> Relationship overview → [README.md](./README.md)

---

## Service Components

| Component | What it does | File |
|-----------|------------|------|
| `___` | `___` | `src/___` |

---

## Internal Architecture

> GOOD examples:
>
> Request-Response (REST service):
> [REST Request]
> ↓
> [Router] — express/fastify
> ↓
> [Auth Middleware] — JWT / API Key
> ↓
> [Validation] — Joi / Zod
> ↓
> [Controller]
> ├── [Service] — business logic
> │   ├── [Repository] ──→ PostgreSQL
> │   ├── [Cache] ──────→ Redis
> │   └── [Client A] ───→ external service
> └── [KafkaProducer] ────→ Kafka
>
> Event-Driven (worker):
> [Kafka Consumer] ← kafka: order.status.changed
> ↓
> [DeduplicateService] — check Redis, was this already processed
> ↓
> [ChannelRouter] — which channel to use
> ├── [EmailAdapter] ──→ SendGrid
> └── [PushAdapter] ───→ Firebase FCM
> ↓
> [NotificationRepository] ──→ PostgreSQL (history)
> [KafkaProducer] ──────────→ notification.sent

```
[Incoming request]
  ↓
[___]
  ↓
[___]
```

---

## Full Interaction Table

### Incoming

| From | Protocol | Endpoint / Topic | Payload | When | Our handler |
|------|----------|-----------------|---------|------|------------|
| `___` | REST | `___` | `{ ___ }` | `___` | `___` |
| `___` | Kafka | `___` | `{ ___ }` | `___` | `___` |

### Outgoing (sync)

| To | Protocol | Endpoint | Payload | Response | Timeout | Their file |
|----|---------|---------|---------|----------|---------|-----------|
| `___` | REST | `___` | `{ ___ }` | `{ ___ }` | `___s` | [→](../___/) |

### Outgoing (async — Kafka)

| Topic | When | Who consumes | Their file |
|-------|------|-------------|-----------|
| `___` | `___` | `___` | [→ dataflow.md#___](./dataflow.md#___) |

---

## Caching

| What | Key | TTL | Where | Why |
|------|-----|-----|-------|-----|
| `___` | `___:{___}` | `___m` | Redis | `___` |

→ Redis ENV: [configs.md#redis](./configs.md#redis)
→ Redis keys in detail: [databases.md#redis](./databases.md#redis)

---

## Tracing

| Header | From | Where we pass | Why |
|--------|------|--------------|-----|
| `X-Trace-ID` | `___` | All calls + Kafka | End-to-end tracing |
| `X-Request-ID` | `___` | Response to client | Idempotency |
| `___` | `___` | `___` | `___` |

→ Tracing in Grafana: [observability.md#tracing](./observability.md#tracing)

---

## Architectural Invariants

> Rules that must not be broken when changing code.
>
> GOOD examples:
> "Only Order Service writes to the orders table — no direct UPDATEs from other services"
> "Every status change publishes order.status.changed — no silent transitions"
> "Reservation is always created before initiating payment — can't pay for unchecked stock"
> "Deduplication is mandatory — every incoming Kafka event is checked in Redis before processing"

- `___`
- `___`

---

## Links

- Business meaning → [business-logic.md](./business-logic.md)
- Scenarios → [flows.md](./flows.md)
- Kafka events in detail → [dataflow.md](./dataflow.md)
- API in detail → [api-contracts.md](./api-contracts.md)
- Relationship overview → [README.md](./README.md)
