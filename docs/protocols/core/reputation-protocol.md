# Reputation Protocol – COL-Points and Review System

**Version:** 1.0.0-draft
**Date:** 2026-03-22
**Status:** [PHASE 3]

---

## Overview

The Colabonate reputation system is a **dual-track model**:

1. **COL-Points** — Off-chain, non-transferable quantitative score earned through protocol participation
2. **Nostr Reviews** — On-chain (Nostr relay), qualitative 5-star reviews published as Kind 30024 events

Together these form a **Reputation Score** (0.0–5.0) that is publicly visible, manipulation-resistant, and bound to a user's Nostr pubkey (soulbound).

### Design Principles

- **Non-transferable** — reputation cannot be bought, sold, or transferred
- **Non-deletable** — reviews are permanent Nostr events; they cannot be retracted
- **Manipulation-resistant** — minimum thresholds, identity-level weighting, and mutual review requirements prevent spam and fake reviews
- **Pubkey-bound** — reputation is tied to a Nostr pubkey; changing pubkeys resets reputation
- **Transparent** — all reputation data is queryable from public Nostr relays

---

## COL-Points

COL-Points are an **off-chain, non-transferable accumulation metric**. They represent total verified participation in the Colabonate protocol.

### Earning Rules

| Action | COL-Points Awarded | Conditions |
|--------|-------------------|------------|
| Complete a ticket (as seller) | 10 points | Ticket reaches `COMPLETED` status |
| Complete a ticket (as buyer) | 5 points | Ticket reaches `COMPLETED` status |
| Submit a mutual review | 3 points | Both parties submit review within window |
| Governance vote | 2 points | Vote published for a completed proposal |
| HID Level 2 upgrade | 50 points | One-time, on successful peer verification |
| HID Level 3 upgrade | 100 points | One-time, on successful Humanode verification |
| Verifier completion | 20 points | After successfully verifying a Level 2 candidate |
| Protocol contribution | TBD | Awarded by Foundation DAO governance vote |

### Properties

- **Minimum cap:** No minimum — all participants start at 0
- **Maximum cap:** None in Phase 3. The DAO may introduce a soft cap in Phase 5.
- **Decay:** No automatic decay in Phase 3. The DAO may introduce time-based decay in Phase 5.
- **Not transferable:** COL-Points cannot be sent, sold, or assigned to another pubkey
- **Not COLA:** COL-Points are distinct from COLA governance tokens. See [economic-protocol.md](../governance/economic-protocol.md)

### Storage

COL-Points are stored:
1. **Locally** — in the Colabonate application database (authoritative for Phase 1–3)
2. **Published** — as Kind 30024 Nostr events (the current point total is included in review events)

In Phase 4, COL-Points migrate to Nostr-only storage to enable cross-platform portability.

---

## Nostr Reviews (Kind 30024)

Reviews are qualitative assessments published as Nostr events after ticket completion. They are the primary public reputation signal.

### Review Trigger

A review window opens when a ticket reaches `COMPLETED` status. Both parties (buyer and seller) can publish a review.

**Review window:** 14 days from the `COMPLETED` status timestamp. After 14 days, the review window closes and no new reviews for that ticket are accepted.

### Mutual Review Requirement

COL-Points for reviews are only awarded if **both parties submit a review**. This prevents one-sided gaming where only the winning party leaves feedback.

- If only one party reviews: Both receive no COL-Points for the review action. The single review is still published and affects reputation.
- If both review: Both receive 3 COL-Points each.

### Review Event Schema

See [nostr-events.md](./nostr-events.md) — Kind 30024 for the full schema.

**Key fields:**

```json
{
  "kind": 30024,
  "pubkey": "<reviewer-pubkey>",
  "tags": [
    ["d", "<review-id>"],
    ["review_for", "<reviewed-pubkey>"],
    ["ticket", "<ticket-id>"],
    ["stars", "4"],
    ["col_points", "125"]
  ],
  "content": "<review-text-max-2048-chars>"
}
```

### Review Validation Rules

| Rule | Details |
|------|---------|
| Ticket must be `COMPLETED` | Cannot review a ticket in any other status |
| Within 14-day window | Reviews after window are rejected |
| Reviewer must be a ticket participant | Only buyer or seller can review; no third-party reviews |
| Minimum activity threshold | Reviewer must have ≥ 1 prior completed ticket for reviews to receive weight |
| Identity level weighting | Level 3 reviews carry 1.5× weight in score computation |
| One review per party per ticket | Duplicate reviews for the same ticket from the same pubkey are ignored |

---

## Reputation Score Computation

The **Reputation Score** is a float from 0.0 to 5.0, computed from four components:

```
Reputation Score = (
  star_average × 0.50
  + completion_rate × 5.0 × 0.25
  + (1 - dispute_loss_rate) × 5.0 × 0.15
  + col_points_factor × 0.10
) × identity_level_multiplier
```

### Component Definitions

**star_average** (0.0–5.0)
- Weighted average of all `stars` values in Kind 30024 events where `review_for` = this pubkey
- Level 3 reviewers: weight = 1.5; Level 2: weight = 1.2; Level 0/1: weight = 1.0
- Minimum 3 reviews before star_average affects score (below 3 reviews: star_average treated as 3.0)

**completion_rate** (0.0–1.0)
- `tickets_completed / tickets_accepted`
- A seller who accepts tickets but never completes them has a low completion rate
- Minimum 5 accepted tickets before completion_rate affects score (below 5: treated as 1.0)

**dispute_loss_rate** (0.0–1.0)
- `disputes_lost / disputes_participated`
- Losing a dispute (Level 3 DAO verdict against you) increases this rate
- Minimum 1 dispute before rate affects score (below 1: treated as 0.0)

