# DAO Creation Protocol – User-Created Community DAOs

**Version:** 1.0.0-draft
**Date:** 2026-03-22
**Status:** [PHASE 4]

---

## Overview

The Colabonate Protocol enables any qualified participant to create a **community DAO** with a custom Codex. This is distinct from the Colabonate Foundation DAO (which governs the protocol itself).

Community DAOs allow groups of participants to:
- Define shared rules for their marketplace or cooperation community
- Opt into stronger legal safety by binding transactions to their Codex
- Operate specialized marketplaces (e.g. "Berlin Freelancer DAO", "Open Source Contributor DAO")
- Create governance structures for their community (voting, treasury, dispute resolution)

**Key principle:** Community DAOs operate *within* the Colabonate Protocol — they extend and specialize it, they do not replace it. A transaction bound to a community DAO still follows all base Colabonate Protocol rules.

---

## Prerequisites for DAO Creation

All founding members must meet these requirements:

| Requirement | Value | Rationale |
|------------|-------|-----------|
| Identity level | Minimum Level 2 (peer verified) | Ensures real humans are founding the DAO |
| Founding members | Minimum 5 | Prevents single-person DAOs |
| COLA stake per founder | Minimum determined by Foundation DAO governance vote | Skin-in-the-game commitment |
| Active protocol history | At least 3 completed tickets per founder | Ensures founders understand the protocol |

These thresholds are protocol defaults. The Foundation DAO can adjust them via governance vote.

---

## Minimum Codex Requirements

Every community DAO Codex **must** include the following sections. A Codex missing any of these is invalid and the DAO registration will be rejected by the protocol.

### Required Sections

#### 1. Purpose and Scope
- What is the DAO's purpose?
- What types of interactions does it govern? (buy/sell, cooperation, or both)
- Geographic or domain restrictions (if any)

#### 2. Membership Criteria
- Who can join the DAO as a member?
- Minimum identity level required for members
- Any additional entry requirements

#### 3. Governance Rules
- Voting model used (1P1V, token-weighted, reputation-weighted, or hybrid)
- Quorum requirements for ordinary decisions
- Quorum requirements for Codex amendments
- Voting period durations

#### 4. Dispute Resolution Clause
- Which dispute resolution level(s) does the DAO offer? (Level 1 only, up to Level 2, or full Level 3)
- How are mediators/arbitrators selected within the DAO?
- What is the fee structure for DAO dispute resolution?

#### 5. Sanction Levels
- What behaviors are sanctionable?
- What sanctions are available? (Warning → Restriction → Temporary ban → Permanent ban)
- Who can propose sanctions?
- What is the appeal procedure?

#### 6. Codex Amendment Procedure
- How can the Codex be changed?
- What majority is required?
- What is the minimum notice period before a Codex amendment takes effect?

#### 7. Dissolution Clause
- Under what conditions can the DAO be dissolved?
- What happens to DAO treasury funds upon dissolution?

### Optional Sections (Recommended)

