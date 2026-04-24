# Colabonate Compatibility Checklist

**Normativity:** Normative
**Version:** 1.0.0-draft
**Date:** 2026-04-18
**Purpose:** What a third-party client must implement to be compatible with the Colabonate Protocol open standard.

---

## Quick Start

To build a Colabonate-compatible client, implement these core features:

- [ ] Nostr event support (NIP-01: signing, relay subscription)
- [ ] Event kinds 30017–30027, 30414–30415, 30420–30422
- [ ] Lightning Hold Invoice handling (NIP-03 or similar)
- [ ] Ticket state machine (PENDING → ACCEPTED → IN_PROGRESS → COMPLETED, with disputes)
- [ ] Dispute escalation protocol (LEVEL_1/2/3 → arbitration)
- [ ] Arbitration outcome handling (ARBITRATION_WON / ARBITRATION_LOST as terminal)

---

## Event Kinds — The Mandatory Set

A Colabonate-compatible client must:

1. **Publish & subscribe to the following Nostr event kinds:**

   | Kind | Name | Layer | Must-Implement |
   |------|------|-------|---|
   | 30017 | Offer | Marketplace | ✅ YES (or 30402 NIP-99 equivalent) |
   | 30018 | Ticket Created | Transaction | ✅ YES (or 30407 NIP-99 equivalent) |
   | 30019 | Ticket Status | Transaction | ✅ YES (or 30408 NIP-99 equivalent) |
   | 30020 | Dispute Opened | Dispute | ✅ YES |
   | 30021 | Verification Credential | Identity | ✅ YES (for arbitrator credentials at minimum) |
   | 30022 | Governance Vote | Governance | ✅ YES (for arbitration verdicts; Kind 30022 content sub_type=arbitration_verdict) |
   | 30024 | Reputation Score | Reputation | ⚠️ OPTIONAL (for review/rating features) |
   | 30027 | Company Profile | Marketplace | ⚠️ OPTIONAL (for company listings) |
   | 30414 | Cooperation Proposal | Cooperation | ⚠️ OPTIONAL (for milestone-based work) |
   | 30415 | Milestone Event | Cooperation | ⚠️ OPTIONAL (depends on 30414) |
   | 30420 | DAO Profile | Governance | ✅ YES (to read arbitration council info) |
   | 30421 | DAO Codex | Governance | ✅ YES (to read governance rules) |
   | 30422 | DAO Membership | Governance | ✅ YES (to verify arbitrator roles) |

   **NIP-99 Dual-Publishing:** Clients may subscribe to either legacy kinds (30017–30019) or NIP-99 kinds (30402, 30407, 30408). The Colabonate protocol will dual-publish both during the transition period. Clients should handle both.

2. **Validate event structure per [nostr-events.md](./nostr-events.md):**
   - Correct tags (e.g., Kind 30019 must have `ticket`, `status`, `buyer`, `seller` tags)
   - Valid Schnorr signatures (secp256k1)
   - Required vs. optional tag handling

---

## Ticket State Machine — The Core Flow

A compatible client must **observe and respect** this state machine:

```
[PENDING]
  ↓ (seller accepts)
[ACCEPTED]
  ├→ (SERVICE offer) [IN_PROGRESS] (buyer pays, starts work)
  └→ (PRODUCT offer) [PAID] (buyer pays, escrow held)
      ↓ (buyer confirms delivery)
      [IN_PROGRESS] (work/delivery in progress)
  ↓ (delivery confirmed)
[COMPLETED] (terminal ✓)
  ↓ (review & reputation published as Kind 30024)

[PENDING] → [CANCELLED] (either party) (terminal ✗)
[IN_PROGRESS] → [DISPUTED] (either party)
  ↓ (dispute resolution via arbitration)
  [ARBITRATION_WON] (seller wins, escrow released) (terminal ✓)
  [ARBITRATION_LOST] (buyer wins, escrow cancelled) (terminal ✗)
```

