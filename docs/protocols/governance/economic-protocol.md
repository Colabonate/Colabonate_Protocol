# Economic Protocol – COLA Token and Fee Structure

**Version:** 1.0.0-draft
**Date:** 2026-03-22
**Status:** [PHASE 4]

---

## Protocol Position Statement

**Colabonate is Bitcoin-only. All actual payments occur in Bitcoin (satoshis).**

The COLA token is a **governance and utility instrument**, not a currency and not a payment medium. Receiving or making payments in Colabonate always means receiving or sending sats via Lightning Network. COLA cannot be used to pay for offers or services within the protocol.

This distinction is fundamental:
- **sats** — the medium of exchange for all trades, services, and fees
- **COL-Points** — non-transferable reputation accumulation (off-chain)
- **COLA token** — transferable governance token (on RSK sidechain)

See [ADR 012](../../decisions/012-col-points-vs-cola-token.md) for the rationale for this separation.

---

## COL-Points (Phase 3)

COL-Points are defined fully in [reputation-protocol.md](../core/reputation-protocol.md). Summary:

| Property | Value |
|----------|-------|
| Type | Off-chain, non-transferable |
| Denominated in | Points (no monetary value) |
| Earned by | Protocol participation |
| Governance use | Reputation-weighted voting |
| Purchasable | No |
| Phase | Phase 3 |

---

## COLA Token (Phase 4)

### Overview

| Property | Value |
|----------|-------|
| Symbol | COLA |
| Network | RSK (Rootstock) sidechain — Bitcoin-secured |
| Standard | RRC-20 (RSK's ERC-20 compatible standard) |
| Max supply | 100,000,000 COLA |
| Initial supply | 0 (minting requires governance vote) |
| Currency function | None — all payments remain in sats |

**Important:** RSK is a Bitcoin sidechain. RBTC (RSK's native token, used for gas) is 1:1 pegged to Bitcoin via two-way peg. Using RSK does not involve Ethereum or ETH.

### Token Distribution

**Source:** Whitepaper v6, Section 5.2 (authoritative)

| Allocation | Percentage | Amount | Lock / Vesting |
|-----------|-----------|--------|----------------|
| Community Rewards Pool | 40% | 40M COLA | Ecosystem incentives, grants, bounties — linear release |
| DAO Treasury | 25% | 25M COLA | DAO-controlled future development — governed release |
| Team & Advisors | 20% | 20M COLA | 4-year vesting, 1-year cliff |
| Early Backers | 10% | 10M COLA | Open Collective, GitHub Sponsors, founding partners |
| Liquidity & Market Making | 5% | 5M COLA | Immediate, for DEX/exchange liquidity |

**Note:** Community Rewards are distributed through protocol participation (not an airdrop). Users earn COLA by accumulating COL-Points above thresholds set by DAO governance vote. The 40% Community Rewards Pool is the largest single allocation — reflecting the protocol's community-first orientation.

> **Discrepancy resolved:** An earlier draft of this document used a different distribution (30/25/20/15/10). The Whitepaper v6 Section 5.2 is the authoritative source. This document now reflects the correct allocation.

### Issuance

- No COLA is minted without a governance vote
- Initial minting requires a Protocol Upgrade vote (25% quorum, 2/3 majority, 21-day voting period)
- Minting schedule is defined in the founding governance vote and can only be changed by a subsequent Protocol Upgrade vote

### Staking

| Property | Value |
|----------|-------|
| Annual yield | 5% APY (distributed quarterly in COLA) |
| Minimum stake | 1 COLA |
| Unstaking lockup | 30 days (configurable by DAO vote) |
| Staking purpose | Governance weight + yield |

Staking is published as a Kind 30025 Nostr event. See [nostr-events.md](../core/nostr-events.md).

### COLA in Governance

COLA enables the **token-weighted voting model** in the DAO:
- 1 staked COLA = 1 governance vote weight in token-weighted decisions
- Token-weighted voting is one of three models; the DAO chooses the model per proposal type
- COLA delegation (Liquid Democracy): token holders can delegate their vote weight to a delegate for 90 days, revocable anytime

See [dao-codex.md](./dao-codex.md) for voting model selection rules.

---

## Fee Structure

### Phase 1–3: No Platform Fee

During Phases 1–3, the Colabonate protocol charges **zero platform fees** on all transactions. This applies to:
- Buy/sell transactions (any amount)
- Cooperation milestone payments
- Any ticket type

Rationale: Community growth and adoption take precedence over revenue in early phases.

### Phase 4: Dispute Resolution Fees

When dispute resolution services are used, a fee is deducted from the escrow:

| Service | Fee | Who Receives |
|---------|-----|-------------|
| Level 2 Mediation | 1% of ticket value | Mediator (in sats) |
| Level 3 Arbitration | 2% of ticket value | Split: arbitrators + DAO community pool |
| No dispute | 0% | — |

Fees are deducted from escrow by the arbitrator verdict instruction event (Kind 30019 with `escrow_action` tag).

### Phase 5: Protocol Royalties

Community-published protocol workflows can carry royalty fees:

| Property | Detail |
|----------|--------|
| Fee type | Sat-denominated, per-use |
| Collection | Lightning Keysend to protocol author's pubkey |
| Amount | Set by protocol author (protocol may define max cap) |
| Distribution | 100% to protocol author (minus optional Foundation share if using Foundation infrastructure) |
| Governance | DAO can set maximum royalty rate cap |

This enables a **protocol marketplace** where community members create, publish, rate, and monetize workflow protocols. See [governance-roadmap.md](./governance-roadmap.md) Phase 5.

### Foundation Revenue Model

The Foundation's sustainability is funded by:

| Source | Amount | Notes |
|--------|--------|-------|
| Protocol Royalty (Foundation share) | 0–0.5% of licensed protocol royalties | Only applies when Foundation infrastructure is used |
| COLA token allocation | 25M COLA (vested) | Foundation uses for development funding |
| Grants | Variable | Spiral, OpenSats, Bitcoin grants |
| Donations | Variable | [Open Collective](https://opencollective.com/colabonate), [Patreon](https://www.patreon.com/c/Colabonate), Lightning donations |

The Foundation explicitly does NOT collect fees on base protocol usage (buy/sell/cooperation without dispute) in Phase 1–4.

---

## Economic Attack Resistance

| Attack | Mitigation |
|--------|-----------|
| Token-weighted governance capture (large buyer) | Quorum requirements (10–25% depending on proposal type); 1P1V override available for critical decisions; HID Level 3 required for 1P1V |
| Inflation via frequent minting votes | Protocol Upgrade vote required for any new minting (25% quorum, 2/3 majority) |
| Fee extraction without community benefit | Fee structure defined in protocol spec; any fee increase requires governance vote |
| Fake royalty traffic (self-referral) | Protocol marketplace reputation system tracks usage patterns; DAO can sanction abuse |
| Staking centralization | No maximum stake; DAO can vote to introduce limits |

---

## References

- [ADR 012: COL-Points vs COLA Token](../../decisions/012-col-points-vs-cola-token.md)
- [docs/protocols/core/reputation-protocol.md](../core/reputation-protocol.md) — COL-Points
- [docs/protocols/governance/dao-codex.md](./dao-codex.md) — Governance voting models
- [docs/protocols/core/nostr-events.md](../core/nostr-events.md) — Kind 30025 (stake events)
- [RSK / Rootstock](https://rootstock.io)

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
