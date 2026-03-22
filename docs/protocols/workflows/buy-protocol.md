# Buy Protocol

**Status:** [IMPLEMENTED] Base flow | [PHASE 2] Escrow phases, timeout, auto-refund
**Affects:** `POST /api/tickets`, `PATCH /api/tickets/:id`, `prisma/schema.prisma`

## Purpose

Describes the complete flow when a buyer (Partner) purchases an offer
from a seller (Initiator).

---

## Phase 1 — Currently Implemented

```
1. Buyer browses offers
   GET /api/offers  or  via Nostr Relay (Kind 30017)

2. Buyer creates ticket
   POST /api/tickets
   { offerId, buyerPubkey, sellerPubkey, amountSats }
   → Status: PENDING
   → Lightning invoice created (if LNBits is configured)

3. Buyer pays invoice (in Lightning wallet)
   → Webhook: POST /api/lightning/webhook
   → Status: PAID  (or directly IN_PROGRESS depending on config)

4. Seller accepts ticket
   PATCH /api/tickets/:id  { action: "accept" }
   → Status: IN_PROGRESS

5. Service / goods delivered
   (off-chain, no automatic tracking in Phase 1)

6. Completion
   PATCH /api/tickets/:id  { status: "COMPLETED" }
   → manual, by either party

7. On problem
   PATCH /api/tickets/:id  { status: "DISPUTED", disputeReason: "..." }
```

---

## Phase 2 — Escrow Flow (planned)

```
1. Buyer creates ticket
2. Server creates Phase 1 Hold Invoice (25% reservation)
   → LNBits Hold Invoice (payment held, not yet released)
3. Buyer pays → reservation confirmed → Status: ACCEPTED
4. Seller confirms start of delivery → Status: IN_PROGRESS
   → Phase 2 Invoice (50%) created
5. Buyer pays Phase 2 → seller receives 75%
6. Buyer confirms receipt of service
   → Phase 3 Invoice (25%) created
7. Buyer pays Phase 3 → seller receives remaining 25%
   → Status: COMPLETED
8. Review (Nostr Event, Phase 3)
```

---

## Error Cases

| Situation | Phase 1 (now) | Phase 2 (planned) |
|-----------|--------------|-------------------|
| Seller does not respond | Manual CANCELLED | Timeout (72h) → auto-CANCELLED + refund |
| Buyer does not pay | Ticket stays PENDING | BOLT11 expiry → auto-CANCELLED |
| Dispute after delivery | DISPUTED, manual resolution | Dispute Protocol (Phase 4) |
| Partial failure | Manual resolution | Escrow phase split via Dispute |

---

## Database State (Phase 1)

```typescript
// After step 2 (ticket created):
{
  status: "PENDING",
  lightningInvoice: "lnbc...",  // null if no LNBits configured
  amountSats: 1000
}

// After step 3 / step 4 (accepted):
{ status: "IN_PROGRESS" }

// After step 6 (completed):
{ status: "COMPLETED" }

// On dispute:
{ status: "DISPUTED", disputeReason: "Item never arrived" }
```

---

## Open Items (Phase 1 Backlog)

- `PATCH /api/offers/:id` missing — offer cannot be closed when ticket reaches COMPLETED
- `ACCEPTED` status is in schema but not set by routing
- No notification system — seller must manually poll for new tickets
