# Protocol Spec v1 – Colabonate Technical Specification

**Normativity:** Descriptive

**Version:** 1.0.0-draft
**Date:** 2026-03-22
**Status:** [IMPLEMENTED] Authentication + Marketplace | [PHASE 2] Escrow + Lightspark | [PHASE 4] RSK + Dispute

---

## Document Map

This document is the technical hub of the Colabonate Protocol. For full specifications, follow these links:

| Topic | Document |
|-------|----------|
| Payment layer (L1 / Lightspark / RSK) | [payment-architecture.md](./payment-architecture.md) |
| All Nostr event schemas (30017–30026) | [nostr-events.md](./nostr-events.md) |
| Lightning escrow mechanics | [escrow-protocol.md](./escrow-protocol.md) |
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
| **Payments (everyday)** | Lightspark Grid (Lightning) | [payment-architecture.md](./payment-architecture.md) | PHASE 2 |
| **Payments (base)** | Bitcoin Lightning Network (LNBits) | [escrow-protocol.md](./escrow-protocol.md) | IMPLEMENTED |
| **Payments (security anchor)** | Bitcoin Layer 1 | [payment-architecture.md](./payment-architecture.md) | IMPLEMENTED |
| **Stablecoins** | Spark Stablecoins (Lightspark) | [payment-architecture.md](./payment-architecture.md) | PHASE 2 |
| **Smart Contracts** | RSK (Bitcoin sidechain, Solidity) | [payment-architecture.md](./payment-architecture.md) | PHASE 4 |
| **Escrow** | LNBits Hold Invoice | [escrow-protocol.md](./escrow-protocol.md) | PHASE 2 |
| **Reputation** | COL-Points (off-chain) + Nostr Reviews | [reputation-protocol.md](./reputation-protocol.md) | PHASE 3 |
| **Governance Token** | COLA Token (RSK/RRC-20) | [governance/economic-protocol.md](../governance/economic-protocol.md) | PHASE 4 |
| **Zaps / Tips** | NIP-57 Lightning Zaps | [nostr-events.md](./nostr-events.md) | PHASE 2 |

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
│  ├── Nostr (transport, events Kind 30017–30026)   [IMPLEMENTED] │
│  ├── Ticket System (8 types)                      [PHASE 1-5]  │
│  ├── COL-Points (reputation, off-chain)           [PHASE 3]    │
│  └── COLA Token (governance, RSK)                 [PHASE 4]    │
├─────────────────────────────────────────────────────────────────┤
│  Payment Layer (Multi-Layer Bitcoin Architecture)                │
│  ├── Bitcoin L1 (security anchor)                 [ALWAYS]     │
│  ├── Lightspark Grid (everyday payments)          [PHASE 2]    │
│  │   └── Spark Stablecoins (optional)             [PHASE 2]    │
│  ├── LNBits Hold Invoices (escrow)                [PHASE 2]    │
│  └── RSK (governance + complex contracts)         [PHASE 4]    │
│      └── Codex Forks (economic sub-units)         [PHASE 4]    │
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

### Phase 1 (Implemented) — Single Invoice

```
Buyer requests ticket creation -> Lightning invoice generated (LNBits or mock BOLT11)
  → Status: PENDING

Buyer pays invoice in Lightning wallet
  → Status: IN_PROGRESS

Completion:
  → Status: COMPLETED
```

### Phase 2 (Planned) — Three-Phase Escrow via Lightspark Grid

```
Ticket created → Phase 1 Hold Invoice (25% reservation)
Buyer pays Phase 1 → Status: ACCEPTED
Seller starts work → Phase 2 Invoice (50% delivery)
Buyer pays Phase 2 → Status: IN_PROGRESS
Buyer confirms receipt → Phase 3 Invoice (25% final)
Buyer pays Phase 3 → Status: COMPLETED

Timeout without payment → Hold Invoice expires → auto-CANCELLED
```

Full escrow specification: [escrow-protocol.md](./escrow-protocol.md)
Full payment layer specification: [payment-architecture.md](./payment-architecture.md)

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
| 30420 | DAO Governance Proposal | IMPLEMENTED (ADR-128) |
| 30421 | DAO Governance Vote | IMPLEMENTED (ADR-128) |
| 30422 | DAO Proposal Outcome | IMPLEMENTED (ADR-128) |

NIP-57 (Lightning Zaps) is also supported via Lightspark Grid in Phase 2 for tipping and micro-rewards.

---

## Security Boundaries

| Rule | Detail |
|------|--------|
| Private keys never leave the client | LNURL-Auth transmits only the Schnorr signature |
| No Bitcoin custody | Server holds no BTC; LNBits Admin Key is server-side env var only |
| No raw biometrics | Humanode biotoken is on-device hash; no raw data transmitted |
| Pubkey = identifier, not secret | May be public; used as permanent pseudonymous ID |
| Escrow freeze on dispute | Once `DISPUTED` set, escrow only released by arbitrator event |
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

## Phase 1 Known Gaps (Backlog)

| Gap | Impact | Resolution |
|-----|--------|-----------|
| No push notifications | Seller not notified of new tickets | Phase 2 |
| No Lightspark integration | Using LNBits mock only | Phase 2 |
| No HID enforcement | All transactions anonymous | Phase 3 (opt-in), Phase 4 (enforced) |


---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