**Minimum implementation:**
- [ ] Display tickets in all states
- [ ] Transition PENDING → ACCEPTED (seller action)
- [ ] Transition ACCEPTED → IN_PROGRESS or PAID (buyer action, depends on OfferType)
- [ ] Transition IN_PROGRESS → COMPLETED (buyer action)
- [ ] Transition IN_PROGRESS → DISPUTED (either party)
- [ ] Observe ARBITRATION_WON / ARBITRATION_LOST (from arbitrator Kind 30022 vote)
- [ ] Mark COMPLETED, CANCELLED, ARBITRATION_* as terminal states (no further transitions)

**Authorization:** Every state transition is signed by the publishing pubkey. Validate:
- PENDING → ACCEPTED: only seller pubkey can publish
- ACCEPTED → IN_PROGRESS: only buyer pubkey can publish
- ACCEPTED → PAID: only buyer pubkey can publish (PRODUCT offers only)
- IN_PROGRESS → COMPLETED: only buyer pubkey can publish
- IN_PROGRESS → DISPUTED: buyer OR seller pubkey
- DISPUTED → ARBITRATION_*: only DAO arbitrator pubkey (verify via Kind 30422 membership)

---

## Lightning Escrow — The Payment Engine

A compatible client must:

1. **Understand Hold Invoices** ([ADR-121](../../decisions/121-lightning-payment-gate-escrow.md)):
   - Buyer pays a Lightning Hold Invoice (HTLC-locked funds)
   - Funds are held (not released) until seller completes delivery
   - Invoice expiry auto-cancels the ticket (funds return to buyer)
   - Only when buyer confirms delivery can seller claim funds

2. **Respect Lightning-level timeouts** ([ADR-131](../../decisions/131-timeout-policy-app-vs-lightning.md)):
   - The authoritative timeout is the Hold Invoice expiry (on the Lightning Network)
   - App-level timeouts (e.g., "Pay by [date]") are informative only
   - When an invoice expires, the ticket auto-transitions to CANCELLED
   - **Do NOT** implement app-level timer logic; use webhook notifications or relay-polled status

3. **Validate payment proof:**
   - Ticket transitions to PAID when invoice is held (not immediately when payment begins)
   - Transition to IN_PROGRESS only after buyer confirms receipt/delivery has started
   - COMPLETED only when buyer explicitly confirms

**Integration model:**
- Fetch Invoice from Colabonate server (or directly from Lightning provider)
- Call `lnd` / `lnbits` / Lightning wallet to pay the invoice
- Subscribe to Kind 30019 (Ticket Status) events; transition appears as a published event
- Monitor invoice status via relay or webhook (when it expires)

---

## Dispute & Arbitration — The Resolution Path

A compatible client must:

1. **Observe dispute escalation:**

   | Level | Duration | Agents | Resolution |
   |-------|----------|--------|-----------|
   | LEVEL_1 | 7 days | Buyer + Seller (direct negotiation) | Either party can propose a settlement or escalate |
   | LEVEL_2 | 14 days | Community mediator + parties | Mediator proposes resolution |
   | LEVEL_3 | 24–48h | DAO arbitration council | Arbitrators vote; verdict is binding |

2. **Publish/subscribe to Kind 30020 (Dispute Opened):**
   - Reason tag: `dispute_reason` one of `non_delivery`, `quality`, `non_payment`, `terms_violation`, `other`
   - Status field tracks escalation level
   - Either party can escalate after timeout expires

3. **Monitor arbitration verdicts via Kind 30022 (Governance Vote):**
   - `sub_type: arbitration_verdict` in content JSON
   - `outcome: RELEASE | CANCEL | SPLIT`
   - Transition ticket to ARBITRATION_WON (RELEASE outcome) or ARBITRATION_LOST (CANCEL outcome)
   - Dissenting votes are retained; client may expose full voting record for transparency

4. **Escrow resolution:**
   - ARBITRATION_WON → escrow is released to seller (settle Lightning payment)
   - ARBITRATION_LOST → escrow is cancelled to buyer (refund via Lightning HTLC timeout or explicit release)
   - SPLIT → perform proportional distribution (Colabonate server handles; client reflects in UI)

---

## Identity & Reputation — Optional but Recommended

### For arbitrators:

- [ ] Verify arbitrator credentials via Kind 30021 (`credential_type: arbitrator`) from the DAO
- [ ] Check Kind 30422 membership record to confirm active arbitrator role

### For buyer/seller trust:

- [ ] Display reputation score (Kind 30024: `score` tag, `review_for` tag)
- [ ] Show identity verification level (Kind 30021: `hid_level` tag)
- [ ] Link to peer reviews (query relays for Kind 30024 events with `review_for: <pubkey>`)

---

## Signing & Authentication

A compatible client must:

1. **Support Nostr key management:**
   - [ ] NIP-07 (browser extension for signing)
   - [ ] OR local key storage + client-side signing
   - [ ] OR ephemeral key (signed by parent key, for throwaway clients)

2. **Sign all published events correctly:**
   - Event `pubkey` matches signing key
   - Schnorr signature valid (secp256k1)
   - Event `created_at` is Unix timestamp
   - All required tags present before signing

3. **For LNURL-Auth** (login):
   - Implement NIP-118 (or Colabonate's LNURL-Auth variant)
   - Sign the `k1` challenge with your Nostr key
   - Receive `access_token` from server (if using REST API convenience layer)

---

## Relay Subscription Strategy

A compatible client should:

1. **Subscribe to multiple relays** (decentralization):
   - Gossip: wss://relay.primal.net (or similar)
   - Colabonate-friendly: wss://relay.colabonate.net (if exists)
   - Fallback: wss://nos.lol

2. **Query filters to find events:**
   - By kind: `{"kinds": [30017, 30019, 30020]}`
   - By author: `{"kinds": [30019], "authors": ["<my-pubkey>"]}`
   - By tags: `{"kinds": [30019], "#ticket": ["<ticket-id>"]}`
   - By `since` / `until` for historical queries

3. **Cache & deduplicate:**
   - Deduplicate events from multiple relays (by event ID)
   - Cache recent events locally to avoid re-querying
   - Listen for replacements (Kind 30017–30027 are replaceable; newer `created_at` wins)

---



## Testing Your Implementation

Use this checklist to verify compatibility:

### Minimum Viable Implementation (MVP)

- [ ] Can publish Kind 30017 (Offer) with correct tags
- [ ] Can publish Kind 30018 (Ticket Created) and subscribe to it
- [ ] Can publish Kind 30019 (Ticket Status) state transitions
- [ ] Can observe a complete ticket lifecycle (PENDING → ACCEPTED → IN_PROGRESS → COMPLETED)
- [ ] Validates Schnorr signatures on all events
- [ ] Respects state machine authorization (seller-only transitions, buyer-only transitions)

### Full Protocol Support

- [ ] Supports all event kinds 30017–30027, 30414–30415, 30420–30422
- [ ] Handles disputes: can open Kind 30020 and observe escalation
- [ ] Handles arbitration: can observe Kind 30022 verdict and update ticket to ARBITRATION_WON/LOST
- [ ] Integrates Lightning Hold Invoices (payment flow)
- [ ] Implements LNURL-Auth if offering login
- [ ] Fallback handling: graceful degradation if unsupported event kind encountered
- [ ] Multi-relay subscription: queries multiple relays for redundancy

### Interoperability Test

- [ ] Can create a ticket against an offer from another Colabonate-compatible client
- [ ] Can accept a ticket from another client and transition states
- [ ] Ticket events are visible on independent Nostr relay queries (not locked to one server)

---

## Reference

- **[nostr-events.md](./nostr-events.md)** – Event schemas
- **[ticket-system.md](./ticket-system.md)** – State machine details
- **[dispute-protocol.md](../workflows/dispute-protocol.md)** – Dispute escalation
- **[escrow-protocol.md](./escrow-protocol.md)** – Lightning Hold Invoice mechanics
- **[ADR-121](../../decisions/121-lightning-payment-gate-escrow.md)** – Escrow design
- **[ADR-131](../../decisions/131-timeout-policy-app-vs-lightning.md)** – Timeout authority
- **[NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md)** – Nostr protocol
- **[NIP-99](https://github.com/nostr-protocol/nips/blob/master/99.md)** – Classified Listings

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