**col_points_factor** (0.0–1.0)
- `min(col_points, 1000) / 1000` — capped at 1000 points for this component
- Full impact requires 1000 COL-Points

**identity_level_multiplier**
- Level 0: 0.80 (20% penalty for unverified identity)
- Level 1: 0.90
- Level 2: 1.00 (baseline)
- Level 3: 1.10 (10% trust bonus)

### Example Computation

A seller with:
- 4.2 star average (Level 2 reviewers only)
- 0.92 completion rate (23/25 tickets completed)
- 0.05 dispute loss rate (1 loss in 20 disputes)
- 350 COL-Points
- Identity Level 2

```
Score = (4.2 × 0.50) + (0.92 × 5.0 × 0.25) + ((1-0.05) × 5.0 × 0.15) + (0.35 × 0.10)
      = 2.10 + 1.15 + 0.7125 + 0.035
      = 3.9975
      × 1.00 (Level 2 multiplier)
      ≈ 4.0 / 5.0
```

---

## Public Visibility

All reputation data derived from Nostr events is publicly visible to anyone who queries the relay:

| Data | Visible | Privacy Notes |
|------|---------|--------------|
| Star average | Yes | Aggregated; individual reviews traceable |
| Review text | Yes | Permanent; reviewer pubkey visible |
| COL-Points total | Yes | Published in Kind 30024 events |
| Reputation score | Yes | Computable from public events |
| Ticket completion rate | Yes | Count only; ticket details may be pseudonymous |
| Dispute outcomes | Yes | Verdict published as Nostr event (Level 3) |
| Identity level | Yes | Kind 30021 credential is public |

**What is NOT public:**
- Raw biometric data (never leaves device)
- Full identity document details (not collected in protocol)
- Private Nostr DM content

---

## Soulbound Guarantee

Reputation is bound to a Nostr pubkey through the mechanism of Nostr event signatures:
- All Kind 30024 (review) events are signed by the reviewer's private key
- All Kind 30021 (credential) events are signed by the subject's private key
- Since the Colabonate protocol never takes custody of private keys, a reputation "transfer" would require transferring the private key itself — which defeats the purpose of the reputation system

**Key compromise procedure:** If a user's private key is compromised:
1. User publishes a NIP-38 key migration event from their old pubkey to a new pubkey (requires access to old key)
2. Foundation DAO can issue an emergency migration credential if old key is lost (requires Level 2 in-person verification of identity continuity)
3. COL-Points are migrated by the protocol; Nostr review events remain on the old pubkey (not migrated)
4. Reputation score resets for the new pubkey — this is a known limitation and incentive to protect private keys

---

## Mediator Eligibility

The mediator pool (Phase 3+) requires:

| Requirement | Value |
|------------|-------|
| Reputation score | ≥ 4.0 / 5.0 |
| Identity level | ≥ Level 2 |
| Completed tickets | ≥ 10 |
| Active dispute loss rate | ≤ 0.10 (max 10% disputes lost) |
| Opt-in credential | Must publish Kind 30021 `credential_type: verifier_pool_member` |

Mediators are randomly selected from the eligible pool for each dispute. Selection algorithm:
1. Filter pool by eligibility criteria above
2. Exclude mediators who have completed tickets with either disputing party within 180 days
3. Random selection from remaining pool (verifiable randomness source: DAO-governed)

Mediators receive **1% of the disputed ticket value** as compensation (deducted from escrow).

---

## Manipulation Resistance

### Fake Review Prevention

| Attack | Mitigation |
|--------|-----------|
| Create multiple accounts and review each other | Minimum threshold (≥1 prior completed ticket); Level 0 reviews carry 1.0× weight only |
| Pay for positive reviews | Reviews require completed tickets; ticket escrow requires real Bitcoin payment |
| Downvote competitors | Downvotes require real ticket history; dispute loss rate visible |
| Review bomb (many 1-star reviews) | Minimum threshold; weight proportional to reviewer's COL-Points |

### Identity-Level Weighting

Higher-identity-level reviewers carry more weight. This creates an incentive for legitimate, long-term participants to build Level 2/3 identity — and makes it costly to create fake high-reputation accounts for review manipulation (each fake account would require real in-person verification).

### Minimum Ticket Threshold

Reviews from pubkeys with zero prior completed tickets receive no weight in score computation. They are published but marked `unweighted` in client display. This prevents day-zero fake accounts from impacting reputation.

---

## Reputation Impact of Sanctions

The DAO governance system can apply sanctions that directly affect reputation:

| Sanction | Reputation Impact |
|---------|------------------|
| Warning | No direct score impact |
| Temporary restriction | Dispute loss rate increases by 0.1 per restriction |
| Protocol ban | Account marked as `SANCTIONED` in Kind 30021 credential; score visible but account cannot participate |
| Arbitration verdict against | Dispute loss rate increases; COL-Points deducted (amount per DAO Codex) |

---

## Phase Roadmap

| Phase | Feature |
|-------|---------|
| Phase 1 (current) | No reputation system; tickets tracked only |
| Phase 2 | COL-Points tracking in-app; no Nostr publication |
| Phase 3 | Full system: Kind 30024 reviews, Level 1-2 identity, reputation score computation |
| Phase 4 | Mediator pool using reputation threshold; governance voting weighted by reputation |
| Phase 5 | Protocol marketplace reputation (protocol author ratings, royalty distribution weight) |

---

## References

- [docs/protocols/core/nostr-events.md](./nostr-events.md) — Kind 30024 schema
- [docs/protocols/identity/identity-protocol.md](../identity/identity-protocol.md) — Identity levels
- [docs/protocols/governance/economic-protocol.md](../governance/economic-protocol.md) — COL-Points vs COLA Token
- [docs/protocols/governance/dao-codex.md](../governance/dao-codex.md) — Sanction procedures

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
