# 012 – Payment Layer Architecture: L1 / Lightspark Grid / RSK

**Status:** accepted
**Date:** 2026-03-22
**Decider:** Deniz Yilmaz (Founder & Protocol Architect)
**Source:** Whitepaper v6, Section 3.1–3.2

## Context

Colabonate requires a payment infrastructure that simultaneously satisfies four competing requirements:

1. **Security** — Bitcoin's proven monetary security must be the foundation
2. **Performance** — Everyday marketplace payments must be near-instant and low-cost (suitable for microtransactions)
3. **Programmability** — Escrow, governance, royalty distribution, and Codex Forks require conditional logic beyond simple payment
4. **User simplicity** — Non-technical users must not need to manage Lightning channels, liquidity, or multi-layer wallets

No single Bitcoin technology satisfies all four requirements simultaneously. A three-layer architecture was designed.

**Additional drivers:**
- The Whitepaper v6 (Section 3.1) formally defines the three-layer stack as the protocol architecture
- Lightspark Grid provides enterprise-grade Lightning infrastructure with managed liquidity — critical for the managed custodial model
- Spark Stablecoins (Lightspark) enable price-stable payments without leaving the Bitcoin/Lightning ecosystem
- Codex Forks (Whitepaper 3.2) require RSK-level programmability and cannot be implemented on Lightning alone

## Decision

We use a **three-layer Bitcoin payment architecture**:

| Layer | Technology | Primary Use |
|-------|-----------|-------------|
| **Security Anchor** | Bitcoin Layer 1 | Reserve, RSK peg, high-value finality |
| **Performance Layer** | Lightspark Grid (Lightning) | Everyday payments, escrow, stablecoins, zaps |
| **Contract Layer** | RSK (Rootstock sidechain) | Governance, Codex Forks, complex escrow |

**Unified Wallet** abstracts all three layers from end users. Payment path selection is automatic based on transaction type and amount.

**HID Linkage:** Per Whitepaper Section 3.2, every transaction must be linked to the sender's validated HID. Enforcement is progressive:
- Phase 1–2: Optional (LNURL-Auth pubkey sufficient)
- Phase 3: Required for governance and mediator actions
- Phase 4+: Required for RSK interactions and high-value tickets (threshold defined by DAO)

**Codex Forks** are a new protocol primitive (Whitepaper 3.2): RSK-based economic sub-units created by HID-verified users, governed by a referenced Codex, trading exclusively on the contract layer.

## Alternatives

### 1. Lightning-only (no RSK)

Use only Lightning Network for all protocol functions, encoding governance logic as Lightning payment conventions.

**Why rejected:** Lightning HTLCs cannot express conditional releases beyond time-locks and hash-locks. Governance vote counting, multi-party payment splits for royalties, Codex Forks, and arbitration verdicts with partial fund releases require programmable smart contract logic. Lightning-only would either limit these features or require a centralized server to enforce them — defeating the decentralization goal.

### 2. Raw LND/CLN without Lightspark (self-managed Lightning)

Use standard Lightning node software without Lightspark's managed infrastructure.

**Why rejected:** Requiring users to manage their own Lightning nodes, channels, and liquidity creates an adoption barrier that conflicts with the user-simplicity requirement. The managed custodial model (Whitepaper 3.2) explicitly requires enterprise-grade Lightning infrastructure. Lightspark Grid provides automatic pathfinding and liquidity management that would take significant engineering effort to replicate. The self-custody option still exists for expert users — Lightspark does not prevent this.

### 3. RSK for everything (no Lightspark separation)

Use RSK for all payments including everyday marketplace transactions.

**Why rejected:** RSK has ~30-second block times and gas fees (RBTC). This makes it unsuitable for the sub-second, sub-cent payments needed for marketplace use. Lightning's off-chain nature provides orders-of-magnitude better performance for everyday payments. Mixing RSK into the critical payment path for all transactions would degrade performance and increase cost.

### 4. Two layers (L1 + RSK only, no Lightspark)

Skip Lightspark Grid and use Bitcoin L1 + RSK only.

**Why rejected:** Bitcoin L1 is unsuitable for everyday payments (confirmation time, fee). Without Lightning (via Lightspark Grid), Colabonate cannot offer the near-instant, low-fee payment experience needed for competitive marketplace positioning. Lightspark Grid specifically solves the liquidity management problem that has historically made Lightning adoption difficult for businesses.

### 5. Single non-Bitcoin chain (e.g. Ethereum + ERC-20)

Use Ethereum for smart contracts and payments.

**Why rejected:** This contradicts the Bitcoin-only principle (ADR 007). EVM chains have different security models, value systems, and communities. RSK achieves equivalent smart contract functionality while remaining anchored to Bitcoin's security through merge-mining and 1:1 BTC peg. No ETH or other altcoin is involved.

## Consequences

### Positive

- Three-layer separation provides the right tool for each payment category — no compromises
- Lightspark Grid enables the managed custodial model without sacrificing expert user self-custody
- Spark Stablecoins enable price-stable payments without leaving the Bitcoin ecosystem
- RSK enables Codex Forks — a new protocol primitive enabling economic governance not possible with Lightning alone
- Unified Wallet abstracts complexity; users interact with one wallet regardless of which layer processes their payment
- Bitcoin L1 as security anchor ensures the ultimate monetary foundation is always Bitcoin

### Negative

- Three-layer architecture increases implementation complexity
- Lightspark Grid is an external dependency — changes to Lightspark's API or business model could affect Phase 2 delivery
- RSK requires users to hold RBTC (via peg-in) for governance interactions — additional friction vs. Lightning-only
- Codex Forks (Phase 4) require RSK smart contract development and auditing — significant engineering investment
- Stablecoin integration (Spark) raises the open question of exchange rate risk management in escrow (documented as open issue in payment-architecture.md)

## Affected Documents

- `docs/protocols/core/payment-architecture.md` — full specification (new, created with this ADR)
- `docs/protocols/core/protocol-spec-v1.md` — technology stack table updated
- `docs/protocols/core/escrow-protocol.md` — RSK option clarified
- `docs/protocols/governance/economic-protocol.md` — RBTC and COLA token on RSK
- `docs/protocols/governance/dao-creation-protocol.md` — COLA stake via RSK

## References

- [Whitepaper v6 Section 3.1 — Multi-Layer Bitcoin Architecture](authoritative)
- [Whitepaper v6 Section 3.2 — Colabonate Payment System](authoritative)
- [Lightspark Grid](https://lightspark.com)
- [RSK / Rootstock](https://rootstock.io)
- [NIP-57: Lightning Zaps](https://github.com/nostr-protocol/nostr/blob/master/57.md)
- [ADR 010: LNBits Hold Invoice as Escrow](./010-lnbits-hold-invoice-escrow.md)
- [ADR 007: Bitcoin-only protocol](./007-protocol-documentation-bitcoin-only.md)

---

*Decision made by Deniz Yilmaz (Founder) | 2026-03-22*
