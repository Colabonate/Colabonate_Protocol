# Buy Protocol

**Normativity:** Mixed

**Status:** [IMPLEMENTED] Base flow | [PHASE 2] Escrow phases, timeout, auto-refund, OfferType distinction
**Affects:** `POST /api/tickets`, `PATCH /api/tickets/:id`, `prisma/schema.prisma`

> (PDC: see ADR-021) – Added OfferType SERVICE vs PRODUCT distinction

## Purpose

Describes the complete flow when a buyer (Partner) purchases an offer
from a seller (Initiator).

---

## Offer Types (Phase 2)

| Type    | Description | UI Action | Status after creation | Seller approval |
|---------|-------------|-----------|----------------------|-----------------|
| **SERVICE** | 1:1 service per buyer (e.g., consulting, coaching, custom work) | "Request" | `PENDING` | Required |
| **PRODUCT** | Unlimited quantity (e.g., e-books, digital assets, templates) | "Buy Now" | `IN_PROGRESS` | Auto-accepted |

---

## SERVICE Flow (Seller must approve)

```
1. Buyer browses offers
   GET /api/offers

2. Buyer requests service
   POST /api/tickets
   { offerId, buyerPubkey, sellerPubkey, amountSats }
   → Availability check: Buyer has no active ticket for this offer?
   → Status: PENDING
   → Seller notification (Nostr Kind 30019)

3. Seller reviews request in dashboard
   → Sees "Accept" ✓ / "Reject" ✗ buttons
   
4a. Seller accepts:
   PATCH /api/tickets/:id  { action: "accept" }
   → Status: IN_PROGRESS
   → Buyer notification
   → Continue to step 5

4b. Seller rejects:
   PATCH /api/tickets/:id  { action: "reject" }
   → Status: CANCELLED
   → Buyer cannot request again

5. Seller creates Phase 1 invoice (25% reservation)
    POST /api/tickets/:id/escrow/phase1
    → EscrowStatus: PHASE_1_PENDING

6. Buyer pays invoice (in Lightning wallet)
    → Webhook: POST /api/lightning/webhook
    → Escrow Status: RESERVED (25% locked)
    → Ticket Status: PAID

7. Seller delivers service/goods
    → Seller signals work start: POST /api/tickets/:id/escrow/phase2
    → EscrowStatus: PHASE_2_PENDING → DELIVERY_STARTED (after buyer pays)
    → 75% total held

8. Buyer confirms receipt
    → Seller confirms delivery: POST /api/tickets/:id/escrow/phase3
    → Buyer pays Phase 3: EscrowStatus: DELIVERY_CONFIRMED → RELEASED
    → Status: COMPLETED

9. Review (Nostr Event Kind 30024, Phase 3)
```

---

## PRODUCT Flow (Auto-accepted)

```
1. Buyer browses offers
   GET /api/offers

2. Buyer clicks "Buy Now"
   POST /api/tickets
   { offerId, buyerPubkey, sellerPubkey, amountSats }
   → Status: IN_PROGRESS (auto-accepted, no seller action needed)
   → Seller can create invoice immediately

3. Seller creates Phase 1 invoice (25% reservation)
    POST /api/tickets/:id/escrow/phase1
    → EscrowStatus: PHASE_1_PENDING

4. Buyer pays invoice (in Lightning wallet)
    → Webhook: POST /api/lightning/webhook
    → Escrow Status: RESERVED (25% locked)
    → Ticket Status: PAID

5. Seller delivers product
    → Seller signals work start: POST /api/tickets/:id/escrow/phase2
    → EscrowStatus: PHASE_2_PENDING → DELIVERY_STARTED (after buyer pays)
    → 75% total held

6. Buyer confirms receipt
    → Seller confirms delivery: POST /api/tickets/:id/escrow/phase3
    → Buyer pays Phase 3: EscrowStatus: DELIVERY_CONFIRMED → RELEASED
    → Status: COMPLETED

7. Review (Nostr Event Kind 30024, Phase 3)
```

---

## Phase 1 — Legacy (deprecated)

> **Note:** Phase 1 flow (all tickets start in PENDING) is deprecated.
> New implementations should use SERVICE/PRODUCT distinction.

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

## Error Cases

| Situation | SERVICE Flow | PRODUCT Flow | Phase 2 (planned) |
|-----------|--------------|--------------|-------------------|
| Seller does not respond | Manual CANCELLED | N/A (auto-accepted) | Timeout (72h) → auto-CANCELLED + refund |
| Buyer does not pay | Ticket stays PENDING | Ticket stays PAID | BOLT11 expiry → auto-CANCELLED |
| Dispute after delivery | DISPUTED, manual resolution | DISPUTED, manual resolution | Dispute Protocol (Phase 4) |
| Partial failure | Manual resolution | Manual resolution | Escrow phase split via Dispute |

---

## Database State (Phase 2)

```typescript
// SERVICE Ticket after creation:
{
  status: "PENDING",
  offerType: "SERVICE",
  lightningInvoice: null,  // Created after seller accepts
  amountSats: 1000
}

// SERVICE Ticket after seller accepts:
{ status: "IN_PROGRESS" }

// PRODUCT Ticket after creation:
{
  status: "IN_PROGRESS",  // Auto-accepted
  offerType: "PRODUCT",
  lightningInvoice: null,  // Seller can create immediately
  amountSats: 1000
}

// After payment:
{ status: "PAID", escrowStatus: "RESERVED" }

// After completion:
{ status: "COMPLETED", escrowStatus: "RELEASED" }

// On dispute:
{ status: "DISPUTED", disputeReason: "Item never arrived" }
```

---

## Open Items (Phase 1 Backlog)

- No notification system — seller must manually poll for new tickets


---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
