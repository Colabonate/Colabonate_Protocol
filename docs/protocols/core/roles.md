# Roles in the Colabonate System

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

### Mediator [PHASE 4]

Community expert who mediates at dispute level 2.

**Requirements:**
- Valid Verification Ticket (Phase 3, reputation score above threshold)
- No stake in either disputing party

**Actions:**
- View ticket history and submitted evidence
- Propose a resolution (non-binding)
- 14-day mediation window

**Limitation:** Mediators cannot force payments. They can only make a proposal
that both parties must voluntarily accept.

---

### Arbitrator [PHASE 4]

DAO juror at dispute level 3.

**Requirements:**
- Elected via Governance Ticket
- Staking deposit [PHASE 4]

**Actions:**
- Review all evidence and mediation protocol
- Vote (majority decision, 3–5 arbitrators)
- Verdict published as Nostr Event (public, immutable)
- Lightning escrow split according to verdict

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
