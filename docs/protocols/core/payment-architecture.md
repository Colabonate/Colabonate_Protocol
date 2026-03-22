# Payment Architecture – Multi-Layer Bitcoin Stack

**Version:** 1.0.0-draft
**Date:** 2026-03-22
**Status:** [IMPLEMENTED] Layer 1 + basic Lightning | [PHASE 2] Lightspark Grid | [PHASE 4] RSK Contract Layer

---

## Overview

Colabonate uses a **three-layer Bitcoin architecture**. Each layer serves a distinct purpose. The Unified Wallet abstracts this complexity from end users — they see one wallet, one balance, one payment experience.

```
┌────────────────────────────────────────────────────────────────┐
│                    Colabonate Unified Wallet                    │
│  (abstracts layer selection, path routing, currency conversion) │
└────────────────┬──────────────┬───────────────┬───────────────┘
                 │              │               │
     ┌───────────▼───┐  ┌───────▼──────┐  ┌───▼────────────────┐
     │  Bitcoin L1   │  │  Lightspark  │  │   RSK Sidechain    │
     │  Security     │  │  Grid        │  │   Contract Layer   │
     │  Anchor       │  │  Performance │  │   Governance/Logic │
     └───────────────┘  └──────────────┘  └────────────────────┘
```

---

## Layer 1: Bitcoin (Security Anchor)

**Status:** [IMPLEMENTED] — always available as base layer

### Purpose

Bitcoin Layer 1 is the **ultimate security anchor** for the Colabonate protocol. It is not used for everyday payments (too slow, too expensive for microtransactions) but serves as:

- Settlement finality for high-value transactions
- Anchor for RSK sidechain security (merge-mined with Bitcoin)
- Reserve currency layer — RBTC on RSK is backed 1:1 by BTC via two-way peg
- Fallback for dispute resolution verdicts that require on-chain finality

### When L1 is Used

| Use Case | Threshold | Notes |
|----------|-----------|-------|
| RSK two-way peg (BTC → RBTC) | Any amount | User moves BTC to RSK for contract interactions |
| High-security escrow finality | > threshold set by DAO | Protocol recommends L1 anchor for high-value tickets |
| Treasury settlements | Large DAO treasury movements | Multi-sig Lightning + L1 anchor |

### What L1 is NOT Used For

- Everyday marketplace payments → use Lightspark Grid
- Protocol escrow → use LNBits Hold Invoices (Lightning)
- Governance votes → use RSK/Nostr
- Microtransactions → use Lightspark Grid

---

## Layer 2 Performance: Lightspark Grid

**Status:** [PHASE 2]

### Purpose

Lightspark Grid is the **performance layer** for all everyday protocol interactions. It provides enterprise-grade Lightning Network infrastructure with automatic pathfinding and liquidity management — users do not need to manage channels or liquidity.

### Core Capabilities

| Feature | Description |
|---------|-------------|
| **Automatic pathfinding** | Lightspark handles routing; no manual channel management |
| **Liquidity management** | Automatic rebalancing; outbound capacity always available |
| **Microtransactions** | Sub-cent payments without on-chain fees |
| **Near-instant settlement** | Sub-second payment confirmation |
| **Spark Stablecoins** | Lightning-native price-stable transfers (see below) |
| **NIP-57 Zaps** | Nostr zap payments for tipping and micro-rewards |

### Managed vs. Self-Custody Model

The protocol supports two integration modes:

**Managed Custodial Model (standard users):**
- Colabonate manages Lightspark Grid integration on behalf of user
- User does not manage channels, liquidity, or node operations
- Suitable for most buyers, sellers, and cooperators
- Trade-off: lower sovereignty, higher convenience

**Self-Custody Model (expert users):**
- User connects their own Lightning node (LND or CLN)
- Colabonate interacts with user's node via standard BOLT11
- Full sovereignty over funds and routing
- Suitable for advanced users, merchants with high volume

**Protocol rule:** The payment protocol is identical for both models. The settlement events (Nostr Kind 30019) are identical regardless of custody model. Client implementations must support both.

### Spark Stablecoins

Spark Stablecoins are Lightning-native price-stable assets integrated via Lightspark Grid:

- Denominated in fiat currencies (USD, EUR) but transferred over Lightning
- **All Colabonate protocol fees remain denominated in sats** — Spark is an optional user-facing payment currency
- Stablecoin transfers produce the same Nostr events as sat transfers
- The `price` tag in Kind 30017 Offer events includes currency denomination:
  ```json
  ["price", "1000", "sats"]          // sat-denominated
  ["price", "10.00", "USD", "spark"] // Spark stablecoin (optional)
  ```
