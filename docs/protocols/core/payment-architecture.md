# Payment Architecture – Bitcoin-Native, Non-Custodial

**Normativity:** Descriptive

**Version:** 2.0.0-draft
**Date:** 2026-08-05
**Status:** [IMPLEMENTED] Direct-Pay (Lightning Address / NWC / Cashu P2PK) | [BUILD→PILOT] ICP Escrow Canister | [OBSERVE] Lightspark Grid / Spark Stablecoins / RSK

> (PDC: see ADR-253) — Non-custodial strategy; the platform never holds funds.
> (PDC: see ADR-208/228/187/240) — NWC, Alby OAuth, Cashu (NIP-60/61), self-serve Lightning address.
> (PDC: see ADR-235) — Global fiat-currency display (display only; settlement always in sats).

---

## Overview

Colabonate's payment architecture is **Bitcoin-native and non-custodial**. The platform is a matchmaker and order-flow broker: it resolves how a buyer pays a seller, verifies payment proofs, and publishes immutable Nostr status events — **it never takes custody of funds**.

Two live paths serve every trade:

```
                       ┌─────────────────────────────────────────────┐
                       │            Colabonate (no custody)           │
                       │  order-flow broker · payment-proof verifier  │
                       └───────┬─────────────────────────┬────────────┘
                               │                         │
                  ┌────────────▼───────────┐   ┌──────────▼──────────────────┐
                  │   Path 1 — Direct-Pay  │   │  Path 2 — ICP Escrow Canister│
                  │  Lightning Address/NWC │   │   native Bitcoin (t-ECDSA)   │
                  │   or Cashu P2PK        │   │   on-chain escrow            │
                  │   (third-party mints)  │   │   (provable no human control)│
                  └────────────────────────┘   └──────────────────────────────┘
```

Settlement is always denominated and finalized in **sats**. Users may *display* amounts in a fiat currency of their choice (ADR-235), but conversion happens at the wallet layer; the protocol records sats.

---

## Path 1 — Direct-Pay (Launch, Non-Custodial)

**Status:** [IMPLEMENTED]

Listings without the escrow option pay **directly buyer → seller**. The seller's receiving configuration is resolved at ticket creation:

| Receiving method | Rail | Proof | Status |
|------------------|------|-------|--------|
| **Lightning Address** (LUD-16) | LNURL-pay to seller's `lnAddress` | Lightning preimage (`sha256(preimage)==paymentHash`) | Implemented (ADR-240) |
| **NWC** (NIP-47) | Server calls seller wallet `make_invoice` | Lightning preimage | Implemented (ADR-208) |
| **Cashu P2PK** | Buyer locks ecash token on seller pubkey | Seller self-report (no cryptographic equivalent) | Implemented (ADR-187) |

See [escrow-protocol.md](./escrow-protocol.md) §1 for the Direct-Pay flow and buyer-protection model.

### Mint Guardrail (Hard)

> Colabonate **never operates its own Cashu mint.** Only user-chosen third-party mints from a vetted default list are supported. Operating a mint would make Colabonate a custodian / e-money issuer — explicitly rejected (PDC: see ADR-253 / FU-39).

---

## Path 2 — ICP Escrow Canister (Native Bitcoin Escrow)

**Status:** [BUILD → PILOT]

Listings with the escrow option (above the minimum escrow size from the economics gate) route into an **ICP canister holding native Bitcoin** via threshold-ECDSA. The canister derives a per-trade P2WPKH address, detects funding itself (native UTXO API, no oracle), and signs payouts to buyer/seller only — a compile-time-verified invariant. **No human has control of the funds**, provably.

Full specification: [escrow-canister-protocol.md](./escrow-canister-protocol.md). Escrow mechanics and the `escrowProvider` discriminator: [escrow-protocol.md](./escrow-protocol.md) §2.

---

## Wallet Model

Colabonate connects to **the user's own wallets** rather than hosting a wallet. A user may have several connected wallets (one row per provider). Custody always stays with the user or the user-chosen third party.

### Wallet Provider Matrix

