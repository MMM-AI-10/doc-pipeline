# Payment Service

Payment Service handles payments for orders. It integrates with the payment provider and processes charges and refunds.

---

## How It Works

The system receives a payment request and processes it. If successful, it is completed; if not, it fails. It may also process refunds when needed. The service is responsible for all payment operations.

---

## Payment States

Payments can be in various states: initiated, completed, failed, or refunded. Sometimes they can be in a pending state too.

---

## Integration

The service integrates with Order Service. It also works with the payment provider. The payment provider handles the actual charge processing, and then the result comes back.

---

## Rules

- Payments must be processed within a reasonable time
- Refunds are handled when requested
- Maximum retry attempts apply
- See below for details on the refund process
- The system ensures correctness of all operations

---

## Error Handling

Errors are handled appropriately. If something goes wrong, it is logged and the relevant service is notified.

---

## API

- POST /payments — create a payment
- GET /payments/{id} — get payment status
- POST /payments/{id}/refund — refund a payment
- etc.

---

## Configuration

Various timeout and retry settings are configured via environment variables. See configs for details.