- Protocol-level escrow always uses sats as the escrow unit; stablecoin conversion happens at the wallet layer

### When Lightspark Grid is Used

| Use Case | Layer | Notes |
|----------|-------|-------|
| Marketplace payments (buy/sell) | Lightspark Grid | Default for all ticket payments |
| Escrow (Phase 1/2/3 invoices) | Lightspark Grid | LNBits Hold Invoices via Lightspark |
| COL-Points micro-rewards | Lightspark Grid | Small sat amounts |
| NIP-57 Zaps / tips | Lightspark Grid | Community engagement micro-payments |
| Cooperation milestone payments | Lightspark Grid | Per-milestone Lightning invoices |

---

## Layer 2 Contract: RSK (Rootstock)

**Status:** [PHASE 4]

### Purpose

RSK is the **contract layer** for complex logic that cannot be expressed with Lightning HTLCs alone. RSK is a Bitcoin sidechain — RBTC is 1:1 pegged to Bitcoin via two-way peg and the chain is secured by Bitcoin merge-mining.

**Important:** RSK is Bitcoin technology, not Ethereum. While RSK uses Solidity for smart contracts (EVM-compatible bytecode for tooling reuse), the economic security is provided by Bitcoin miners, not ETH. No ETH is involved.

### Three Defined Use Cases on RSK

#### 1. Governance Logic
- DAO voting smart contracts (COLA token staking, vote counting, quorum enforcement)
- Protocol upgrade execution (automated parameter changes after successful governance votes)
- Arbitrator election contracts

#### 2. Ticket System Logic (Complex Cases)
- Multi-party cooperation contracts (N-party payment splits)
- Conditional release contracts (oracle-based delivery verification)
- Royalty distribution contracts (Phase 5: automatic sat distribution to protocol authors)
- Return/refund automation (Phase 4+)

#### 3. Escrow Functions (Complex Cases)
- High-value escrow requiring on-chain finality guarantee
- Cross-DAO escrow (when two different community DAOs are parties)
- Codex-regulated financial instruments (Codex Forks — see below)

### Codex Forks

**Codex Forks** are a unique Colabonate primitive: Bitcoin-based economic sub-units governed by a Codex, implemented as RSK smart contracts.

#### What is a Codex Fork?

A Codex Fork is an economic unit that:
- Is created by a HID-verified user (Identity Level 2 minimum)
- Has predefined rules encoded in an RSK smart contract referencing a Codex
- Functions as a financial instrument for a specific purpose (e.g. project financing, community treasury, cooperative fund)
- Trades exclusively on the RSK contract layer to ensure governance compliance
- Cannot be used outside the DAO Codex that regulates it

#### Codex Fork vs. Community DAO

| | Codex Fork | Community DAO |
|--|-----------|---------------|
| Purpose | Economic sub-unit (instrument) | Governance community |
| Created by | Individual HID user | Min. 5 HID users |
| Governed by | Referenced Codex | Own Codex |
| Scope | Financial instrument | Full governance structure |
| Layer | RSK smart contract | Nostr + RSK |

#### Codex Fork Creation Protocol

```
1. Creator must have Identity Level ≥ 2
2. Creator references an existing valid Codex (own DAO or Foundation DAO)
3. Creator deploys RSK smart contract with:
   - codexHash (SHA-256 of governing Codex version)
   - daoId (which DAO governs this fork)
   - rulesHash (SHA-256 of the fork's specific rules)
   - participants (list of authorized pubkeys)
   - maturityConditions (release conditions)
4. Fork is registered as a Nostr Kind 30022 event (sub_type: codex_fork_created)
5. Fork can receive BTC via RSK peg-in
```

#### Example Use Cases

| Codex Fork Type | Purpose |
|----------------|---------|
| Project Financing Fork | Fund a specific project milestone; release funds on completion proof |
| Cooperative Fund Fork | Pool resources for a group purchase; distribute on vote |
| Escrow Arbitration Fork | Hold disputed funds until DAO verdict |
| Community Bounty Fork | Fund open work; release to first verified completer |

---

## Unified Wallet Protocol

The Unified Wallet is the user-facing abstraction of the three-layer stack. From a **protocol specification perspective**, the Unified Wallet must implement the following routing logic:

### Payment Path Selection

```
For any outgoing payment:

1. Determine payment type:
   ├── Simple marketplace payment (sats, < threshold)
   │     → Route via Lightspark Grid
   │
   ├── Stablecoin payment (Spark)
   │     → Route via Lightspark Grid (stablecoin rail)
   │
   ├── Escrow payment (Hold Invoice)
   │     → Route via Lightspark Grid (LNBits Hold Invoice)
   │
   ├── Governance / DAO interaction
   │     → Route via RSK (if RBTC available) or Nostr event only
   │
   └── Codex Fork interaction / high-value escrow
         → Route via RSK (requires peg-in if only BTC available)
```

