# DAO Codex – Colabonate Foundation

**Version:** 1.0.0-draft
**Date:** 2026-03-22
**Status:** [PHASE 4] — Implementation begins Phase 4+
**Supersedes:** Conceptual draft (pre-2026-03-22)

---

## Preamble

The DAO Codex defines the constitutional rules of the Colabonate community governance system. It is **not a legal document** and is not directly enforceable in state courts. It applies within the technical system and is enforced through Nostr Events and Lightning Escrow.

Tickets with an optional DAO binding (`daoId` + `codexHash` fields) are subject to this Codex's dispute resolution and sanction procedures. See [legal-binding-layer.md](../core/legal-binding-layer.md) for the opt-in binding mechanism.

**Organizational Context:**

| Entity | Role |
|--------|------|
| **Colabonate Foundation** (Stiftung) | Protocol development, IP/trademark holder, protocol spec steward |
| **Colabonate DAO** | Community governance via this Codex, token-weighted and reputation-weighted voting |
| **Colabonate GmbH** | Commercial services entity, premium features, governed by separate terms |

This Codex governs the DAO layer only. The Foundation operates independently with its own board. The GmbH is governed by its own corporate structure. See [economic-protocol.md](./economic-protocol.md) for the DFINITY-style (Foundation → DAO → GmbH) governance rationale.

---

## Chapter I – Governance Protocols

### I.1 Voting Rights

Three voting models are available. The DAO selects the applicable model per proposal type (see I.2).

| Model | Requirement | Voting Power | Phase |
|-------|-------------|--------------|-------|
| **1P1V (One Person One Vote)** | Humanode Biomapper verification — Identity Level 3 | 1 vote per verified human | Phase 4+ |
| **Token-Weighted** | COLA Token staked via Kind 30025 event | 1 governance unit per 1 staked COLA | Phase 4+ |
| **Reputation-Weighted** | COL-Points (off-chain) | Score-based multiplier (DAO sets formula) | Phase 5+ |

