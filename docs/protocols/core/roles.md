# Roles in the Colabonate System

**Normativity:** Normative

**Status:**
- Initiator, Partner: [IMPLEMENTED] (via Ticket schema)
- Mediator, Arbitrator: [PHASE 4]
- Observer: [IMPLEMENTED] (public offers readable via Nostr)

## Overview

```
┌──────────────┐     creates Offer       ┌─────────────────┐
│  Initiator   │ ──────────────────────► │  Nostr Relay    │
│  (Seller)    │                         │  (public)       │
└──────┬───────┘                         └────────┬────────┘
       │                                          │ sees Offer
       │  Ticket gets created                     ▼
       │ ◄──────────────────────────── ┌──────────────────┐
       │                               │   Partner        │
       │  Service + Lightning payment  │   (Buyer)        │
       └──────────────────────────────►└──────────────────┘
```

## Role Descriptions

### Initiator (Seller / Provider)

Creates offers and makes them available.

**Actions:**
- Create offer (`POST /api/offers`)
- Accept ticket (`PATCH /api/tickets/:id { action: "accept" }`)
- Confirm completion
- Deposit into escrow [PHASE 2]

**In schema:** `Offer.creatorPubkey`, `Ticket.sellerPubkey`

---

### Partner (Buyer / Contractor)

Finds offers and buys or cooperates.

**Actions:**
- Browse offers (`GET /api/offers`)
- Create ticket (`POST /api/tickets`)
- Pay via Lightning
- Complete milestones [PHASE 2]
- Leave review [PHASE 3]

**In schema:** `Ticket.buyerPubkey`

> **Note on schema naming:** The current fields are named `sellerPubkey` (= Initiator)
> and `buyerPubkey` (= Partner). This naming reflects the simplest buy/sell use case.
> For cooperations both parties are equal — a `ticketType` field will generalize
> this in Phase 2.

---

### Mediator [DEFERRED — Phase 3]

Level 2 mediator role. **Not yet implemented** — disputes in bootstrap mode skip directly from Level 1 (cooling-off) to Level 3 (Council).

See [arbitration-council.md](../governance/arbitration-council.md) Appendix A for details.

---

### Arbitrator [BOOTSTRAP]

DAO juror at dispute Level 3. In bootstrap mode (M0), the single admin (`FOUNDATION_DAO_ADMIN_PUBKEY`) serves as the sole arbitrator.

**Requirements (full Codex, Phase 4+):**
- Elected via Governance Vote
- HID Level 3 (Humanode biometric verification)
- COLA stake

**Bootstrap requirements (M0):**
- Pubkey matches `FOUNDATION_DAO_ADMIN_PUBKEY`
- `DaoMember.status = 'ACTIVE'`, `role ∈ {'ADMIN', 'ARBITRATOR'}`

**Actions:**
- Review all evidence and mediation record
- Issue verdict (RELEASE or CANCEL in M0; SPLIT deferred)
- Verdict published as Nostr Kind 30022 (`sub_type: arbitration_verdict`)
- Lightning escrow transition enforced via `verifyActor(ARBITRATOR)` in escrow router

---

### Observer

Anyone not holding an active role in a transaction.

**Actions:**
- Read public offers (`GET /api/offers` or via Nostr)
- View other users' reputation [PHASE 3]
- Read governance tickets [PHASE 4]

**No** action rights on tickets or payments.

## Role Switching

A single user (pubkey) can hold different roles across different tickets:
- Pubkey A is Initiator on Ticket 1, Partner on Ticket 2
- No separate account needed — role is determined by ticket context


---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
