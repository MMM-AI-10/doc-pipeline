# `{service-name}` — Flows

> Step-by-step execution scenarios with concrete calls, payloads, and links to details.
> Read a flow top to bottom — you understand everything. Details — by links.
> Business meaning of each flow → [business-logic.md](./business-logic.md)

---

## Navigation

| Flow | Trigger | Anchor |
|------|---------|--------|
| Happy Path | `___` | [#happy-path](#happy-path) |
| Error Handling | `___` | [#error-handling](#error-handling) |
| `___` | `___` | |

---

## Flow: Happy Path {#happy-path}

**Trigger:** `___`

> GOOD examples:
> "Customer clicked 'Place Order', cart is not empty, items in stock"
> "Order Service published order.status.changed, user subscribed to email"
> "User sent POST /payments with a valid card"

**Participants (chain):** `___`

> GOOD examples:
> "Web UI → API Gateway → Order Service → Warehouse Service → Payment Service → Notification Service"
> "Order Service (kafka) → Notification Service → User Service → SendGrid"

**Business context:** [business-logic.md#process-___](./business-logic.md#process-___)

---

### Step 1: Request arrives

```
[___] → [{service-name}]
  Protocol: REST / Kafka / ___
  Endpoint / Topic: ___
  Payload: {
    ___
  }
```

**What we validate on input:**

- `___`

**If validation fails:** `___` → `[errors.md#___](./errors.md#___)`

→ Validation details: [api-contracts.md#___](./api-contracts.md#___)

---

### Step 2: `___`

```
[{service-name}] → [___]
  Protocol: ___
  Payload: { ___ }
  ← Response: { ___ }
```

**What we do with the response:**
- Success: `___`
- Error: `___`

→ Details: [___/api-contracts.md#___](../___/api-contracts.md#___)

---

### Step N: Incoming response from another service {#step-N}

```
[___] → [{service-name}] ← incoming
  Protocol: Kafka / Webhook
  Topic / Path: ___
  Payload: { ___ }
  Our handler: ___
```

**What we do with received data:**
- If `status = success`: `___`
- If `status = failed`: `___` → [Flow: Error Handling](#error-handling)

→ From sender's side: [___/dataflow.md#___](../___/dataflow.md#___)

---

### Flow Final State

```
Status: ___
Published events: ___
User outcome: ___
```

→ Kafka events: [dataflow.md#___](./dataflow.md#___)

---

## Flow: Error Handling {#error-handling}

**Trigger:** `___`

---

### Dependency unavailable

| Dependency | Behavior | Client receives |
|-----------|----------|----------------|
| `postgres` | Service down | `503 DATABASE_UNAVAILABLE` |
| `redis` | Working without cache | `200` (degraded) |
| `___` | `___` | `___` |

→ Full table: [errors.md#dependencies](./errors.md#dependencies)

---

### Retry strategy

| Error | Retry | Backoff |
|-------|-------|---------|
| `5xx / timeout` | 3× | `1s → 2s → 4s` |
| `429` | 3× | By `Retry-After` |
| `4xx` | no | — |

→ [errors.md#retry](./errors.md#retry)

---

## Flow: `___` {#___}

> Add a separate flow for each non-trivial scenario.
> One flow = one scenario. Better 8 short flows than 1 long one covering everything.
>
> GOOD examples of scenarios to separate out:
> - Order cancellation after payment (requires refund)
> - Fallback to backup channel (push → email)
> - Admin action (differs from user action)
> - Bulk operation
> - Recovery after failure (retry/resume)

**Trigger:** `___`
**Business context:** [business-logic.md#___](./business-logic.md#___)

*(steps)*

---

## Links

- Business meaning of flows → [business-logic.md](./business-logic.md)
- API call details → [api-contracts.md](./api-contracts.md)
- Kafka event details → [dataflow.md](./dataflow.md)
- Error codes → [errors.md](./errors.md)
- Data structures → [entities.md](./entities.md)
- Incoming/outgoing overview → [README.md](./README.md)