### Asset Management

The Unified Wallet manages:

| Asset | Network | Use |
|-------|---------|-----|
| BTC | Bitcoin L1 | Reserve, RSK peg-in |
| Lightning sats | Lightspark Grid | Everyday payments, escrow |
| Spark Stablecoins | Lightspark Grid | Price-stable payments (optional) |
| RBTC | RSK | Governance, contract interactions |
| Codex Fork units | RSK | Regulated economic instruments |
| COLA Token | RSK | Governance staking |

### HID Linkage Requirement

Per Whitepaper Section 3.2, every transaction must be linked to the validated Human Identity (HID) of the sender:

| Phase | HID Requirement | Enforcement |
|-------|----------------|-------------|
| Phase 1–2 | Optional (LNURL-Auth pubkey sufficient) | No enforcement |
| Phase 3 | Required for governance votes and mediator actions | Protocol-level check |
| Phase 4+ | Required for high-value tickets above DAO-set threshold | Smart contract enforcement |
| RSK interactions | Always required (Identity Level 2 minimum) | Smart contract enforcement |

**Rationale:** HID linkage ensures that all ecosystem payments come from known, trusted participants with full traceability for DAO governance — without requiring central KYC.

---

## Architecture Decision Summary

| Decision | Choice | ADR |
|---------|--------|-----|
| Why three layers instead of one | Different trade-offs: L1 (security), Lightning (speed/cost), RSK (logic) | [ADR 012](../../decisions/012-payment-layer-architecture.md) |
| Why Lightspark Grid over raw LND | Enterprise reliability, automatic liquidity, managed model for users | [ADR 012](../../decisions/012-payment-layer-architecture.md) |
| Why RSK over other Bitcoin sidechains | Bitcoin merge-mining security, EVM tooling availability, 1:1 BTC peg | [ADR 010](../../decisions/010-lnbits-hold-invoice-escrow.md) |

---

## Phase Roadmap

| Phase | Payment Layer Features |
|-------|----------------------|
| Phase 1 (current) | Basic Lightning via LNBits, LNURL-Auth |
| Phase 2 | Lightspark Grid integration, Hold Invoice escrow, Spark Stablecoins |
| Phase 3 | HID-linked transactions, NIP-57 zaps |
| Phase 4 | RSK governance contracts, Codex Fork creation, high-value escrow |
| Phase 5 | Full Unified Wallet with automatic path selection, Codex Fork marketplace |

---

## Discovered Issues and Open Questions

> These items were identified during the synchronization of the Whitepaper v6 with the protocol specs and require clarification or future ADRs.

1. **Spark Stablecoins denomination in escrow:** The escrow protocol currently specifies sats only. If a buyer pays in Spark Stablecoins, the conversion to sats for Hold Invoice must happen at the Lightspark layer. The protocol must define the conversion point and who bears exchange risk. → [OPEN]

2. **NIP-57 Zap integration:** The Whitepaper references NIP-57 (Nostr zaps) in the tech stack (Appendix A). This is not currently in the Nostr events spec. Zaps for tipping and micro-rewards should be added to `nostr-events.md` as a supported event type. → [OPEN]

3. **Codex Fork — RSK contract standard:** The Codex Fork mechanism requires a standardized RSK smart contract interface. This needs a dedicated spec document in Phase 4. → [PLANNED: Phase 4]

4. **HID Linkage enforcement threshold:** The Whitepaper states every transaction requires HID linkage. The protocol currently has no enforcement threshold. The DAO should define the sat-value threshold above which HID Level 2 is required. → [OPEN — DAO governance decision]

---

## References

- [Whitepaper v6, Section 3.1–3.2](../../Colabonate_Konzepte_privat/Konzept/Whitepaper/Whitepaper-Colabonate-v6-Final-Public-Release.md) — authoritative source
- [docs/protocols/core/escrow-protocol.md](./escrow-protocol.md) — escrow mechanics
- [docs/protocols/identity/identity-protocol.md](../identity/identity-protocol.md) — HID levels
- [docs/protocols/governance/economic-protocol.md](../governance/economic-protocol.md) — COLA token
- [docs/protocols/governance/dao-creation-protocol.md](../governance/dao-creation-protocol.md) — DAO and Codex prerequisites
- [Lightspark](https://lightspark.com)
- [RSK / Rootstock](https://rootstock.io)
- [NIP-57: Lightning Zaps](https://github.com/nostr-protocol/nostr/blob/master/57.md)

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
