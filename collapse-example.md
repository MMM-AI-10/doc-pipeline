# Collapse Example — Order Service Documentation

This example shows how collapse happens in practice when documentation contains competing versions of the same fact.

---

## The Two Files

### File A: `order-service/README.md`

```
Order Service manages the order lifecycle.

When a customer places an order:
1. Order Service validates items with Warehouse Service
2. Payment is processed by Payment Service
3. On payment success, Order Service confirms the order

Order states: DRAFT → PENDING_PAYMENT → PAID → SHIPPED → DELIVERED

Order Service calls Payment Service synchronously.
If payment fails, the order moves to CANCELLED.
```

### File B: `order-service/business-logic.md`

```
Order Service is the single source of truth for order status.

Process: Place Order
1. Order Service validates items with Warehouse Service
2. Order Service reserves items in Warehouse
3. Order Service publishes order.created event
4. Payment Service picks up the event and processes payment
5. On payment.completed event: Order Service transitions to PAID

Order states: DRAFT → PENDING_PAYMENT → PAID → IN_FULFILLMENT → SHIPPED → DELIVERED

Payment is initiated asynchronously via event.
If payment fails, the order returns to DRAFT (not CANCELLED — customer can retry).
```

---

## What Happens When an LLM Reads Both

```
1. LLM reads README.md first:
   "Payment is synchronous. Order calls Payment. Failed = CANCELLED.
    States: DRAFT → PENDING_PAYMENT → PAID → SHIPPED → DELIVERED"

2. LLM reads business-logic.md:
   "Payment is asynchronous via events. Failed = DRAFT (retry).
    States: includes IN_FULFILLMENT"

3. LLM must produce ONE answer about how payment works.
   → COLLAPSE

4. Without the protocol:
   LLM silently picks one version (often whichever was read last or is more detailed)
   → outputs with 100% confidence
   → no trace of the choice
   → human cannot detect the error

5. With the protocol:
   LLM emits a marker:
   [COLLAPSE:RED]
   CHOSEN: Async via events — order-service/business-logic.md
   ALTERNATIVE: Sync call — order-service/README.md
   REASON: business-logic.md is more detailed and describes the full process

   [COLLAPSE:YELLOW]
   CHOSEN: Failed → DRAFT — order-service/business-logic.md
   ALTERNATIVE: Failed → CANCELLED — order-service/README.md
   REASON: business-logic.md provides explicit business rationale (customer can retry)

   [COLLAPSE:YELLOW]
   CHOSEN: 6 states (includes IN_FULFILLMENT) — order-service/business-logic.md
   ALTERNATIVE: 5 states — order-service/README.md
   REASON: business-logic.md is the deeper layer per pyramid priority
```

---

## Why Each Collapse Level Matters

**COLLAPSE:RED (sync vs async)**
This is an architectural decision. A developer building a new service that integrates with Order Service will write fundamentally different code depending on which version they believe. Sync = HTTP call with retry logic. Async = event consumer with idempotency. Getting this wrong means the integration doesn't work.

**COLLAPSE:YELLOW (CANCELLED vs DRAFT on failure)**
This changes implementation details. If an agent is generating code to handle payment failure, it will either clean up and close the order (CANCELLED) or keep it open for retry (DRAFT). Different UX, different data, different downstream effects.

**COLLAPSE:YELLOW (5 vs 6 states)**
The IN_FULFILLMENT state means there's a whole phase between payment and shipping. Code that transitions directly from PAID to SHIPPED will break the state machine. An LLM that collapses to the 5-state version will generate code that skips a mandatory step.

---

## How to Fix

```
1. Resolve COLLAPSE:RED first:
   - Ask the team: is payment sync or async?
   - Update the INCORRECT file to match reality
   - Add [DEPRECATED] to the old version or rewrite it

2. Resolve COLLAPSE:YELLOW (failure state):
   - Clarify in business-logic.md with explicit business rule
   - If the rule is "return to DRAFT for retry within 30 min, then CANCELLED"
     then BOTH files were incomplete — add the full rule

3. Resolve COLLAPSE:YELLOW (state list):
   - README should summarize: "5+ states, see business-logic.md"
   - business-logic.md should contain the complete state diagram
   - This is also a COLLAPSE:LAYER fix: README and business-logic
     should not contain competing versions of the same fact

4. After fix:
   - README says "Payment initiated asynchronously (see business-logic.md)"
   - business-logic.md has the full process with all states and rules
   - No competing versions → no collapse possible
```

---

## The Takeaway

Without the Collapse Protocol, this situation produces **silent, confident errors**. The LLM picks one version, proceeds, and every subsequent answer builds on that choice. The error compounds.

With the protocol, every collapse point is **visible**. You see exactly where documentation contradicts itself, exactly what was chosen, and exactly what to fix. The goal is not to prevent collapse — it's to make it traceable and fixable.
