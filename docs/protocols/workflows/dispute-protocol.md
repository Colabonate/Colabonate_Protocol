# Dispute Protocol

**Normativity:** Normative

**Status:** [ACTIVE] — Level 1 + Level 3 implemented via ADR-125 M0 + ADR-126 M2; Level 2 deferred
**Prerequisites:** Lightning Escrow (Phase 2), Reputation System (Phase 3)

## Purpose

Three-level conflict resolution without a central authority.
Goal: a fair, transparent outcome with minimal escalation.

---

## Triggers

A dispute is opened when:

- Buyer reports: goods not received / quality issue / service not delivered
- Seller reports: payment not released even though service was fully delivered
- Timeout expires without completion [PHASE 2: automatic]

**Technically (Phase 1, already possible):**
```
POST /api/disputes/:ticketId (creates Dispute + sets ticket status to DISPUTED)
```

---

## 3 Levels

### Level 1: Cooling-off Period (7 days default, configurable)

```
Dispute opened (status: OPEN)
  → Ticket status: DISPUTED
  → 7-day (configurable) window starts
  → Both parties notified via Nostr

Communication via Nostr DM (NIP-04, end-to-end encrypted)
Parties can reach a direct agreement:

Possible outcomes:
  ✓ Opener withdraws                   → Status: WITHDRAWN (terminal)
  ✓ Agreement reached                  → Status: COMPLETED
  ✗ No agreement after cooling-off     → Escalate to Level 3
```

### Level 2: Mediation (14 days) — Deferred to Phase 3

```
Community mediator elected from verified pool
  → Requires: Verification Ticket (Phase 3), reputation score above threshold
  → No stake in either disputing party

Mediator can view:
  → All ticket events (timestamps, status changes)
  → Evidence voluntarily submitted by parties (photos, text)
  → Chat log (only with party consent)

Mediator proposes resolution (NOT binding)
  → Both parties must actively agree

Possible outcomes:
  ✓ Both accept proposal   → Status: COMPLETED (per proposal)
  ✗ No agreement after 14 days → Escalate to Level 3
```

### Level 3: DAO Court / Arbitration Council

**Status:** [ACTIVE] — Multi-member operation via ADR-126 M2

```
Dispute escalated (status: COUNCIL_QUEUE)
  → Quorum config snapshotted from DaoConfig
  → First verdict → status: IN_REVIEW
  → Quorum met → status: RESOLVED, escrow transitions

Multi-member Council (M2):
  → Arbitrators invited by admin (PENDING → ACTIVE via Kind 30021 credential)
  → Quorum: SINGLE | MAJORITY | THRESHOLD (configurable)
  → Each arbitrator can submit one verdict per dispute (immutable)
  → Dissent is public and permanent (audit trail)
  → Tie-break: admin decisive vote (M2 only)
```

**Level 3 Status State Machine:**
```
OPEN → COUNCIL_QUEUE → IN_REVIEW → RESOLVED
                 ↓            ↓
             WITHDRAWN   (tie → IN_REVIEW awaiting admin)
```

---

## What This System Cannot Do

| Limitation | Reason |
|-----------|--------|
| Not a legal system | No state-enforceable ruling |
| No physical enforcement | Only Lightning Escrow can be split |
| No protection against new wallet | Banned user can create a new wallet |
| No guarantee with bad escrow | If escrow was not set up correctly |

**Consequence:** The system works best when both parties cooperate voluntarily
and reputation matters to them.

---

## Implementation Requirements

| Component | Dependency |
|-----------|------------|
| Dispute status in API | Phase 1 (already present: `OPEN`, `COUNCIL_QUEUE`, `IN_REVIEW`, `RESOLVED`, `WITHDRAWN`) |
| Automatic timeout | Phase 2 (Lightning Escrow) |
| Mediator pool | Phase 3 (Reputation System) |
| Nostr DM (NIP-04) | Phase 3 |
| Arbitrator election | Phase 4 (Governance) |
| Verdict enforcement | Implemented (M2: escrow triggers on quorum) |

---

## Example Use Case

