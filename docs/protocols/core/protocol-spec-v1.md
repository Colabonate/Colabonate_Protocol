# Protocol Spec v1 – Colabonate Technical Specification

**Normativity:** Descriptive

**Version:** 1.1.0-draft
**Date:** 2026-08-05
**Status:** [IMPLEMENTED] Authentication + Marketplace + Direct-Pay | [BUILD→PILOT] ICP Escrow Canister | [OBSERVE] Lightspark / Spark / RSK

> (PDC: see ADR-253) — Non-custodial payment/escrow. The earlier "Lightspark Grid / LNBits Hold-Invoice / RSK" stack is superseded by Direct-Pay + ICP Canister.

---

## Document Map

This document is the technical hub of the Colabonate Protocol. For full specifications, follow these links:

| Topic | Document |
|-------|----------|
| Payment rails (Direct-Pay / Cashu / NWC / ICP) | [payment-architecture.md](./payment-architecture.md) |
| All Nostr event schemas (30017–30029, 30402–30423) | [nostr-events.md](./nostr-events.md) |
| Non-custodial payment & escrow paths | [escrow-protocol.md](./escrow-protocol.md) |
| ICP native-Bitcoin escrow canister (Path 2) | [escrow-canister-protocol.md](./escrow-canister-protocol.md) |
| Identity system (Level 0–3) | [identity/identity-protocol.md](../identity/identity-protocol.md) |
| Reputation and reviews | [reputation-protocol.md](./reputation-protocol.md) |
| Ticket types and state machine | [ticket-system.md](./ticket-system.md) |
| Legal binding layer | [legal-binding-layer.md](./legal-binding-layer.md) |
| DAO governance | [governance/dao-codex.md](../governance/dao-codex.md) |
| COLA Token and fees | [governance/economic-protocol.md](../governance/economic-protocol.md) |

---

## Protocol Layers

| Layer | Technology | Specification | Status |
|-------|-----------|--------------|--------|
| **Identity** | LNURL-Auth (LUD-04) | [identity-protocol.md](../identity/identity-protocol.md) | IMPLEMENTED |
| **Identity Verification** | Humanode Biomapper (biometric ZK) | [identity-protocol.md](../identity/identity-protocol.md) | PHASE 3 |
| **Human Identity** | HID (4-level, non-transferable) | [identity-protocol.md](../identity/identity-protocol.md) | PHASE 3 |
| **Transport** | Nostr Protocol (NIP-01) | [nostr-events.md](./nostr-events.md) | IMPLEMENTED |
| **Payments — Direct-Pay** | Lightning Address / NWC / Cashu P2PK (non-custodial) | [payment-architecture.md](./payment-architecture.md) | IMPLEMENTED |
| **Escrow — Path 2** | ICP Canister (native Bitcoin, t-ECDSA) | [escrow-canister-protocol.md](./escrow-canister-protocol.md) | BUILD→PILOT |
| **Reputation** | COL-Points (off-chain) + Nostr Reviews | [reputation-protocol.md](./reputation-protocol.md) | PHASE 3 |
| **Governance Token** | COLA Token (RSK/RRC-20) | [governance/economic-protocol.md](../governance/economic-protocol.md) | PHASE 4 |
| ~~Payments — Lightspark~~ | Lightspark Grid / Spark Stablecoins | [payment-architecture.md](./payment-architecture.md) | OBSERVE |
| ~~Smart Contracts — RSK~~ | RSK sidechain (Solidity) / Codex Forks | [payment-architecture.md](./payment-architecture.md) | OBSERVE |
| ~~Escrow — Hold-Invoice~~ | LNBits Hold Invoice | [escrow-protocol.md](./escrow-protocol.md) | LEGACY (flag, never public) |
| ~~Zaps / Tips~~ | NIP-57 Lightning Zaps | [nostr-events.md](./nostr-events.md) | OBSERVE |

---

