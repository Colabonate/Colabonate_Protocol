# Escrow Protocol – Lightning Escrow for Trustless Trade

**Version:** 1.0.0-draft
**Date:** 2026-03-22
**Status:** [PHASE 2] Escrow | [IMPLEMENTED] Phase 1 single invoice

---

## Overview

The Colabonate Escrow Protocol defines the mechanism for **trustless payment handling** in decentralized trade and cooperation. It ensures that:

- Buyers only release full payment after confirmed delivery
- Sellers receive guaranteed payment for completed work
- Neither party can unilaterally steal funds
- Disputes can be resolved by a third party without either party controlling the funds

The protocol uses **Bitcoin Lightning Network** as the payment rail and **LNBits Hold Invoices** as the escrow mechanism in Phase 2. For complex multi-party or conditional release scenarios (Phase 4+), RSK (Rootstock) smart contracts provide an alternative.

---

## Why Three-Phase Escrow

A simple "pay then deliver" model requires trust in the seller. A simple "deliver then pay" model requires trust in the buyer. The three-phase escrow model distributes risk:

| Phase | Amount | Purpose | Who Controls Release |
|-------|--------|---------|---------------------|
| Phase 1 | 25% | Reservation / commitment deposit | Seller (accepts ticket) |
| Phase 2 | 50% | Delivery payment | Buyer (confirms delivery received) |
| Phase 3 | 25% | Final payment / quality confirmation | Buyer (confirms quality) |

This structure means:
- Seller has 25% committed before starting work → incentive to start
- Buyer confirms delivery before 50% is released → protection against non-delivery
- The final 25% acts as a quality incentive → seller completes work well
- No single party controls the full amount at any time during the transaction

---

## Phase 1: Single Invoice (Implemented)

**Status:** [IMPLEMENTED]

The current Phase 1 implementation uses a standard Lightning invoice without escrow:

```
1. Ticket created (buyerPubkey, offerId, amountSats)
   → Lightning invoice generated (standard BOLT11)
   → Status: PENDING

2. Buyer pays invoice in Lightning wallet
   → Webhook: POST /api/lightning/webhook
   → Status: IN_PROGRESS

3. Completion
   → PATCH /api/tickets/:id { status: "COMPLETED" }
   → No escrow: payment is final upon webhook confirmation
```

**Limitation:** In Phase 1, the buyer has no recourse after paying. The seller could theoretically not deliver. This is acceptable for the MVP because the protocol recommends low-value initial transactions and the dispute flag (`DISPUTED` status) is available.

---

## Phase 2: Three-Phase Lightning Escrow

**Status:** [PHASE 2]

### LNBits Hold Invoice Mechanism

A **Hold Invoice** is a BOLT11 Lightning invoice where the receiving node does not automatically settle the payment. Instead, the payment is "held" in the routing path until the invoice holder explicitly settles or cancels it.

This enables escrow: the buyer pays, but the funds are held in transit until the seller settles (indicating delivery) and the buyer confirms.

**How Hold Invoices differ from standard invoices:**

| Property | Standard Invoice | Hold Invoice |
|---------|-----------------|-------------|
| Settlement | Automatic on payment | Manual by invoice holder |
| Cancellation | Not possible after payment | Possible until settlement |
| Fund location | Moved to payee immediately | Held in routing path (HTLC) |
| Timeout | Invoice expiry only | Invoice expiry OR hold timeout |
| Use case | Immediate payment | Conditional payment (escrow) |

### Escrow State Machine

```
                    ┌─────────────────────┐
                    │     INITIATED        │  Ticket created, escrow requested
                    └──────────┬──────────┘
                               │ Seller accepts ticket
                               ▼
                    ┌─────────────────────┐
                    │  PHASE_1_PENDING     │  25% Hold Invoice generated
                    └──────────┬──────────┘
                               │ Buyer pays Phase 1
                               ▼
                    ┌─────────────────────┐
                    │     RESERVED         │  25% held in Lightning HTLC
                    └──────────┬──────────┘
                               │ Seller starts work (signals start)
                               ▼
                    ┌─────────────────────┐
                    │  PHASE_2_PENDING     │  50% Invoice generated
                    └──────────┬──────────┘
                               │ Buyer pays Phase 2
                               ▼
                    ┌─────────────────────┐
                    │  DELIVERY_STARTED    │  75% held/transferred
                    └──────────┬──────────┘
                               │ Seller delivers & confirms delivery
                               ▼
                    ┌─────────────────────┐
                    │  PHASE_3_PENDING     │  25% Invoice generated
                    └──────────┬──────────┘
                               │ Buyer pays Phase 3
                               ▼
                    ┌─────────────────────┐
                    │  DELIVERY_CONFIRMED  │  100% held/pending confirmation
                    └──────────┬──────────┘
                               │ Buyer confirms quality (or auto-release on timeout)
                               ▼
                    ┌─────────────────────┐
                    │      RELEASED        │  All funds settled to seller ✓
                    └─────────────────────┘

    At any PENDING state → timeout → CANCELLED (funds returned to buyer)
    At DELIVERY_STARTED or DELIVERY_CONFIRMED → dispute → DISPUTED
```