- Choice-of-law clause (which jurisdiction's law applies, if any)
- Treasury management rules
- Membership fee structure
- Protocol royalty sharing rules
- Integration with Foundation DAO governance

---

## Minimum Viable Codex Template

This template satisfies all minimum requirements. Founders should adapt it to their community's needs.

```markdown
# [DAO Name] Codex v1.0

## 1. Purpose and Scope
[DAO Name] is a community DAO operating within the Colabonate Protocol.
Purpose: [describe purpose]
Scope: [buy/sell | cooperation | both]
Domain: [geographic or topical restriction, or "open to all"]

## 2. Membership Criteria
Any Colabonate participant with Identity Level [2|3] may apply for membership.
Membership is approved by [governance vote | founding members | automatic].

## 3. Governance Rules
Voting model: [1P1V | token-weighted | reputation-weighted | hybrid]
Ordinary decisions: [X]% quorum, >[Y]% majority, [N] days
Codex amendments: [X]% quorum, 2/3 majority, 21 days

## 4. Dispute Resolution
This DAO provides dispute resolution up to Level [1|2|3].
Mediators are selected from: [DAO member pool | Colabonate Foundation pool]
Mediation fee: [X]% of disputed ticket value (in sats)

## 5. Sanctions
Sanctionable behaviors: [list]
Sanction levels: Warning → Restriction → Temporary ban → Permanent ban
Sanction proposals require: [X]% of active members or [role]
Appeal period: [N] days

## 6. Codex Amendments
Amendments require 2/3 majority vote with 21-day voting period.
Amendments take effect [N] days after vote completion.
Retroactive amendments affecting active tickets: not permitted.

## 7. Dissolution
The DAO is dissolved if: [conditions]
Upon dissolution: treasury funds are distributed [proportionally to members | to Foundation DAO | burned].
```

---

## DAO Registration Procedure

### Step 1: Founding Member Assembly

1. Minimum 5 founding members gather (in-person or via Nostr DM)
2. Each founding member must have Identity Level ≥ 2
3. Founding members collectively draft the Codex
4. Each founding member reviews and signs off on the Codex

### Step 2: Codex Validation

1. Founding members submit the Codex to the protocol's validation endpoint (or via Foundation DAO)
2. The protocol checks that all required sections are present
3. A Codex hash is generated: `SHA-256(codex-content)` — this hash becomes the permanent identifier for this Codex version

### Step 3: Stake Deposit

1. Each founding member stakes the required minimum COLA via a Kind 30025 event
2. The stake is locked for the minimum founding period (default: 90 days)
3. If the DAO is dissolved within 90 days, stakes are returned minus a protocol fee

### Step 4: DAO Registration Event

The lead founding member (or all jointly) publishes a Nostr event registering the DAO:

```json
{
  "kind": 30022,
  "pubkey": "<lead-founder-pubkey>",
  "tags": [
    ["d", "<dao-id>"],
    ["sub_type", "dao_creation"],
    ["dao_name", "<DAO Name>"],
    ["codex_hash", "<sha256-of-codex>"],
    ["codex_version", "1.0"],
    ["founding_members", "<pubkey1>,<pubkey2>,<pubkey3>,<pubkey4>,<pubkey5>"],
    ["member_count", "5"]
  ],
  "content": "<codex-full-text-or-nostr-reference>"
}
```

### Step 5: Confirmation

After the registration event is published:
- Protocol validates the DAO (checks all founding member requirements)
- Foundation DAO receives a notification (observation only — not approval)
- DAO ID is active and can be referenced in ticket `daoId` fields

The DAO does not require Foundation DAO approval. It is **permissionless** as long as all requirements are met.

---

## Interaction Binding to a DAO

Once a DAO is registered, participants can optionally bind their interactions to it:

### Binding a Ticket

When creating a ticket (Kind 30018), include:

```json
{
  "tags": [
    ["dao_id", "<dao-id>"],
    ["codex_hash", "<sha256-of-codex-version-at-binding>"],
    ["codex_version", "1.0"]
  ]
}
```

### What Binding Means

| Aspect | Effect |
|--------|--------|
| Dispute resolution | Disputes on this ticket use the DAO's dispute procedure instead of the base protocol defaults |
| Arbitration | Level 3 verdicts use DAO arbitrators (if DAO provides Level 3) |
| Codex version freeze | The Codex version at the time of binding is locked for this ticket; later Codex amendments do not affect active tickets |
| Legal implication | Parties implicitly agree to the DAO's Codex as a private arbitration framework |
| Protocol rules | Base Colabonate Protocol rules still apply; DAO Codex may add requirements but cannot remove protocol guarantees |

### Binding is Optional

The base Colabonate Protocol operates without any DAO binding. Binding is a user choice for those who want:
- Stronger community accountability
- Specialised dispute resolution for their domain
- Legal framework reference (private arbitration agreement)

See [legal-binding-layer.md](../core/legal-binding-layer.md) for the full specification.

---

## DAO Lifecycle

### Creation
- Minimum 5 founders with Level 2+
- Codex validated
- Registration event published

### Active
- Members can join per Codex membership criteria
- Transactions can be bound to DAO
- Governance votes can be run
- Treasury can be funded (optional Lightning multi-sig)

### Amendment
- Codex amendments follow the amendment procedure defined in the Codex
- A new Codex version creates a new `codex_hash` — all future bindings use the new hash
- Active tickets retain the old hash (Codex version frozen at binding)

### Dissolved
- Dissolution event published (Kind 30022 with `sub_type: dao_dissolution`)
- Existing bound tickets continue under the dissolved Codex until completion
- No new tickets can be bound to a dissolved DAO
- Treasury distribution per dissolution clause

---

## Foundation DAO Relationship

Community DAOs are **independent** of the Foundation DAO. The Foundation DAO:
- Sets the protocol defaults (minimum founder count, minimum COLA stake, etc.) via governance vote
- Does not approve or veto individual community DAOs
- Does not interfere with community DAO governance

A community DAO MAY reference the Foundation DAO Codex for dispute escalation (Level 3) if it does not have its own Level 3 arbitration infrastructure.

---

## References

- [docs/protocols/core/legal-binding-layer.md](../core/legal-binding-layer.md) — opt-in binding mechanism
- [docs/protocols/governance/dao-codex.md](./dao-codex.md) — Foundation DAO Codex
- [docs/protocols/governance/economic-protocol.md](./economic-protocol.md) — COLA staking requirements
- [docs/protocols/identity/identity-protocol.md](../identity/identity-protocol.md) — Level 2 requirement
- [docs/protocols/core/nostr-events.md](../core/nostr-events.md) — Kind 30022 (governance/DAO events)
- [ADR 010: Identity Level Model](../../decisions/010-identity-level-model.md)

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