## Architecture Diagram (Phase 2–4)

```
┌─────────────────────────────────────────────────────────────────┐
│                       Colabonate Protocol                        │
├─────────────────────────────────────────────────────────────────┤
│  Identity Layer                                                  │
│  ├── Level 0: LNURL-Auth (Lightning pubkey)       [IMPLEMENTED] │
│  ├── Level 1: Nostr Profile (NIP-01, NIP-05)      [PHASE 2]    │
│  ├── Level 2: Peer Verification (Proximity Proof) [PHASE 3]    │
│  └── Level 3: HID + Humanode Biomapper (ZK)       [PHASE 3]    │
├─────────────────────────────────────────────────────────────────┤
│  Protocol Layer                                                  │
│  ├── Nostr (transport, events 30017–30029, 30402–30423) [IMPL] │
│  ├── Ticket System (8 types)                      [PHASE 1-5]  │
│  ├── COL-Points (reputation, off-chain)           [PHASE 3]    │
│  └── COLA Token (governance, RSK)                 [PHASE 4]    │
├─────────────────────────────────────────────────────────────────┤
│  Payment Layer (Bitcoin-native, non-custodial)                   │
│  ├── Direct-Pay: Lightning Address / NWC / Cashu  [IMPLEMENTED] │
│  ├── ICP Escrow Canister (native BTC, t-ECDSA)    [BUILD/PILOT] │
│  ├── Bitcoin L1 (security anchor / canister base) [ALWAYS]      │
│  ├── Lightspark Grid / Spark Stablecoins          [OBSERVE]     │
│  └── RSK (governance + COLA, if adopted)          [OBSERVE]     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Authentication [IMPLEMENTED]

Colabonate uses **LNURL-Auth** (LUD-04) for passwordless identity:

```
1. Client requests k1 challenge:  GET /api/auth/lnurl
   ← { lnurl: "lnurl1...", k1: "abc123..." }

2. User opens LNURL in Lightning wallet
   Wallet signs k1 with secp256k1 (deterministic per domain, same key always)

3. Wallet calls callback:  GET /api/auth/lnurl/callback?k1=...&sig=...&key=...
   ← Server verifies Schnorr signature

4. Client polls status:  GET /api/auth/lnurl/status/:k1
   ← { status: "OK", pubkey: "02a1b2c3..." }
```

**No password. No email. Just a Lightning wallet.**

The resulting `pubkey` is the user's permanent pseudonymous identifier across all Colabonate interactions.

---

## Marketplace [IMPLEMENTED]

Offers are published as Nostr Kind 30017 events.

**Nostr Event format** (Kind 30017 — see [nostr-events.md](./nostr-events.md) for full schema):

```json
{
  "kind": 30017,
  "pubkey": "<seller-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<offer-id>"],
    ["title", "<title>"],
    ["price", "<sats>", "sats"],
    ["category", "<category>"]
  ],
  "content": "<description>",
  "sig": "<schnorr-signature>"
}
```

Optional for Phase 2+ offers with Spark Stablecoin pricing:
```json
["price", "10.00", "USD", "spark"]
```

---

## Payment Flow

Colabonate is **non-custodial**: the platform never holds funds. Two paths, chosen per listing (PDC: see ADR-253):

### Path 1 — Direct-Pay (Implemented, default)

```
Buyer creates ticket against an offer with useEscrow = false
  → Server resolves seller receiving method (Lightning Address → NWC → Cashu P2PK)
  → Buyer pays seller directly (no platform custody)
  → Payment proof verified (Lightning preimage OR seller self-report for Cashu)
  → Status: PAID → IN_PROGRESS → COMPLETED