```
Context: Design service order. Buyer pays 50,000 sats escrow.
Situation: Designer delivers, buyer claims quality is unacceptable.

Level 1 (Days 1–7):
  → Buyer opens dispute
  → Parties communicate. No agreement.
  → Cooling-off expires.

Level 3 (M2 — no Level 2 in M2):
  → Admin invites second arbitrator; invitee accepts
  → Config set to THRESHOLD(2)
  → Arbitrator A votes RELEASE (seller delivered)
  → Arbitrator B votes CANCEL (quality issues)
  → Tie — admin tie-break: RELEASE
  → 50,000 sats to seller, audit trail shows dissent
  → Nostr Event published: Dispute #xyz → Released to seller
```
PATCH /api/tickets/:id
{ "status": "DISPUTED", "disputeReason": "Item never arrived after 7 days" }
```

---

## 3 Levels

### Level 1: Cooling-off Period (7 days)

```
Dispute opened
  → Ticket status: DISPUTED
  → 7-day window starts
  → Both parties notified (Nostr DM or in-app)

Communication via Nostr DM (NIP-04, end-to-end encrypted)
Parties can reach a direct agreement:

Possible outcomes:
  ✓ Buyer withdraws dispute              → Status: COMPLETED
  ✓ Seller offers partial refund         → Agreement, Status: COMPLETED
  ✗ No agreement after 7 days            → Escalate to Level 2
```

### Level 2: Mediation (14 days)

```
Community mediator elected from verified pool
  → Requires: Verification Ticket (Phase 3), reputation score above threshold
  → No stake in either disputing party

Mediator can view:
  → All ticket events (timestamps, status changes)
  → Evidence voluntarily submitted by parties (photos, text)
  → Chat log (only with party consent)

Mediator proposes resolution (NOT binding)
  → Both parties must actively agree

Possible outcomes:
  ✓ Both accept proposal   → Status: COMPLETED (per proposal)
  ✗ No agreement after 14 days → Escalate to Level 3
```

### Level 3: DAO Court / Arbitration Council (24–48 hours)

**Status:** [BOOTSTRAP] — Operative via ADR-125 M0 (single admin arbitrator, 1P1V bypass)

> In bootstrap mode, Level 2 (Mediator) is not yet implemented. Disputes proceed directly from Level 1 (cooling-off) to Level 3 (Council ruling).

```
Single arbitrator (FOUNDATION_DAO_ADMIN_PUBKEY in bootstrap mode)
  → Issues verdict: RELEASE or CANCEL (SPLIT deferred)
  → Published as Nostr Kind 30022 (sub_type: arbitration_verdict)
  → Escrow transition enforced via verifyActor(ARBITRATOR)

Full Codex (Phase 4+):
  3–5 arbitrators elected via Governance Vote
  HID Level 3 required
  COLA stake required
  SPLIT outcome supported
  2% arbitrator fee from escrow
```

---

## What This System Cannot Do

| Limitation | Reason |
|-----------|--------|
| Not a legal system | No state-enforceable ruling |
| No physical enforcement | Only Lightning Escrow can be split |
| No protection against new wallet | Banned user can create a new wallet |
| No guarantee with bad escrow | If escrow was not set up correctly |

**Consequence:** The system works best when both parties cooperate voluntarily
and reputation matters to them.

---

## Implementation Requirements

| Component | Dependency |
|-----------|-----------|
| Dispute status in API | Phase 1 (already present: `DISPUTED`) |
| Automatic timeout | Phase 2 (Lightning Escrow) |
| Mediator pool | Phase 3 (Reputation System) |
| Nostr DM (NIP-04) | Phase 3 |
| Arbitrator election | Phase 4 (Governance) |
| Verdict enforcement | Phase 4 (Escrow + Governance combined) |

---

## Example Use Case

```
Context: Pizza order. Buyer pays 5,000 sats escrow.
Situation: Pizza never arrives, seller claims delivery was successful.

Level 1 (Days 1–7):
  → Buyer opens dispute
  → Parties communicate. No agreement.

Level 3 (Days 8–10, BOOTSTRAP — no Level 2):
  → Single arbitrator (admin) reviews all ticket events on Nostr
  → Majority verdict: buyer is right (no delivery proof).
  → 4,900 sats to buyer, 100 sats arbitrator fee (pro bono in bootstrap)
  → Nostr Event published: Dispute #xyz → Buyer won
```


---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
