# 011 – COL-Points vs COLA Token — Two-Track Reputation and Governance

**Status:** accepted
**Date:** 2026-03-22
**Decider:** Deniz Yilmaz (Founder & Protocol Architect)

## Context

The Colabonate protocol needs mechanisms for two distinct purposes:

1. **Reputation** — A way to signal that a participant has a track record of reliable, high-quality protocol interactions. This should be non-transferable (cannot be bought) and accumulate through genuine participation.

2. **Governance** — A way for participants to express their preferences in protocol decisions. This should allow meaningful participation by community members who are active and invested in the protocol.

The question is whether these two purposes should be served by a single instrument or two separate instruments.

Several systems in the Web3 space attempt to use a single token for both reputation and governance. This approach has well-documented problems (governance captured by token holders with no protocol participation history; reputation that can be accumulated through purchases).

## Decision

We use **two separate instruments**:

**COL-Points** — Off-chain, non-transferable, accumulated through protocol participation. Used for reputation scoring and as one input to governance (reputation-weighted voting model). Cannot be purchased, transferred, or sold. Resets if the user changes their Nostr pubkey.

**COLA Token** — On-chain (RSK sidechain), transferable, issued through governance votes. Used for the token-weighted governance voting model. Can be staked for APY and traded. Provides a mechanism for early contributors and investors to participate in governance proportional to their economic stake.

**Why both are needed:**
- COL-Points alone would exclude early investors who haven't had time to build participation history but have economic alignment
- COLA alone would enable plutocratic governance — wealthy actors with no participation history could dominate votes
- Together, they enable the DAO to choose the appropriate voting model per decision type: 1P1V (identity-gated), token-weighted (COLA), or reputation-weighted (COL-Points)

**Key distinctions:**

| Property | COL-Points | COLA Token |
|----------|-----------|-----------|
| Type | Off-chain accumulation | On-chain token |
| Transferable | No | Yes |
| Purchasable | No | Yes (public sale) |
| Earned by | Protocol participation | Purchased / earned via distribution |
| Governance use | Reputation-weighted voting | Token-weighted voting |
| Currency function | No | No (payments remain in sats) |
| Phase introduced | Phase 3 | Phase 4 |

**Neither is a currency.** All actual payments on Colabonate remain in Bitcoin (sats). COLA is explicitly not a payment medium.

## Alternatives

### 1. Single Token for Both Reputation and Governance

Issue a single token that serves as both reputation score and governance weight. Non-transferable initially, but earnable through participation.

**Why rejected:** Non-transferable tokens cannot be used for initial community bootstrapping (cannot be distributed to early contributors/investors without transfer). If the token becomes transferable later, reputation can then be purchased by transferring to new accounts. The tension between "reputation should be non-transferable" and "governance tokens need liquidity for investors" is fundamental and cannot be resolved by a single instrument.

### 2. Purely Reputation-Based Governance (No COLA Token)

Use only COL-Points for governance. No transferable token.

**Why rejected:** Without a transferable token, there is no mechanism to:
- Compensate early contributors and investors economically
- Create liquidity for the protocol ecosystem
- Enable the "public sale" component needed for open-source funding
- Allow new participants who are economically aligned but new to the protocol to have any governance voice before building participation history

### 3. Purely Token-Based Governance (No COL-Points)

Use only COLA token for governance. No reputation system.

**Why rejected:** Pure token-weighted governance is plutocratic. Large token holders can dominate all decisions regardless of their actual protocol participation or expertise. This creates incentives to accumulate tokens for governance power rather than to genuinely participate in the protocol. It also fails for Sybil resistance — governance can be captured by large token purchases.

### 4. Three Instruments (Add a Separate Staking Token)

Add a third instrument specifically for staking yield, separate from both reputation and governance.

**Why rejected:** Three instruments create unnecessary complexity. COLA token already handles staking (5% APY for protocol participation) as a secondary utility alongside governance. Adding a third instrument would increase protocol complexity without meaningful benefit.

## Consequences

### Positive

- COL-Points cannot be gamed by purchasing token stake — reputation genuinely reflects participation history
- COLA token provides an economic mechanism for early contributors and investors without conflating it with reputation
- The DAO can apply different voting models to different decision types (most critical decisions can require 1P1V, routine decisions can use token-weighted)
- Clear semantic separation: COL-Points = "track record", COLA = "economic stake"
- Reputation-weighted voting model creates an incentive to participate actively in the protocol before governance matters

### Negative

- Two instruments create complexity for both implementers and users — users must understand the difference
- COLA token requires a separate RSK-based implementation (Phase 4 — not needed for Phase 1-3)
- COL-Points off-chain storage in Phase 1-3 creates a centralization risk (server controls the database). Nostr migration in Phase 4 resolves this.
- If COLA becomes highly liquid, governance could still be captured by a large purchaser — mitigated by quorum requirements and 1P1V override options but not fully eliminated

## Affected Documents

- `docs/protocols/governance/economic-protocol.md` — full COLA token specification
- `docs/protocols/core/reputation-protocol.md` — COL-Points definition and computation
- `docs/protocols/governance/dao-codex.md` — governance voting models
- `docs/protocols/core/nostr-events.md` — Kind 30024 (COL-Points events), Kind 30025 (COLA stake events)

## References

- [Vitalik Buterin: "Soulbound"](https://vitalik.eth.limo/general/2022/01/26/soulbound.html) — conceptual inspiration for non-transferable credentials (applied in Bitcoin-native form)
- [docs/protocols/governance/economic-protocol.md](../protocols/governance/economic-protocol.md)
- [docs/protocols/core/reputation-protocol.md](../protocols/core/reputation-protocol.md)

---

*Decision made by Deniz Yilmaz (Founder) | 2026-03-22*
