# `{service-name}` — Business Logic

> What the service does at the business level: why, for whom, by what rules.
> This file is readable without knowledge of the tech stack. No HTTP, Kafka, Redis — only meaning.
> Technical details → [service-logic.md](./service-logic.md)
> Step-by-step scenarios → [flows.md](./flows.md)

---

## Service Purpose

**In one sentence:**

> GOOD examples:
> "Order Service manages the order lifecycle and is the single place where order status decisions are made."
>
> "Notification Service delivers notifications to users on the right channel at the right time, regardless of which service initiated them."
>
> "Warehouse Service knows what's in stock and reserves items for orders, preventing selling more than physically exists."

`___`

**Who it serves:**

| Actor | What they get |
|-------|--------------|
| `___` | `___` |

**Why it exists separately:**

> GOOD examples:
> "Without Order Service, each of 8 services (Payment, Warehouse, Delivery, Notification...) would have to track order state independently. Order Service is the only one that knows 'where the order is right now'."
>
> "Notification Service isolates knowledge about delivery channels. Order Service doesn't know that a user prefers Telegram over email. It publishes an event — Notification Service figures out how and where."

`___`

---

## Key Concepts

> Terms and principles specific to this service.
> Without this section, other files can't be filled correctly.

| Term | Definition |
|------|-----------|
| `___` | `___` |

---

## Business Processes

### Process: `___`

**Trigger:**

> GOOD examples:
> "Customer clicked 'Place Order' in the web UI"
> "Payment Service published payment.completed event"
> "User hasn't logged in for 30 days (cron job)"

`___`

**Actor:** `___`

**Preconditions:**

- `___`

**Steps (business language only, no technical details):**

> GOOD examples for "Create Order":
> 1. Verify all items in cart are in stock
> 2. Reserve items in warehouse (reservation valid for 30 minutes)
> 3. Calculate total with discounts and delivery
> 4. Initiate payment
> 5. On payment success — confirm order and hand off to fulfillment
> 6. Notify customer of confirmation

1. `___`
2. `___`

**Business rules:**

> GOOD examples:
> - Cannot create order if any item is out of stock
> - Reservation expires if payment not received within 30 minutes
> - Order cannot be cancelled once handed to courier (status IN_DELIVERY)
> - Return possible only within 14 days of receipt

- `___`

**Exceptions and edge cases:**

| Situation | What happens |
|-----------|-------------|
| `___` | `___` |

**Result:**
- Success: `___`
- Failure: `___`

→ How implemented: [flows.md#___](./flows.md#___)

---

### Process: `___`

*(repeat block)*

---

## Status Model

> GOOD examples:
>
> Order:
> DRAFT → PENDING_PAYMENT → PAID → IN_FULFILLMENT → SHIPPED → DELIVERED
> ↓               ↓
> CANCELLED     CANCELLED
>
> Notification:
> QUEUED → SENDING → SENT → DELIVERED
> ↓            ↓
> FAILED    BOUNCED → (channel deactivation)

```
[___] → [___] → [___]
  ↓
[___]
```

| Status | Business meaning | Who sets it | Transitions |
|--------|-----------------|------------|------------|
| `___` | `___` | `___` | → `___` |

→ Transition rules in data: [entities.md#___](./entities.md#___)
→ Error on invalid transition: [errors.md#INVALID_STATE_TRANSITION](./errors.md#invalid-state-transition)

---

## Roles and What They Can Do

| Role | Can | Cannot |
|------|-----|--------|
| `___` | `___` | `___` |

→ Detailed matrix: [rbac.md](./rbac.md)

---

## SLA and Constraints

| Parameter | Value | What happens on violation |
|-----------|-------|--------------------------|
| `___` | `___` | `___` |

→ ENV for parameters: [configs.md](./configs.md)

---

## Glossary

| Term | Definition |
|------|-----------|
| `___` | `___` |

---

## Links

- Technical components → [service-logic.md](./service-logic.md)
- Step-by-step scenarios → [flows.md](./flows.md)
- Entities and data → [entities.md](./entities.md)
- Roles and permissions → [rbac.md](./rbac.md)
- Incoming/outgoing → [README.md](./README.md)