| Provider | Custody | Hold-Invoice (escrow) | Role | ADR |
|----------|---------|-----------------------|------|-----|
| **NWC** (NIP-47) | Non-custodial (user's node/wallet) | No (buyer-side dispatch; seller-side `make_invoice` for Direct-Pay) | Pay + receive | ADR-208 |
| **ALBY_OAUTH** | Provider custody (Alby) | No | Pay (+ receive via Lightning address) | ADR-228 |
| **CASHU** | Third-party mint (user-chosen) | No | Pay + receive (P2PK) | ADR-187 |
| **WEBLN** | Browser wallet | No | Pay | — |
| **LNBITS** | Provider/node | Legacy Hold-Invoice (flag, never public) | Dev/regtest only | ADR-193 |
| **LIGHTSPARK** | Provider | Planned (stub, post-V1) | — | (observe) |
| **MOCK / DEMO** | Simulated | No | Dev / Beta-mode only (hard-guarded off in production) | ADR-197 |

`WalletMode` distinguishes `CUSTODIAL` vs `NON_CUSTODIAL`; `WalletModeStrategy` (`HYBRID | CUSTODIAL_ONLY | NONCUSTODIAL_ONLY`) expresses a user's preference. NWC connection URIs are stored AES-256-GCM encrypted (server-side key); the server never derives keys from user credentials.

### Sovereign-Key Onboarding

Users without a Nostr key self-generate one in-browser during onboarding; the server never sees `nsec`. Lightning address is optional so wallet-less onboarding does not dead-end (PDC: see ADR-225).

---

## Global Currency Display

Users may choose a `preferredFiatCurrency` (ADR-235). Amounts are *displayed* in that currency but **settled in sats**; the protocol records sats everywhere. Conversion is a client concern.

---

## Observe-Tracks (parked, re-validated half-yearly via FU-253-F)

The following rails were part of earlier drafts of this specification. They are **not implemented** and are demoted to observe-tracks: they may be reactivated only if a measured need appears. They are documented here so implementers understand what the protocol deliberately does **not** currently rely on.

### Lightspark Grid (Performance Layer — observe)

Enterprise Lightning infrastructure with automatic pathfinding/liquidity and **Spark Stablecoins** (Lightning-native USD/EUR). Earlier drafts positioned this as the default performance layer. Status: **stub, post-V1**. Spark Stablecoins are **not implemented**. Reactivation trigger: production-grade non-custodial Lightning liquidity needs that Direct-Pay cannot meet.

### RSK / Rootstock (Contract Layer — observe)

Bitcoin sidechain (RBTC 1:1 peg, merge-mined) intended for governance logic, complex escrow, Codex Forks, and the COLA token (RRC-20). Status: **not implemented.** The non-custodial escrow need is met by the ICP canister (Path 2) instead. RSK remains the candidate home for COLA / governance contracts should governance move on-chain (Phase 4+) — see [economic-protocol.md](../governance/economic-protocol.md).

> **Important:** RSK, if ever adopted, is Bitcoin technology, not Ethereum. RBTC is 1:1 pegged to BTC; no ETH is involved. EVM terminology remains prohibited in normative specs (see GLOSSARY).

### NIP-57 Lightning Zaps (observe)

Standard Nostr zaps (Kinds 9734/9735) for tipping/micro-rewards. Referenced for future COL-Points incentives. Not wired into the launch rails. See [nostr-events.md](./nostr-events.md) for the event schema.

---

## Asset Summary

| Asset | Network | Role | Status |
|-------|---------|------|--------|
| BTC (sats) | Bitcoin L1 / Lightning / Cashu | All settlement | Implemented |
| Cashu ecash | Third-party mints (NIP-60/61) | Non-custodial Direct-Pay rail | Implemented |
| COL-Points | Off-chain (Nostr Kind 30024) | Reputation | Implemented (ledger) |
| COLA Token | RSK (RRC-20) — planned | Governance only (not payment) | Phase 4 / observe |
| Spark Stablecoins | Lightspark Grid | Display/stable payments | Observe |
| RBTC | RSK | Gas / contract interactions | Observe |

**Rule:** COLA is never a payment medium. All payments occur in sats.

---

## HID Linkage

Per the Whitepaper, transactions should be linked to a validated Human Identity. Current enforcement (PDC: see identity-protocol.md): optional in early phases; the exact sat-value threshold above which HID Level 2 becomes required is an **open DAO decision** (FU / open question). Path 2 (RSK-style contract enforcement) does not apply to the ICP canister, which authorizes by Nostr signature, not HID level.

---

## Open Questions

1. **Path 2 minimum escrow size** — determined by the economics gate (on-chain fee overhead measurement, FU-253-H); working hypothesis ~100–250k sats.
2. **Path 2 security gate** — external review/audit of canister t-ECDSA key handling, UTXO edge cases, reorg depth (FU-253-H).
3. **Path 2 controller governance** — blackhole vs DAO-multisig controller decision (FU-253-H).
4. **HID linkage enforcement threshold** — DAO governance decision (carried over from earlier drafts).
5. **RSK adoption for COLA/governance** — whether governance contracts land on RSK (Phase 4+) or an alternative.

---

## References

- [ADR-253](https://github.com/Colabonate) — Non-custodial strategy (Master)
- [escrow-protocol.md](./escrow-protocol.md) — payment & escrow paths
- [escrow-canister-protocol.md](./escrow-canister-protocol.md) — Path 2 spec
- [ticket-system.md](./ticket-system.md) — `escrowProvider`, `directPayRail`
- [governance/economic-protocol.md](../governance/economic-protocol.md) — COLA, fees
- [identity/identity-protocol.md](../identity/identity-protocol.md) — HID levels
- [NIP-47 (NWC)](https://github.com/nostr-protocol/nips/blob/master/47.md) · [NIP-57 (Zaps)](https://github.com/nostr-protocol/nips/blob/master/57.md) · [NIP-60/61 (Cashu)](https://github.com/nostr-protocol/nips)

---

*Part of the Colabonate Protocol Specification v0.2.0-draft | [docs/protocols/](../README.md)*
