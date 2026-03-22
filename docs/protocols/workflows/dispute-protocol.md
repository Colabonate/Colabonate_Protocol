# Dispute Protocol

**Status:** [PHASE 4] — Concept finalized, implementation after Phase 2+3
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

### Level 3: DAO Court (24–48 hours)

```
3–5 arbitrators elected via Governance Ticket
  → Requires: Governance Protocol (Phase 4), staking deposit

Arbitrators analyze:
  → All ticket events (public on Nostr)
  → Evidence from both parties
  → Level 2 mediation protocol

Vote (anonymous, majority decision):
  → 24–48 hour window
  → Verdict published as Nostr Event (public, immutable)

Enforcement via Lightning Escrow:
  → Escrow split according to verdict
  → Arbitrator fee: ~2% from escrow reserve (split among arbitrators)

Reputation impact:
  → Losing party in a frivolous dispute: reputation penalty
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

Level 2 (Days 8–21):
  → Mediator (reputation score: 850) elected
  → Buyer shares screenshot of empty doorbell area
  → Seller shares GPS log of delivery driver
  → Mediator proposes 50/50 (unclear evidence)
  → Both agree → Buyer receives 2,500 sats back

Alternative (if no agreement):
Level 3 (Days 22–24):
  → 3 arbitrators analyze. Majority: buyer is right (no GPS proof at door).
  → 4,900 sats to buyer, 100 sats arbitrator fee (split)
  → Nostr Event published: Dispute #xyz → Buyer won
```