```

### Path 2 — ICP Escrow Canister (Build → Pilot)

```
Buyer creates ticket against an escrow-enabled offer (useEscrow = true, escrowProvider = ICP)
  → Canister derives a per-trade P2WPKH address (t-ECDSA)
  → Buyer funds it on-chain → canister verifies UTXO (no oracle) → Funded
  → Milestone releases (e.g. 25/50/25) buyer-authorized → Released
  → OR Dispute → DAO verdict (signed) → split payout
  → OR Timeout → permissionless refund to buyer
```

Full escrow specification: [escrow-protocol.md](./escrow-protocol.md) · [escrow-canister-protocol.md](./escrow-canister-protocol.md)
Full payment rails: [payment-architecture.md](./payment-architecture.md)

---

## Nostr Event Kinds

Complete schemas in [nostr-events.md](./nostr-events.md). Summary:

| Kind | Usage | Status |
|------|-------|--------|
| 30017 | Offer (marketplace listing) | IMPLEMENTED |
| 30018 | Ticket created (legacy, dual-published) | IMPLEMENTED (as 30407) |
| 30019 | Ticket status update (legacy, dual-published) | IMPLEMENTED (as 30408) |
| 30020 | Dispute opened | IMPLEMENTED (M0 bootstrap) |
| 30021 | Verification credential (Soulbound) | IMPLEMENTED (arbitrator) / PHASE 3 (identity) |
| 30022 | Governance vote / DAO event | IMPLEMENTED (ADR-128) |
| 30023 | HID attestation (Humanode) | PHASE 3 |
| 30024 | COL-Points / reputation review | IMPLEMENTED |
| 30025 | COLA Token stake event | PHASE 4 |
| 30026 | Proximity Proof (peer verification) | PHASE 3 |
| 30027 | Company Profile | IMPLEMENTED |
| 30414 | Cooperation Proposal (NIP-C) | IMPLEMENTED |
| 30415 | Milestone Event (NIP-C) | IMPLEMENTED |
| 30420 | DAO Proposal | IMPLEMENTED (ADR-128) |
| 30421 | DAO Vote | IMPLEMENTED (ADR-128) |
| 30422 | DAO Membership / Registration | IMPLEMENTED (ADR-128) |
| 30423 | DAO Proposal Comment | IMPLEMENTED |

NIP-57 (Lightning Zaps) is an external NIP, currently on the **observe-track** (not wired into launch rails).

---

## Security Boundaries

| Rule | Detail |
|------|--------|
| Private keys never leave the client | LNURL-Auth transmits only the Schnorr signature; sovereign-key onboarding generates keys client-side (ADR-225) |
| No Bitcoin custody | Platform holds no BTC; Path 1 pays seller directly, Path 2 canister has provably no human control (ADR-253) |
| No own Cashu mint | Only third-party mints from a vetted list; operating a mint = custody (ADR-253/FU-39) |
| No raw biometrics | Humanode biotoken is on-device hash; no raw data transmitted |
| Pubkey = identifier, not secret | May be public; used as permanent pseudonymous ID |
| Escrow freeze on dispute | Path 2: canister blocks release/refund until a DAO verdict; Direct-Pay disputes affect reputation only |
| Immutable audit trail | All state changes published as Nostr events; permanent and verifiable |

---



## Protocol Versioning

The protocol version is declared in Nostr Kind 0 profile metadata:

```json
{
  "colabonate_protocol_version": "1.0.0-draft"
}
```

Breaking changes (Nostr schema changes) → major version bump. Full versioning policy (semver rules, deprecation windows, CHANGELOG location) is specified inline in this document (see the `Versioning` section above) and tracked in [CHANGELOG.md](../CHANGELOG.md).

---

## Known Gaps (Backlog)

| Gap | Impact | Resolution |
|-----|--------|-----------|
| No push notifications | Seller not notified of new tickets | Planned |
| Path 2 not yet live | Escrow canister built/pilot-gated, not Mainnet | FU-253-H gates (economics/security/governance/pilot) |
| No HID enforcement | All transactions anonymous | Phase 3 (opt-in), Phase 4 (enforced) |


---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
