# `payment-service` — Business Logic

> Payment Service processes payments for customer orders and tracks the payment lifecycle from initiation through settlement or failure.

---

## Service Purpose

**In one sentence:**

> Payment Service is the single authority that determines whether a payment has succeeded, failed, or been refunded — no other service makes this decision.

**Who it serves:**

| Actor | What they get |
|-------|--------------|
| Order Service | Payment status for order lifecycle transitions |
| Customer (via API Gateway) | Payment confirmation, refund status |
| Finance Team | Transaction records, settlement reports |

**Why it exists separately:**

> Without Payment Service, Order Service would need to integrate directly with payment providers, understand card network responses, handle idempotency keys, and manage refund rules. Payment Service isolates all payment-provider complexity and exposes a single status model that Order Service can rely on.

---

## Key Concepts

| Term | Definition |
|------|-----------|
| Payment | A single attempt to charge a customer for an order. One order can have multiple payments (retries, partial refunds). |
| Charge | The actual debit request sent to the payment provider. One payment has exactly one charge. |
| Settlement | Transfer of funds from customer to merchant account. Happens asynchronously after charge success. |
| Refund | Reversal of a settled charge. Can be full or partial. |

---

## Business Processes

### Process: Process Payment

**Trigger:** Order Service requests payment for a confirmed order.

**Actor:** Order Service

**Preconditions:**
- Order exists in PENDING_PAYMENT status
- Order total is calculated and final
- Customer has provided payment method

**Steps (business language only):**

1. Validate that the order total matches the payment amount
2. Create a payment record in INITIATED status
3. Send charge request to payment provider
4. On provider response: transition payment to COMPLETED or FAILED
5. On COMPLETED: publish `payment.completed` event
6. On FAILED: publish `payment.failed` event with reason

**Business rules:**
- Payment amount must exactly match order total — no partial payments
- Maximum 3 charge attempts per order (separate payment records)
- Charge attempt must be initiated within 30 minutes of order creation
- Idempotency: retrying the same charge (same idempotency key) returns the original result

**Exceptions and edge cases:**

| Situation | What happens |
|-----------|-------------|
| Payment provider timeout | Payment stays in INITIATED; cron job checks status after 5 minutes |
| Charge declined (insufficient funds) | Payment → FAILED; reason = INSUFFICIENT_FUNDS |
| Charge declined (card expired) | Payment → FAILED; reason = CARD_EXPIRED |
| Duplicate charge attempt | Returns existing payment result (idempotency) |

**Result:**
- Success: Payment in COMPLETED status; Order Service receives `payment.completed`
- Failure: Payment in FAILED status; Order Service receives `payment.failed`

→ How implemented: [flows.md#process-payment](./flows.md#process-payment)

---

### Process: Refund Payment

**Trigger:** Order Service requests a refund for a delivered order.

**Actor:** Order Service

**Preconditions:**
- Payment is in COMPLETED status (settled)
- Refund amount does not exceed the original charge amount minus any prior refunds
- Refund requested within 60 days of original charge

**Steps:**

1. Validate refund amount against remaining refundable balance
2. Create a refund record in PENDING status
3. Send refund request to payment provider
4. On provider confirmation: refund → COMPLETED
5. Publish `payment.refunded` event with refund amount

**Business rules:**
- Partial refunds allowed; total refunded amount cannot exceed original charge
- Refund of $0 is rejected
- If multiple refunds are requested concurrently, total is checked atomically
- Refund after 60 days requires manual Finance Team approval (not handled by this process)

**Exceptions and edge cases:**

| Situation | What happens |
|-----------|-------------|
| Payment provider rejects refund | Refund → FAILED; reason recorded; Order Service notified |
| Refund exceeds balance | Rejected immediately with REMAINING_BALANCE_EXCEEDED |
| Original payment not settled yet | Refund queued; processed after settlement confirmation |

**Result:**
- Success: Refund COMPLETED; `payment.refunded` event published
- Failure: Refund FAILED; reason available for display

→ How implemented: [flows.md#refund-payment](./flows.md#refund-payment)

---

## Status Model

```
Payment:
INITIATED → COMPLETED → REFUND_INITIATED → REFUNDED
 ↓
FAILED

Refund:
PENDING → COMPLETED
 ↓
FAILED
```

| Status | Business meaning | Who sets it | Transitions |
|--------|-----------------|------------|------------|
| INITIATED | Charge sent to provider, awaiting response | Payment Service (on charge request) | → COMPLETED, → FAILED |
| COMPLETED | Charge succeeded, funds will be settled | Payment Service (on provider success) | → REFUND_INITIATED |
| FAILED | Charge declined or errored | Payment Service (on provider failure) | terminal |
| REFUND_INITIATED | Refund requested, awaiting provider | Payment Service (on refund request) | → REFUNDED, → FAILED |
| REFUNDED | Refund confirmed by provider | Payment Service (on provider confirmation) | terminal |

→ Transition rules in data: [entities.md#payment-status](./entities.md#payment-status)

---

## Roles and What They Can Do

| Role | Can | Cannot |
|------|-----|--------|
| Customer | View own payments, request refund (via Order Service) | Initiate charges directly, view other customers' payments |
| Order Service | Request charge, request refund, query payment status | Access raw provider responses, override payment status |
| Finance Team | View all transactions, approve 60+ day refunds | Modify payment status, initiate charges |
| Admin | View all transactions, cancel pending refunds | Change completed payment status |

→ Detailed matrix: [rbac.md](./rbac.md)

---

## SLA and Constraints

| Parameter | Value | What happens on violation |
|-----------|-------|--------------------------|
| Charge processing time | ≤ 10 seconds (p99) | Payment stays INITIATED; cron reconciles after 5 min |
| Refund processing time | ≤ 30 seconds (p99) | Refund stays PENDING; reconciliation after 10 min |
| Maximum charge attempts per order | 3 | 4th attempt rejected with MAX_ATTEMPTS_EXCEEDED |
| Refund window | 60 days | After 60 days, manual Finance approval required |
| Idempotency key TTL | 24 hours | Duplicate detection window; after 24h, new charge allowed |

→ ENV for parameters: [configs.md](./configs.md)

---

## Glossary

| Term | Definition |
|------|-----------|
| Charge | A debit request against a customer's payment method |
| Settlement | Asynchronous transfer of funds from card network to merchant account |
| Idempotency key | Client-generated UUID ensuring duplicate requests return same result |
| Refundable balance | Original charge amount minus sum of all completed refunds |

---

## Links

- Technical components → [service-logic.md](./service-logic.md)
- Step-by-step scenarios → [flows.md](./flows.md)
- Entities and data → [entities.md](./entities.md)
- Roles and permissions → [rbac.md](./rbac.md)
- Incoming/outgoing → [README.md](./README.md)