### Valid State Transitions

| From | To | Trigger | Actor |
|------|-----|---------|-------|
| INITIATED | PHASE_1_PENDING | Seller accepts ticket | Seller |
| PHASE_1_PENDING | RESERVED | Buyer pays Phase 1 invoice | Buyer (Lightning payment) |
| PHASE_1_PENDING | CANCELLED | Phase 1 timeout (72h default) | Protocol (auto) |
| RESERVED | PHASE_2_PENDING | Seller signals work start | Seller |
| PHASE_2_PENDING | DELIVERY_STARTED | Buyer pays Phase 2 invoice | Buyer |
| PHASE_2_PENDING | CANCELLED | Phase 2 timeout (configurable) | Protocol (auto) |
| DELIVERY_STARTED | PHASE_3_PENDING | Seller confirms delivery | Seller |
| PHASE_3_PENDING | DELIVERY_CONFIRMED | Buyer pays Phase 3 invoice | Buyer |
| PHASE_3_PENDING | CANCELLED | Phase 3 timeout | Protocol (auto) |
| DELIVERY_CONFIRMED | RELEASED | Buyer confirms quality | Buyer |
| DELIVERY_CONFIRMED | RELEASED | Auto-release timeout (48h) | Protocol (auto) |
| DELIVERY_STARTED | DISPUTED | Either party raises dispute | Buyer or Seller |
| DELIVERY_CONFIRMED | DISPUTED | Either party raises dispute | Buyer or Seller |
| DISPUTED | RELEASED | DAO verdict: release to seller | Arbitrator |
| DISPUTED | CANCELLED | DAO verdict: refund to buyer | Arbitrator |

### Timeout Parameters

| Timeout | Default | Configurable | Notes |
|---------|---------|-------------|-------|
| Phase 1 payment window | 72 hours | No (protocol default) | Time for buyer to pay first invoice after seller accepts |
| Phase 2 payment window | Per offer | Yes (seller sets in offer) | Time from work-start signal to Phase 2 payment |
| Phase 3 payment window | 24 hours | No | After delivery confirmation |
| Quality confirmation | 48 hours | No | After Phase 3 payment; auto-release if buyer doesn't confirm |
| Dispute resolution | 7/14/24-48h | No (per dispute level) | See dispute-protocol.md |

### Auto-Cancellation

When any payment phase timeout fires:
1. The LNBits Hold Invoice expires (HTLC times out on Lightning routing nodes)
2. Funds return to the buyer's wallet automatically (HTLC failure path)
3. The ticket status transitions to `CANCELLED`
4. A Kind 30019 status update event is published by the protocol
5. COL-Points are NOT awarded for cancelled tickets
6. The seller receives no payment

---

## Escrow in Dispute

When a ticket enters `DISPUTED` status, the escrow funds are frozen:

### Fund Preservation During Dispute

The amounts already paid (Phase 1 hold, Phase 2 hold) remain in the Hold Invoice state:
- LNBits holds them via HTLC
- Neither party can unilaterally release or cancel during active dispute
- The dispute protocol (see [dispute-protocol.md](../workflows/dispute-protocol.md)) assigns a mediator or arbitrator

### Dispute Verdict to Escrow Mapping

| Verdict | Escrow Action |
|---------|--------------|
| Full refund (buyer wins) | Cancel all Hold Invoices; buyer receives full amount back |
| Partial refund (split) | Phase 1 settled to seller; Phase 2 split per verdict percentage |
| Full payment (seller wins) | All phases settled to seller |
| Mediation fee (Level 2) | 1% of ticket value deducted from escrow before distribution |
| Arbitration fee (Level 3) | 2% of ticket value deducted from escrow before distribution |

### Arbitrator Instruction Event

Level 3 arbitrators publish a verdict that includes escrow instructions:

```json
{
  "kind": 30019,
  "tags": [
    ["ticket", "<ticket-id>"],
    ["status", "RESOLVED"],
    ["escrow_action", "partial_refund"],
    ["seller_pct", "30"],
    ["buyer_pct", "68"],
    ["fee_pct", "2"]
  ]
}
```

The Colabonate server processes this event and executes the corresponding LNBits actions.

---

## Multi-Party Escrow (Phase 2+ Cooperation)

For cooperation tickets with multiple milestones and potentially multiple parties:

### Milestone-Based Escrow

Each milestone is an independent escrow:

```
Cooperation Ticket
├── Milestone 1: Design Phase
│   ├── Amount: 30% of total
│   ├── Escrow: Hold Invoice for Milestone 1 amount
│   └── Release: Buyer approves Milestone 1 deliverable
├── Milestone 2: Development Phase
│   ├── Amount: 50% of total
│   ├── Escrow: Hold Invoice for Milestone 2 amount
│   └── Release: Buyer approves Milestone 2 deliverable
└── Milestone 3: Final Delivery
    ├── Amount: 20% of total
    ├── Escrow: Hold Invoice for Milestone 3 amount
    └── Release: Buyer confirms final quality
```

Each milestone follows the same state machine as the main escrow, but scoped to the milestone amount and deliverable.

### N-Party Cooperation

When a cooperation involves multiple sellers (team projects):

- The team defines a multi-sig Lightning wallet or designates a team lead wallet
- All escrow payments go to the team wallet
- Internal distribution is outside the Colabonate protocol scope (handled by the team)
- Future Phase 5: protocol may define a standard for automatic team payment splitting

---

## RSK Sidechain Option (Phase 4)

For use cases requiring complex conditional logic not expressible with Hold Invoices, the RSK (Rootstock) sidechain provides an alternative escrow mechanism.

**When RSK is used instead of LNBits:**
- DAO governance votes requiring automated execution
- Royalty ticket distribution (automated percentage splits to protocol authors)
- Complex N-party cooperation with on-chain verifiable conditions

**How RSK relates to Bitcoin:**
- RSK is a Bitcoin sidechain — RBTC (RSK's native token) is 1:1 pegged to Bitcoin via a two-way peg
- RSK uses Ethereum-compatible smart contract language but runs on Bitcoin merge-mining security
- IMPORTANT: Using RSK does NOT mean using Ethereum. There is no ETH involved. RSK is Bitcoin-secured.

**Trade-offs vs LNBits:**

| Property | LNBits Hold Invoice | RSK Smart Contract |
|---------|--------------------|--------------------|
| Speed | Near-instant | 30-second block time |
| Cost | Routing fees only | Gas fees in RBTC |
| Complexity | Simple | High |
| Trust | LNBits node | RSK node + contract audit |
| Phase | Phase 2 | Phase 4 |

---

## Security Boundaries

| Rule | Rationale |
|------|-----------|
| Server holds no Bitcoin | LNBits Admin Key is a server-side environment variable only; frontend never receives it |
| No key custody | The Colabonate server never holds private keys for buyers or sellers |
| Hold Invoice server-side only | LNBits endpoints are called only from the server; browser JS cannot call them directly |
| Timeout enforced by Lightning | Hold Invoice expiry is enforced at the Lightning protocol level; the Colabonate server cannot prevent it |
| Dispute freeze is protocol-level | Once `DISPUTED` status is set, escrow can only be released by an authorized arbitrator event |
| Audit trail | All escrow state transitions are published as Nostr events (Kind 30019) — immutable and auditable |

---

## Phase 1 Backlog Items

The following items are known gaps in the current Phase 1 implementation that must be resolved before Phase 2 escrow:

- [ ] PATCH /api/offers/:id — offer close endpoint missing
- [ ] ACCEPTED status not yet activated (seller acceptance is not tracked in Phase 1)
- [ ] Phase 1 dispute flow needs webhook-compatible conflict registration

---

## References

- [BOLT11: Lightning Invoice Spec](https://github.com/lightning/bolts/blob/master/11-payment-encoding.md)
- [LNBits Hold Invoice API](https://lnbits.com/extension/boltz)
- [RSK / Rootstock](https://rootstock.io)
- [docs/protocols/workflows/buy-protocol.md](../workflows/buy-protocol.md) — buyer-side escrow flow
- [docs/protocols/workflows/sell-protocol.md](../workflows/sell-protocol.md) — seller-side escrow flow
- [docs/protocols/workflows/dispute-protocol.md](../workflows/dispute-protocol.md) — dispute resolution using escrow
- [docs/protocols/core/nostr-events.md](./nostr-events.md) — Kind 30019 status update events

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
