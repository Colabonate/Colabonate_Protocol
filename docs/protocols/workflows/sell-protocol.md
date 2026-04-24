# Sell Protocol

**Normativity:** Mixed

**Status:** [IMPLEMENTED] Base flow | [PHASE 2] Notifications, offer-close, escrow
**Affects:** `POST /api/offers`, `GET /api/offers`, `PATCH /api/tickets/:id`

## Purpose

Describes the flow for sellers / providers (Initiator role) — from creating
an offer to completing a transaction.

---

## Phase 1 — Currently Implemented

```
1. Seller authenticates via LNURL-Auth
   GET /api/auth/lnurl → LNURL generated
   → Wallet signs k1
   GET /api/auth/lnurl/callback?k1=...&sig=...&key=...
   → pubkey used as identity

2. Seller creates offer
   POST /api/offers
   { title, description, priceSats, category, creatorPubkey }
   → Stored in SQLite (Status: ACTIVE)
   → Automatically published via Nostr (Kind 30017)
   → nostrEventId stored (null if relay unreachable)

3. Offer is publicly visible
   GET /api/offers  →  all ACTIVE offers
   GET /api/offers/:id  →  details

4. Buyer creates ticket (→ Buy Protocol)
   ← No push event in Phase 1, seller must poll manually

5. Seller views their tickets
   GET /api/tickets?pubkey=<creatorPubkey>

6. Seller accepts ticket
   PATCH /api/tickets/:id  { action: "accept" }
   → Status: IN_PROGRESS

7. Deliver service / goods (off-chain)

8. Report completion
   PATCH /api/tickets/:id  { status: "COMPLETED" }
```

---

## Offer Lifecycle

```
ACTIVE
  └─► CLOSED    (seller closes manually)
  └─► ARCHIVED  (admin / timeout)          ← not implemented
```

---

## Nostr Publication

When an offer is created, a Nostr Event is automatically published.

**Behavior on relay error:**
- Error is logged on the server
- Offer is still saved in SQLite
- `nostrEventId` stays `null`
- Offer is visible locally but not via Nostr

**Manually re-publish:**
```
POST /api/nostr/offers  { offerId }
```

---

## Phase 2 Additions (planned)

| Feature | Description |
|---------|-------------|
| Push notification | When buyer creates a ticket → Nostr DM or webhook |
| Offer-close endpoint | `PATCH /api/offers/:id { status: "CLOSED" }` |
| Offer expiry | Offer automatically closes after X days |
| Escrow deposit | Seller can optionally deposit a security bond |
| Multi-ticket logic | Limit enforcement (max N open tickets per offer) |

---

## Key Schema Fields

```typescript
// Offer after step 2:
{
  id: "clxxx...",
  title: "Pizza Margherita",
  description: "Fresh ingredients, 30cm",
  priceSats: 5000,
  category: "food",
  status: "ACTIVE",
  nostrEventId: "abc123...",  // null if relay unreachable
  creatorPubkey: "02abc...",
  createdAt: "2026-03-21T..."
}
```


---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