**Humanode Integration:**
- Humanode Biomapper provides Sybil resistance through biometric ZK verification
- Each verified human receives exactly one HID (Human Identity Level 3)
- HID is non-transferable — bound to a biometric signature on-device
- Prevents: multiple accounts, bot networks, sock puppets
- Verification procedure: see [identity-protocol.md](../identity/identity-protocol.md#identity-level-3)

**COLA Token (Phase 4):**
- RRC-20 governance token on RSK (Bitcoin sidechain — not Ethereum)
- Staking enables token-weighted governance participation and 5% APY yield
- Full specification: [economic-protocol.md](./economic-protocol.md)
- Stake event schema: Nostr Kind 30025 — see [nostr-events.md](../core/nostr-events.md#kind-30025)

### I.2 Proposal Types and Voting Model Selection

| Proposal Type | Voting Model | Quorum | Majority | Duration |
|--------------|-------------|--------|---------|----------|
| Normal decision | Token-weighted or Reputation | 10% | >50% | 7 days |
| Codex amendment | 1P1V (if available) or Token-weighted | 20% | 2/3 | 21 days |
| Treasury spending | Token-weighted | 15% | >50% | 14 days |
| Sanction / ban | 1P1V required | 10% | 2/3 | 7 days |
| Protocol upgrade | Token-weighted + 1P1V confirmation | 25% | 2/3 | 21 days |
| New DAO creation (Foundation recognition) | Token-weighted | 10% | >50% | 7 days |

Quorum is measured as a percentage of active pubkeys (at least 1 COMPLETED ticket in the last 90 days).

### I.3 Voting Procedure

```
1. Proposer publishes a Governance Proposal (Nostr Kind 30022, sub_type: proposal)
   → Title, description, options, deadline, voting model

2. 14-day public comment period before voting opens (Codex amendments only)

3. Voting period opens — participants publish Kind 30022 vote events

4. Quorum and majority checked at deadline by protocol tallying logic

5. Result published as a final Kind 30022 event (immutable, signed by DAO operator pubkey)
```

Vote event schema: see [nostr-events.md](../core/nostr-events.md#kind-30022)

### I.4 Delegation (Liquid Democracy) [PHASE 4+]

- Voting rights can be delegated to another pubkey
- Delegation is revocable at any time
- Valid for 90 days, then automatically expired
- Delegation is published as a public Nostr Event (auditable)
- COLA delegation: delegatee casts vote with combined stake weight
- 1P1V delegation: not permitted — each HID-verified human must vote directly

### I.5 Amendment Procedure (Codex Changes)

- Requirement: 2/3 majority + 21-day voting period (Codex amendment type)
- Proposal must be published 14 days before voting opens
- Change history is traceable through immutable Nostr events
- The `codexHash` field in all active tickets references the Codex version at ticket creation — amendments do not retroactively affect in-flight tickets

---

## Chapter II – Legal and Security Protocols

### II.1 Dispute Resolution

**3-level dispute resolution system:**

| Level | Duration | Mechanism | Phase |
|-------|----------|-----------|-------|
| Level 1 | 7 days | Self-resolution (cooling-off, direct negotiation) | Phase 1+ |
| Level 2 | 14 days | Community Mediator (reputation-selected) | Phase 3+ |
| Level 3 | 24–48h | DAO Court (arbitrator panel, binding verdict) | Phase 4+ |

Full specification: [dispute-protocol.md](../workflows/dispute-protocol.md)

Escrow impact: DAO Court verdicts are published as Kind 30019 events with `escrow_action` tag, triggering Lightning escrow settlement. See [escrow-protocol.md](../core/escrow-protocol.md#dispute-to-escrow-verdict-mapping).

### II.2 Sanctions

| Level | Action | Trigger |
|-------|--------|---------|
| 1 | Warning (public Nostr Event) | First violation, minor breach |
| 2 | Temporary restriction (30 days — no ticket creation) | Repeat or serious violation |
| 3 | Permanent ban (requires DAO Governance Vote — Sanction type) | Fraud, abuse, serious harm |

> **Technical note:** Since identity = Lightning wallet pubkey, a banned user could create a new wallet. Sanctions are therefore effective primarily through reputation loss (COL-Points forfeiture, public Nostr sanction event visible to all counterparties) and combined with HID Level 3 enforcement requirements for high-value transactions.

### II.3 Security Principles

- Private keys never leave the client device
- No custody — Colabonate holds no Bitcoin at any layer
- All governance decisions are Nostr-signed and immutable on relay
- Open source — protocol spec is CC BY 4.0; reference implementation is open source
- Payment layer security: [escrow-protocol.md](../core/escrow-protocol.md)
- Identity layer security: [identity-protocol.md](../identity/identity-protocol.md)

---

## Chapter III – Economic and Financial Protocols

### III.1 Currency and Payment

- **Primary currency:** Bitcoin, denominated in satoshis (sats)
- All marketplace payments, escrow, mediator fees, and arbitrator fees are in sats via Lightning Network
- The COLA Token is a **governance and utility token** — it is not a payment currency
- No altcoin integration; RSK (for COLA) is a Bitcoin sidechain with 1:1 BTC peg (RBTC)

Full economic specification: [economic-protocol.md](./economic-protocol.md)

### III.2 COLA Token Summary

| Property | Value |
|----------|-------|
| Symbol | COLA |
| Network | RSK (Bitcoin sidechain) |
| Max supply | 100,000,000 COLA |
| Use | Governance voting weight, staking yield |
| Payment use | Not permitted — all payments in sats |

Token distribution, issuance, and staking rules: [economic-protocol.md](./economic-protocol.md#cola-token-phase-4)

### III.3 Fees

| Fee | Amount | Recipient | Phase |
|-----|--------|----------|-------|
| Platform fee | 0% | — | Phase 1–3 |
| Mediator fee | 1% of escrow | Mediator (in sats) | Phase 4+ |
| Arbitrator fee | 2% of escrow | Arbitrators + DAO pool (split) | Phase 4+ |
| Protocol royalty | Variable (sat-denominated) | Protocol author | Phase 5+ |

### III.4 Community Treasury

- Optional multi-sig Lightning treasury for each Community DAO
- Funding: voluntary contributions, protocol royalties (Phase 5), DAO grants
- Spending: requires Governance Vote (Treasury type)
- Foundation DAO Treasury: 25M COLA allocation (vested) — see [economic-protocol.md](./economic-protocol.md)

---

## Chapter IV – Social Protocols

### IV.1 Identity

- Pseudonymous identity allowed (pubkey = identity, no real name required at Level 0–1)
- Identity levels 0–3 unlock progressive features — no KYC required at any level
- Level 3 (Humanode) is required for 1P1V voting and high-value DAO interactions
- Full identity specification: [identity-protocol.md](../identity/identity-protocol.md)

### IV.2 Reputation

- Built through: completed tickets (COMPLETED status), mutual reviews (Kind 30024)
- Non-transferable: Soulbound via signed Nostr Events bound to pubkey
- Publicly visible: reputation score is computable from public Nostr events
- Can be lost: losing a dispute, sanctions, abandoning tickets
- Full reputation specification: [reputation-protocol.md](../core/reputation-protocol.md)

### IV.3 Inclusion

- No geographic restriction
- A Lightning-compatible wallet is the only technical prerequisite
- No KYC at base level (Level 0 = LNURL-Auth pubkey only)
- Platform UI will be multilingual [PHASE 3+]

### IV.4 User DAOs (Community DAOs)

Community members can create their own DAOs governed by a custom Codex:
- Based on the Foundation Codex as a template (this document)
- Independent governance from the Foundation DAO
- Minimum requirements: 5 founding members, HID Level 2, COLA stake
- Creation procedure: [dao-creation-protocol.md](./dao-creation-protocol.md)
- Interaction with tickets: [legal-binding-layer.md](../core/legal-binding-layer.md)

### IV.5 Protocol Marketplace [PHASE 5]

Community members can publish their own workflow protocols:
- Rated by the community (efficiency, quality, user satisfaction)
- Successful protocols become standard templates in the Protocol Registry
- Protocol authors can attach Royalty Tickets (sat-denominated, per-use fee)
- Open-source and premium models both supported
- Registry specification: [governance-roadmap.md](./governance-roadmap.md#phase-5)

---

## References

- [docs/protocols/core/legal-binding-layer.md](../core/legal-binding-layer.md) — Opt-in DAO binding for tickets
- [docs/protocols/core/escrow-protocol.md](../core/escrow-protocol.md) — Lightning escrow mechanics
- [docs/protocols/core/reputation-protocol.md](../core/reputation-protocol.md) — COL-Points and reviews
- [docs/protocols/identity/identity-protocol.md](../identity/identity-protocol.md) — 4-level identity system
- [docs/protocols/governance/economic-protocol.md](./economic-protocol.md) — COLA Token economics
- [docs/protocols/governance/dao-creation-protocol.md](./dao-creation-protocol.md) — Community DAO creation
- [docs/protocols/governance/governance-roadmap.md](./governance-roadmap.md) — Phase-by-phase roadmap
- [docs/protocols/workflows/dispute-protocol.md](../workflows/dispute-protocol.md) — Dispute resolution
- [docs/protocols/core/nostr-events.md](../core/nostr-events.md) — Event schemas (Kind 30022, 30025)
- [ADR 011: COL-Points vs COLA Token](../../decisions/011-col-points-vs-cola-token.md)

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
