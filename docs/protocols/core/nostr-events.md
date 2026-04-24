# Colabonate Nostr Event Kinds

**Normativity:** Normative
**Version:** 1.0.0-draft
**Date:** 2026-04-11
**Status:** [IMPLEMENTED] Kind 30017, 30018/30407, 30019/30408, 30020, 30021, 30022, 30024, 30027 (marketplace/governance/reputation) | [IMPLEMENTED] Kind 30414, 30415 (NIP-C cooperation) | [IMPLEMENTED] Kind 30420–30422 (DAO governance) | [PHASE 3] Kind 30023, 30026 (identity attestation/proximity) | [PHASE 4] Kind 30025 (COLA staking)

---

> **⚠️ NIP-15 CONFLICT WARNING (ADR-101)**
> The kind range `30017–30019` conflicts with [NIP-15 (Nostr Marketplace)](https://github.com/nostr-protocol/nips/blob/master/15.md):
> - `30017` (Colabonate Offer) → Use `30402` (NIP-99 Classified Listings) instead
> - `30018` (Ticket Created) → Use `30407` instead
> - `30019` (Ticket Status) → Use `30408` instead
>
> **Resolution:** Dual-publishing to both legacy (30017–30027) and NIP-compliant (30402–30409) kinds.
> See [ADR-101: NIP-15 Kind Conflict Resolution](../../decisions/101-nip-15-kind-conflict.md) for full details.

---

## Overview

Colabonate uses the [Nostr protocol](https://github.com/nostr-protocol/nostr) (NIP-01) as its decentralized transport and event layer. All protocol interactions that require auditability, decentralization, or cross-client interoperability are published as Nostr events.

Colabonate uses **addressable events** (kinds 30000–39999 per NIP-01) because:
- They are replaceable by identifier (`d` tag), enabling state updates
- They can be queried by `pubkey` + `kind` + `d`-tag combinations
- They persist on relays even if the creator goes offline

**Kind range reserved:** `30017` – `30027` (internal convention pending NIP registration; see [NIP registration pathway](#nip-registration-pathway))

---

## Protocol Layer Architecture

```
Colabonate Nostr Event Layers:

Kind 30017  — Marketplace layer    (Offers)
Kind 30018  — Transaction layer    (Ticket creation)
Kind 30019  — Transaction layer    (Ticket status updates)
Kind 30020  — Dispute layer        (Dispute opened)
Kind 30021  — Identity layer       (Verification credentials, soulbound)
Kind 30022  — Governance layer     (Governance votes)
Kind 30023  — Identity layer       (HID attestation)
Kind 30024  — Reputation layer     (COL-Points score, reviews)
Kind 30025  — Economic layer       (COLA token stake events)
Kind 30026  — Identity layer       (Proximity proof)
Kind 30027  — Marketplace layer    (Company Profile)

External NIPs (not Colabonate-defined):
Kind 9734   — Economic layer       (NIP-57: Lightning Zap Request)
Kind 9735   — Economic layer       (NIP-57: Lightning Zap Receipt / settlement proof)

NIP-C Cooperation Layer (draft):
Kind 30414  — Cooperation Proposal  (NIP-C: proposal + partner acceptance)
Kind 30415  — Milestone Event       (NIP-C: per-milestone completion proof)
```

---

## Tag Reference

All Colabonate events use standard NIP-01 tags plus the following custom tags:

| Tag | Usage | Events |
|-----|-------|--------|
| `["d", "<id>"]` | Addressable identifier (all kinds) | All |
| `["title", "<text>"]` | Human-readable title | 30017, 30022 |
| `["price", "<sats>", "sats"]` | Price in satoshis | 30017 |
| `["category", "<cat>"]` | Offer category | 30017 |
| `["ticket", "<ticket-id>"]` | Reference to ticket | 30018, 30019, 30020 |
| `["offer", "<offer-id>"]` | Reference to offer | 30018 |
| `["buyer", "<pubkey>"]` | Buyer pubkey | 30018, 30019 |
| `["seller", "<pubkey>"]` | Seller pubkey | 30018, 30019 |
| `["status", "<status>"]` | Ticket/escrow status | 30019, 30020 |
| `["dispute_reason", "<text>"]` | Dispute reason | 30020 |
| `["credential_type", "<type>"]` | Type of credential | 30021 |
| `["hid_level", "<0-3>"]` | Identity level | 30021, 30023 |
| `["proposal", "<id>"]` | Governance proposal ID | 30022 |
| `["vote", "yes\|no\|abstain"]` | Vote choice | 30022 |
| `["voting_model", "1p1v\|token\|reputation"]` | Voting mechanism used | 30022 |
| `["cola_staked", "<amount>"]` | COLA staked for token-weighted vote | 30022 |
| `["humanode_proof", "<zk-commitment>"]` | Humanode ZK proof reference | 30023 |
| `["verifier_1", "<pubkey>"]` | First verifier pubkey | 30023 |
| `["verifier_2", "<pubkey>"]` | Second verifier pubkey | 30023 |
| `["validity_years", "<n>"]` | HID validity duration | 30023 |
| `["col_points", "<amount>"]` | COL-Points balance | 30024 |
| `["score", "<0.0-5.0>"]` | Reputation score | 30024 |
| `["review_for", "<pubkey>"]` | Subject of review | 30024 |
| `["stars", "<1-5>"]` | Star rating | 30024 |
| `["cola_amount", "<amount>"]` | COLA token amount | 30025 |
| `["stake_action", "stake\|unstake"]` | Stake or unstake | 30025 |
| `["lock_until", "<unix-timestamp>"]` | Stake lock expiry | 30025 |
| `["location_hash", "<hash>"]` | Privacy-preserving location hash | 30026 |
| `["proof_record", "<cid>"]` | Reference to proof record | 30026 |
| `["proximity_verifier", "<pubkey>"]` | Verifier pubkey | 30026 |
| `["radius", "<km>", "coop"]` | Cooperation radius in km | 30027 |
| `["fields", "<unspsc-code>"]` | Business field UNSPSC codes | 30027 |
| `["logo", "<url>"]` | Company logo URL | 30027 |
| `["banner", "<url>"]` | Company banner URL | 30027 |
| `["images", "<url-1>", "<url-2>"]` | Company image gallery URLs (single tag, multiple values) | 30027 |
| `["amount", "<msats>"]` | Amount in millisatoshis | 9734 (NIP-57) |
| `["bolt11", "<invoice>"]` | Lightning invoice | 9735 (NIP-57) |
| `["preimage", "<hex>"]` | Payment preimage | 9735 (NIP-57) |
| `["lnurl", "<url>"]` | LNURL-pay endpoint | 9734 (NIP-57) |

---

## Kind 30017 — Offer

**Status:** [IMPLEMENTED] (informal convention)
**Layer:** Marketplace
**Purpose:** A seller publishes a listing to the decentralized marketplace.

### Schema

```json
{
  "kind": 30017,
  "pubkey": "<seller-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<offer-id>"],
    ["title", "<offer-title>"],
    ["price", "<amount-sats>", "sats"],
    ["category", "<category-slug>"]
  ],
  "content": "<offer-description>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Format | Description |
|-----|--------|-------------|
| `d` | UUID or slug | Unique offer identifier |
| `title` | String (max 120 chars) | Human-readable offer title |
| `price` | Integer string + "sats" | Price in satoshis |

### Optional Tags

| Tag | Format | Description |
|-----|--------|-------------|
| `category` | Slug (e.g. "services/tech") | Offer category |
| `expiry` | Unix timestamp | When offer expires |
| `location` | ISO 3166-1 country code | Service location (Legacy) |
| `country` | ISO 3166-1 alpha-2 | Geo filtering |
| `region` | String | Geo filtering |
| `city` | String | Geo filtering |

### Content

Free-text description of the offer. Markdown supported. Max 4096 characters.

### Validation Rules

| Rule | Scope |
|------|-------|
| `pubkey` must match valid secp256k1 pubkey | Client + Relay |
| `sig` must be valid Schnorr signature over event hash | Client + Relay |
| `price` integer must be > 0 | Client |
| `d` tag must be unique per pubkey | Client |

### Offer Status

Offers have no explicit status field in the event. Status is determined by:
- Absence = ACTIVE
- Replacement event with `content: ""` and `price: "0"` = CLOSED
- No update within expiry timestamp = EXPIRED

---

## Kind 30018 — Ticket Created

**Status:** [IMPLEMENTED] as NIP-compliant Kind 30407 (legacy 30018 dual-published per ADR-101/106)
**Layer:** Transaction
**Purpose:** A buyer creates a ticket (contract) against an offer. Published by the buyer after ticket creation.
**Draft:** [docs/plans/NIP-B-transaction-layer.md](../../plans/NIP-B-transaction-layer.md)

### Schema

```json
{
  "kind": 30018,
  "pubkey": "<buyer-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<ticket-id>"],
    ["ticket", "<ticket-id>"],
    ["offer", "<offer-id>"],
    ["buyer", "<buyer-pubkey>"],
    ["seller", "<seller-pubkey>"],
    ["status", "PENDING"]
  ],
  "content": "<optional-buyer-message>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Ticket ID (same as `ticket` tag) |
| `ticket` | Unique ticket identifier |
| `offer` | Reference to the associated Kind 30017 offer |
| `buyer` | Buyer's pubkey |
| `seller` | Seller's pubkey |
| `status` | Initial status: always `PENDING` |

### Optional Tags

| Tag | Description |
|-----|-------------|
| `dao_id` | DAO identifier if ticket is bound to a DAO+Codex |
| `codex_hash` | Hash of the Codex version at time of binding |
| `ticket_type` | `smart_order` \| `milestone` \| `cooperation` (default: `smart_order`) |

### Content

Optional message from buyer to seller. Markdown supported. Max 2048 characters.

---

## Kind 30019 — Ticket Status Update

**Status:** [IMPLEMENTED] as NIP-compliant Kind 30408 (legacy 30019 dual-published per ADR-101/106)
**Layer:** Transaction
**Purpose:** Either party updates the ticket status. Published by the party making the status change.
**Draft:** [docs/plans/NIP-B-transaction-layer.md](../../plans/NIP-B-transaction-layer.md)

### Schema

```json
{
  "kind": 30019,
  "pubkey": "<updater-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<ticket-id>"],
    ["ticket", "<ticket-id>"],
    ["status", "<new-status>"],
    ["buyer", "<buyer-pubkey>"],
    ["seller", "<seller-pubkey>"]
  ],
  "content": "<optional-note>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Ticket ID (same as `ticket` tag) |
| `ticket` | Reference to the ticket being updated |
| `status` | New status value |

### Valid Status Transitions

```
PENDING → ACCEPTED           (by seller)
PENDING → CANCELLED          (by buyer or seller)
ACCEPTED → IN_PROGRESS       (by buyer after Phase 1 escrow payment — SERVICE only)
ACCEPTED → PAID              (by buyer after escrowed payment held — PRODUCT only, Phase 1)
IN_PROGRESS → COMPLETED      (by buyer after delivery confirmation)
PAID → IN_PROGRESS           (by buyer after delivery starts — PRODUCT only, Phase 2)
IN_PROGRESS → DISPUTED       (by either party)
COMPLETED → (terminal)
CANCELLED → (terminal)
PAID → CANCELLED             (on invoice expiry or buyer cancellation)
DISPUTED → ARBITRATION_WON   (by arbitrator, releases escrow to seller)
DISPUTED → ARBITRATION_LOST  (by arbitrator, cancels escrow to buyer)
ARBITRATION_WON → (terminal)
ARBITRATION_LOST → (terminal)
```

### Authorization Rules

| Transition | OfferType | Authorized Publisher | Notes |
|-----------|-----------|---------------------|-------|
| PENDING → ACCEPTED | SERVICE, PRODUCT | Seller pubkey only | Seller approves |
| PENDING → CANCELLED | SERVICE, PRODUCT | Buyer or Seller pubkey | Either party cancels |
| ACCEPTED → IN_PROGRESS | SERVICE | Buyer pubkey only | Buyer initiates work (holds escrow) |
| ACCEPTED → PAID | PRODUCT | Buyer pubkey only | Buyer pays; escrow held, awaiting delivery |
| PAID → IN_PROGRESS | PRODUCT | Buyer pubkey only | Buyer confirms delivery start (Phase 2) |
| IN_PROGRESS → COMPLETED | SERVICE, PRODUCT | Buyer pubkey only | Buyer confirms completion |
| IN_PROGRESS → DISPUTED | SERVICE, PRODUCT | Buyer or Seller pubkey | Either party opens dispute |
| PAID → CANCELLED | PRODUCT | System (via webhook) or Buyer | Invoice expires or buyer refunds |
| DISPUTED → ARBITRATION_WON | SERVICE, PRODUCT | Arbitration Council (ADR-125/126) | Arbitrator rules seller wins |
| DISPUTED → ARBITRATION_LOST | SERVICE, PRODUCT | Arbitration Council (ADR-125/126) | Arbitrator rules buyer wins |

**PAID status meaning (ADR-121):** Escrow is held on Lightning (Hold Invoice active). Funds are cryptographically locked but not yet released. Invoice expiry may auto-transition to CANCELLED. Only valid for PRODUCT offers in Phase 1.

---

## Kind 30020 — Dispute Opened

**Status:** [IMPLEMENTED] M0 Bootstrap (ADR-125/126)
**Layer:** Dispute
**Purpose:** A party formally opens a dispute on a ticket. Initiates the 3-level dispute resolution process.

### Schema

```json
{
  "kind": 30020,
  "pubkey": "<complainant-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<dispute-id>"],
    ["ticket", "<ticket-id>"],
    ["buyer", "<buyer-pubkey>"],
    ["seller", "<seller-pubkey>"],
    ["status", "LEVEL_1"],
    ["dispute_reason", "<reason-slug>"]
  ],
  "content": "<detailed-dispute-description>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique dispute ID |
| `ticket` | Reference to disputed ticket |
| `buyer` | Buyer pubkey |
| `seller` | Seller pubkey |
| `status` | Initial level: always `LEVEL_1` |
| `dispute_reason` | One of: `non_delivery`, `quality`, `non_payment`, `terms_violation`, `other` |

### Dispute Level Status Values

| Status | Meaning |
|--------|---------|
| `LEVEL_1` | Self-service / cooling-off period (7 days) |
| `LEVEL_2` | Mediation by community mediator (14 days) |
| `LEVEL_3` | DAO court arbitration (24-48h) |
| `RESOLVED` | Dispute closed, outcome published |

### Content

Detailed description of the dispute. Evidence references (hashes of screenshots, chat logs, etc.) should be included. Max 8192 characters.

---

## Kind 30021 — Verification Credential (Soulbound)

**Status:** [IMPLEMENTED] for arbitrator credentials (ADR-126) | [PHASE 3] for identity verification
**Layer:** Identity
**Purpose:** A verifier or the protocol publishes a non-transferable credential for a user. Soulbound — bound to a pubkey and cannot be transferred.

### Schema

```json
{
  "kind": 30021,
  "pubkey": "<subject-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<credential-id>"],
    ["credential_type", "<type>"],
    ["hid_level", "<0-3>"],
    ["issued_by", "<issuer-pubkey>"]
  ],
  "content": "<credential-metadata-json>",
  "sig": "<schnorr-signature>"
}
```

### Credential Types

| `credential_type` | Description | Issuer |
|------------------|-------------|--------|
| `identity_level_1` | Nostr profile verified | Self (pubkey owner) |
| `identity_level_2` | Peer-verified (2 verifiers, Kind 30026) | Verifiers |
| `identity_level_3` | Humanode biometric verified | Protocol (automated) |
| `mediator` | Authorized to mediate disputes | Foundation DAO |
| `arbitrator` | DAO court member | DAO vote |
| `skill_verified` | Skill attestation by peer | Peer pubkey |
| `protocol_contributor` | Protocol contribution badge | Foundation |

### Content (JSON)

```json
{
  "credential_type": "identity_level_2",
  "subject": "<subject-pubkey>",
  "issued_at": 1234567890,
  "expires_at": null,
  "proof_reference": "<kind-30026-event-id>",
  "notes": ""
}
```

### Soulbound Guarantee

A credential published as Kind 30021 is bound to the `pubkey` field of the event. Since Nostr events are signed with the private key corresponding to that pubkey, and the protocol never takes custody of private keys, the credential cannot be transferred to another pubkey without re-issuance by the original issuer.

---

## Kind 30022 — Governance Vote

**Status:** [IMPLEMENTED] for DAO governance + arbitration verdicts (ADR-128)
**Layer:** Governance
**Purpose:** A verified participant publishes their vote on a governance proposal.

### Schema

```json
{
  "kind": 30022,
  "pubkey": "<voter-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<vote-id>"],
    ["proposal", "<proposal-id>"],
    ["vote", "yes"],
    ["voting_model", "1p1v"],
    ["hid_level", "3"]
  ],
  "content": "<optional-rationale>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique vote ID |
| `proposal` | Governance proposal identifier |
| `vote` | `yes` \| `no` \| `abstain` |
| `voting_model` | `1p1v` \| `token` \| `reputation` |

### Optional Tags (model-specific)

| Tag | Required for Model | Description |
|-----|-------------------|-------------|
| `hid_level` | `1p1v` | Must be `3` for 1P1V (HID Level 3 required) |
| `cola_staked` | `token` | COLA tokens staked to cast this vote |
| `reputation_score` | `reputation` | COL-Points score at time of vote |

### Governance Proposal Types and Quorum Requirements

| Proposal Type | Quorum | Majority | Duration |
|--------------|--------|---------|----------|
| Normal decision | 10% | >50% | 7 days |
| Codex amendment | 20% | 2/3 | 21 days |
| Treasury spending | 15% | >50% | 14 days |
| Sanction/ban | 10% | 2/3 | 7 days |
| Protocol upgrade | 25% | 2/3 | 21 days |

### Sub-types via Content

The `content` field can carry a sub-type tag for DAO creation proposals:

```json
{
  "sub_type": "dao_creation" | "dao_amendment" | "protocol_upgrade" | "treasury" | "sanction" | "arbitration_verdict",
  "dao_id": "<target-dao-id-if-applicable>",
  "rationale": "<optional free text>"
}
```

**`arbitration_verdict` sub-type (ADR-125):** Used by arbitrators to publish verdicts in Level 3 disputes. Content JSON:
```json
{
  "sub_type": "arbitration_verdict",
  "dao_id": "colabonate-foundation",
  "codex_hash": "<sha256>",
  "codex_version": "1.0.0-draft",
  "dispute_id": "<kind-30020-event-id>",
  "ticket_id": "<ticket-uuid>",
  "outcome": "RELEASE" | "CANCEL" | "SPLIT",
  "split_ratio": { "buyer": 0.5, "seller": 0.5 },
  "reasoning": "<arbitrator-rationale>",
  "evidence_reviewed": ["<event-id-1>", "<event-id-2>"]
}
```
Tags: `["proposal", "<dispute-id>"]`, `["vote", "yes"]`, `["voting_model", "1p1v"]` (bootstrap, HID not enforced). — See [arbitration-council.md](../governance/arbitration-council.md)

**Dissent retention:** All submitted verdicts (including dissenting votes) are permanent and public. Dissenting verdicts are not deleted upon dispute resolution. The audit trail retains the complete voting record.

---

## Kind 5 — Deletion (NIP-09)

**Status:** [IMPLEMENTED] — M2 revocation
**Purpose:** Deletion of a previously published Nostr event (e.g., credential revocation).

Published by admin when revoking a member's Kind 30021 credential (ADR-126).

```json
{
  "kind": 5,
  "pubkey": "<admin-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["e", "<kind-30021-event-id-to-revoke>"]
  ],
  "content": "Credential revocation due to membership revoke"
}
```

| Tag | Description |
|-----|-------------|
| `e` | Event ID of the Kind 30021 credential to revoke |

The deletion is signed by the admin's key and published to relays. The original credential event remains on relays but is marked deleted per NIP-09.

**Status:** [PHASE 3]
**Layer:** Identity
**Purpose:** Published after successful Humanode Biomapper verification. Records the zero-knowledge proof of biometric uniqueness. Grants Identity Level 3.

### Schema

```json
{
  "kind": 30023,
  "pubkey": "<subject-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<hid-attestation-id>"],
    ["hid_level", "3"],
    ["humanode_proof", "<zk-commitment-hash>"],
    ["verifier_1", "<verifier-1-pubkey>"],
    ["verifier_2", "<verifier-2-pubkey>"],
    ["validity_years", "7"]
  ],
  "content": "<attestation-metadata-json>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique attestation ID |
| `hid_level` | Always `3` for Humanode-verified HID |
| `humanode_proof` | SHA-256 hash of the Humanode ZK commitment |
| `verifier_1` | First in-person verifier's Nostr pubkey |
| `verifier_2` | Second in-person verifier's Nostr pubkey |
| `validity_years` | Default: `7` (per Humanode's validity window) |

### Content (JSON)

```json
{
  "attestation_version": "1.0",
  "subject_pubkey": "<subject-pubkey>",
  "humanode_biotoken_hash": "<irreversible-on-device-hash>",
  "proof_record_ids": ["<kind-30026-event-id-1>", "<kind-30026-event-id-2>"],
  "attested_at": 1234567890,
  "expires_at": 1456789012
}
```

### Privacy Guarantees

- **No raw biometric data** is transmitted or stored in this event
- The `humanode_proof` field contains only the SHA-256 hash of the ZK commitment — a one-way hash that cannot be reversed to reveal biometric data
- On-device processing: the biotoken is generated locally; only the hash leaves the device
- The ZK commitment proves uniqueness (one person, one HID) without revealing identity

### Verification Procedure Reference

See [docs/protocols/identity/identity-protocol.md](../identity/identity-protocol.md) for the complete in-person verification procedure that precedes this event.

---

## Kind 30024 — Reputation Score Update / Review

**Status:** [IMPLEMENTED] (reviews.ts, ADR-071)
**Layer:** Reputation
**Purpose:** Published after a completed ticket to record a mutual review and update the reputation score. One event per review (buyer reviews seller, seller reviews buyer — two separate events).

### Schema

```json
{
  "kind": 30024,
  "pubkey": "<reviewer-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<review-id>"],
    ["review_for", "<reviewed-pubkey>"],
    ["ticket", "<ticket-id>"],
    ["stars", "5"],
    ["col_points", "150"],
    ["score", "4.7"]
  ],
  "content": "<review-text>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique review ID |
| `review_for` | Pubkey of the person being reviewed |
| `ticket` | Reference to the completed ticket |
| `stars` | Integer 1–5 |

### Optional Tags

| Tag | Description |
|-----|-------------|
| `col_points` | Reviewer's current COL-Points total (for weight calculation) |
| `score` | Computed reputation score of the reviewed party (float 0.0–5.0) |

### Content

Free-text review. Max 2048 characters. Markdown not rendered — plain text only.

### Review Protocol Rules

- Reviews are only valid when the referenced ticket has status `COMPLETED`
- Reviews must be published within 14 days of ticket completion
- Both parties must review for COL-Points to be awarded (mutual review)
- Reviews cannot be deleted — they are permanent Nostr events
- Manipulation protection: reviewer must have at least 1 prior completed ticket before their reviews receive weight

### Reputation Score Computation

See [docs/protocols/core/reputation-protocol.md](./reputation-protocol.md) for the full scoring formula.

---

## Kind 30025 — COLA Token Stake Event

**Status:** [PHASE 4]
**Layer:** Economic / Governance
**Purpose:** A participant publishes a stake or unstake action for COLA governance tokens. Required for token-weighted governance voting.

### Schema

```json
{
  "kind": 30025,
  "pubkey": "<staker-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<stake-event-id>"],
    ["cola_amount", "1000"],
    ["stake_action", "stake"],
    ["lock_until", "1456789012"]
  ],
  "content": "<optional-note>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Values | Description |
|-----|--------|-------------|
| `d` | UUID | Unique stake event ID |
| `cola_amount` | Integer string | COLA tokens being staked/unstaked |
| `stake_action` | `stake` \| `unstake` | Direction of action |

### Optional Tags

| Tag | Description |
|-----|-------------|
| `lock_until` | Unix timestamp of stake lock expiry (for fixed-lock staking) |
| `purpose` | `governance` \| `protocol_royalty` \| `liquidity` |

### Economic Rules

- Minimum stake for governance: 1 COLA
- Staking APY: 5% (distributed quarterly, governed by DAO)
- Unstaking lockup: 30 days (configurable by DAO governance vote)
- COLA is not a currency — all actual payments remain in Bitcoin sats

See [docs/protocols/governance/economic-protocol.md](../governance/economic-protocol.md) for full token economics.

---

## Kind 30026 — Proximity Proof

**Status:** [PHASE 3]
**Layer:** Identity
**Purpose:** Published by an in-person verifier after a real-world encounter with a subject seeking Identity Level 2. Two such events (two different verifiers) are required for Level 2 confirmation.

### Schema

```json
{
  "kind": 30026,
  "pubkey": "<verifier-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<proof-id>"],
    ["proximity_verifier", "<verifier-pubkey>"],
    ["p", "<subject-pubkey>"],
    ["location_hash", "<sha256-of-location-salt>"],
    ["proof_record", "<encrypted-proof-record-reference>"]
  ],
  "content": "<verifier-signed-attestation-json>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique proof ID |
| `proximity_verifier` | Verifier's Nostr pubkey (same as `pubkey`) |
| `p` | Subject's Nostr pubkey (NIP-01 contact tag, enables filtering) |
| `location_hash` | SHA-256 of (location-string + random-salt) — privacy-preserving location proof |

### Optional Tags

| Tag | Description |
|-----|-------------|
| `proof_record` | Reference to encrypted proof record stored off-relay |

### Content (JSON, signed by verifier)

```json
{
  "attestation_version": "1.0",
  "verifier_pubkey": "<verifier-pubkey>",
  "subject_pubkey": "<subject-pubkey>",
  "encounter_type": "in_person",
  "document_partial_match": true,
  "joint_scan_validated": true,
  "attested_at": 1234567890,
  "verifier_signature": "<additional-signature-over-content>"
}
```

### Two-Verifier Requirement

Identity Level 2 requires **two independent Proximity Proof events** (Kind 30026) from **two different verifier pubkeys** pointing to the same subject pubkey. The subject's identity-protocol.md handler aggregates these and issues a Kind 30021 credential upon confirmation.

See [docs/protocols/identity/proximity-proof.md](../identity/proximity-proof.md) for the complete verifier procedure.

---

## Kind 30027 — Company Profile

**Status:** [IMPLEMENTED]
**Layer:** Marketplace
**Purpose:** A company owner publishes their business profile to the decentralized directory. Enables cross-platform discovery and cooperation radius filtering.

### Schema

```json
{
  "kind": 30027,
  "pubkey": "<owner-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<company-id>"],
    ["name", "<company-name>"],
    ["address", "<street-address>"],
    ["city", "<city>"],
    ["zip", "<zip-code>"],
    ["country", "<iso-3166-code>"],
    ["radius", "<km>", "coop"],
    ["fields", "<unspsc-code-1>", "<unspsc-code-2>"],
    ["logo", "<url>"],
    ["banner", "<url>"],
    ["images", "<url-1>", "<url-2>"],
    ["website", "<url>"],
    ["email", "<email>"],
    ["phone", "<phone>"],
    ["twitter", "<handle>"],
    ["nostr", "<pubkey>"]
  ],
  "content": "<company-description-json>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Format | Description |
|-----|--------|-------------|
| `d` | UUID or slug | Unique company identifier |
| `name` | String (max 120 chars) | Human-readable company name |

### Optional Tags

| Tag | Format | Description |
|-----|--------|-------------|
| `address` | String | Street address |
| `city` | String | City name |
| `zip` | String | ZIP/postal code |
| `country` | ISO 3166-1 alpha-2 | Country code (e.g., "DE", "US") |
| `region` | String | Region or state |
| `radius` | Integer + "coop" | Cooperation radius in km |
| `fields` | UNSPSC codes | Business field categories (multiple allowed) |
| `logo` | URL | Company logo image URL |
| `banner` | URL | Company banner image URL |
| `images` | URLs (multiple values in single tag) | Company image gallery |
| `website` | URL | Company website URL |
| `email` | Email address | Contact email |
| `phone` | Phone number | Contact phone |
| `twitter` | Handle | Twitter/X handle (without @) |
| `nostr` | Pubkey | Nostr pubkey for contact |

### Content (JSON)

```json
{
  "description": "<company-description-text>",
  "opening_hours": {
    "mon": "09:00-17:00",
    "tue": "09:00-17:00",
    "wed": "09:00-17:00",
    "thu": "09:00-17:00",
    "fri": "09:00-15:00",
    "sat": null,
    "sun": null
  },
  "terms": "<terms-and-conditions-html-or-markdown>"
}
```

### Business Fields (UNSPSC Codes)

Business fields use UNSPSC codes per ADR-059 (Standardized Category Protocol). Each `fields` tag contains one UNSPSC code representing a business category.

Examples:
- `["fields", "50151500"]` — Office supplies
- `["fields", "80101500"]` — IT consulting services
- `["fields", "55121500"]` — Food service

Multiple fields allowed: `["fields", "80101500", "80111600"]` (IT consulting + software development)

### Cooperation Radius

The `["radius", "<km>", "coop"]` tag defines the geographic range for cooperation:
- Other clients can filter companies within this radius
- Used for local business discovery
- Units: kilometers (km)

### Validation Rules

| Rule | Scope |
|------|-------|
| `pubkey` must match valid secp256k1 pubkey | Client + Relay |
| `sig` must be valid Schnorr signature over event hash | Client + Relay |
| `d` tag must be unique per pubkey | Client |
| `radius` integer must be > 0 if present | Client |

---

## External NIP: NIP-57 Lightning Zaps

**Status:** [PHASE 2]
**Layer:** Economic / Reputation
**Purpose:** Lightning payment events (zaps) integrated via Lightspark Grid. Used for micro-rewards, tipping, and COL-Points incentive events. Colabonate uses the standard NIP-57 event kinds — these are **not** Colabonate-defined kinds.

### NIP-57 Event Kinds Used

| Kind | Name | Direction | Usage in Colabonate |
|------|------|-----------|---------------------|
| `9734` | Zap Request | Sender → Relay | Initiates a zap payment to a recipient pubkey |
| `9735` | Zap Receipt | LNURL Server → Relay | Published by Lightspark Grid upon payment settlement; proof of zap |

### Kind 9734 — Zap Request (Sender-side)

Published by the zap sender before payment. Routed to the recipient's LNURL server.

```json
{
  "kind": 9734,
  "pubkey": "<sender-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["p", "<recipient-pubkey>"],
    ["e", "<event-id-being-zapped>"],
    ["amount", "<millisatoshis>"],
    ["relays", "<relay-1>", "<relay-2>"],
    ["lnurl", "<recipient-lnurl>"]
  ],
  "content": "<optional-zap-message>",
  "sig": "<schnorr-signature>"
}
```

| Tag | Required | Description |
|-----|----------|-------------|
| `p` | Yes | Recipient's Nostr pubkey |
| `amount` | Yes | Amount in millisatoshis |
| `relays` | Yes | Relays where the receipt should be published |
| `e` | No | Event being zapped (offer, ticket, or review) |
| `lnurl` | No | Recipient's LNURL-pay endpoint |

### Kind 9735 — Zap Receipt (Settlement proof)

Published by Lightspark Grid (acting as LNURL server) after the Lightning payment is settled. This is the authoritative proof that a zap occurred.

```json
{
  "kind": 9735,
  "pubkey": "<lnurl-server-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["p", "<recipient-pubkey>"],
    ["e", "<event-being-zapped>"],
    ["bolt11", "<lightning-invoice>"],
    ["description", "<zap-request-event-json>"],
    ["preimage", "<payment-preimage>"]
  ],
  "content": "",
  "sig": "<schnorr-signature>"
}
```

| Tag | Required | Description |
|-----|----------|-------------|
| `p` | Yes | Recipient pubkey |
| `bolt11` | Yes | The Lightning invoice that was paid |
| `description` | Yes | The original Kind 9734 Zap Request event (JSON string) |
| `e` | No | Event that was zapped |
| `preimage` | No | Payment preimage (proof of payment) |

### Colabonate-specific Zap Rules

1. **COL-Points from Zaps** — In Phase 2+, zaps on completed tickets or reviews may contribute to COL-Points. The DAO sets the conversion rate per governance vote.
2. **Lightspark Grid integration** — The `pubkey` of Kind 9735 receipts published via Lightspark Grid is the Lightspark LNURL server pubkey. Clients must validate this matches the Lightspark-registered pubkey for the recipient.
3. **Zap receipt validation** — Clients should verify: `bolt11` invoice amount matches `amount` in Kind 9734; `description` hash in invoice matches the Kind 9734 event JSON; `preimage` satisfies the invoice payment hash.
4. **Spam protection** — Zaps below 1 sat (< 1000 msats) are ignored for COL-Points purposes.

### Reference

- [NIP-57 Specification](https://github.com/nostr-protocol/nostr/blob/master/57.md)
- [Lightspark Grid](https://lightspark.com) — Performance layer providing LNURL-pay endpoint
- [docs/protocols/core/payment-architecture.md](./payment-architecture.md) — Lightspark Grid integration context

---

## Kind 30414 — Cooperation Proposal

**Status:** [IMPLEMENTED] NIP-C (ADR-108/109/119)
**Layer:** Cooperation
**Purpose:** Published by the initiator to propose a cooperation; partner accepts by publishing an acceptance event with the same `d` tag, forming a cryptographically signed agreement.
**Draft:** [docs/plans/NIP-C-cooperation-layer.md](../../plans/NIP-C-cooperation-layer.md)

### Schema

```json
{
  "kind": 30414,
  "pubkey": "<initiator-or-partner-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d",          "<cooperation-id>"],
    ["type",       "proposal" | "acceptance"],
    ["cooperation","<cooperation-id>"],
    ["initiator",  "<initiator-pubkey>"],
    ["partner",    "<partner-pubkey>"],
    ["p",          "<counterparty-pubkey>"],
    ["title",      "<cooperation-title>"],
    ["budget",     "<sats>", "sats"],
    ["status",     "OPEN" | "ACCEPTED" | "ACTIVE" | "COMPLETED" | "CANCELLED"],
    ["ticket_type","cooperation"]
  ],
  "content": "<proposal-description>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique cooperation identifier (UUID) |
| `type` | `proposal` (initiator) or `acceptance` (partner) |
| `cooperation` | Same as `d`; redundancy aids filtering |
| `initiator` | Initiator's Nostr pubkey |
| `partner` | Partner's Nostr pubkey |
| `title` | Human-readable cooperation title |
| `budget` | Total budget in satoshis |
| `status` | Cooperation status (OPEN/ACCEPTED/ACTIVE/COMPLETED/CANCELLED) |
| `ticket_type` | Always `cooperation` |

### Optional Tags

| Tag | Description |
|-----|-------------|
| `timeline` | Expected duration or deadline |
| `cooperation_type` | `transactional` \| `production` \| `sales` \| `service` \| `educational` \| `crowdfunding` \| `barter` |
| `skills` | Comma-separated required skills |

### Agreement Formation

The cooperation agreement is formed by two Kind 30414 events with the same `d` tag:
1. Initiator's proposal event (`type=proposal`, `status=OPEN`)
2. Partner's acceptance event (`type=acceptance`, `status=ACCEPTED`)

Because Kind 30414 is addressable, the partner's acceptance replaces the proposal at the
`(partner-pubkey, kind, d)` triple. Both events are independently signed, constituting
the complete cryptographically verifiable agreement.

---

## Kind 30415 — Milestone Event

**Status:** [IMPLEMENTED] NIP-C (ADR-113/120)
**Layer:** Cooperation
**Purpose:** Published by the performer to report milestone completion; initiator accepts to release payment.
**Draft:** [docs/plans/NIP-C-cooperation-layer.md](../../plans/NIP-C-cooperation-layer.md)

### Schema

```json
{
  "kind": 30415,
  "pubkey": "<completer-or-initiator-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d",              "<cooperation-id>-<milestone-index>"],
    ["cooperation",     "<cooperation-id>"],
    ["milestone",       "<milestone-id>"],
    ["milestone_index", "<1-based-index>"],
    ["initiator",       "<initiator-pubkey>"],
    ["partner",         "<partner-pubkey>"],
    ["p",               "<counterparty-pubkey>"],
    ["title",           "<milestone-title>"],
    ["status",          "COMPLETED" | "ACCEPTED" | "DISPUTED"],
    ["amount",          "<sats>", "sats"],
    ["released",        "true" | "false"]
  ],
  "content": "<completion-note>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Composite: `<cooperation-id>-<milestone-index>` |
| `cooperation` | Reference to the parent cooperation |
| `milestone` | Milestone database ID |
| `milestone_index` | 1-based milestone sequence number |
| `initiator` | Cooperation initiator's pubkey |
| `partner` | Cooperation partner's pubkey |
| `title` | Milestone title |
| `status` | `COMPLETED` (performer) / `ACCEPTED` (initiator) / `DISPUTED` |

### Optional Tags

| Tag | Description |
|-----|-------------|
| `evidence` | URL or hash of deliverable proof |
| `released` | `true` if payment was released (set by initiator on ACCEPTED) |

### Status Transitions

```
COMPLETED ──► ACCEPTED   (initiator approves; payment released)
COMPLETED ──► DISPUTED   (initiator rejects)
```

---

## NIP-99 Compliance: Dual-Publishing Layer

**Status:** [IMPLEMENTED] (ADR-101/106)

To comply with [NIP-99 (Classified Listings Standard)](https://github.com/nostr-protocol/nips/blob/master/99.md), Colabonate dual-publishes marketplace and transaction events in both legacy (30017–30019) and NIP-compliant (30402–30408) kinds.

| Legacy Kind | NIP-99 Kind | Purpose | Notes |
|-------------|-------------|---------|-------|
| 30017 | 30402 | Classified Listing (Offer) | Marketplace layer |
| 30018 | 30407 | Listing/Offer Interaction (Ticket) | Transaction layer |
| 30019 | 30408 | Listing Status Update | Transaction state machine |

**Implementation:** Both kinds are published in parallel. Clients may subscribe to either range. Legacy kinds (30017–30019) will be deprecated after a deprecation period (see [ADR-101](../../decisions/101-nip-15-kind-conflict.md) for timeline).

**Schemas:** NIP-99 kinds 30402, 30407, 30408 use identical schemas to their legacy counterparts (30017, 30018, 30019 above). No schema duplication needed here — implementers should reference the legacy kind documentation and treat the NIP-99 kinds as direct replacements.

---

## Kind 30420 — DAO Profile / Creation

**Status:** [IMPLEMENTED] (ADR-105, Foundation DAO bootstrap)
**Layer:** Governance
**Purpose:** A DAO publishes its governance profile, codex version, and member list. Addressable by DAO ID.

### Schema

```json
{
  "kind": 30420,
  "pubkey": "<dao-admin-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<dao-id>"],
    ["name", "<dao-name>"],
    ["description", "<dao-description>"],
    ["codex_version", "<semantic-version>"],
    ["codex_hash", "<sha256-hash-of-codex>"],
    ["member_count", "<int>"],
    ["voting_model", "1p1v|token|reputation"]
  ],
  "content": "<dao-metadata-json>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Unique DAO identifier (e.g., "colabonate-foundation") |
| `name` | Human-readable DAO name |
| `codex_version` | Semantic version of the current Codex (e.g., "1.0.0") |
| `codex_hash` | SHA-256 hash of the full Codex text (for verification) |

### Optional Tags

| Tag | Description |
|-----|-------------|
| `description` | DAO mission / description text |
| `voting_model` | Primary voting mechanism: `1p1v` (Humanode), `token` (COLA-weighted), `reputation` (COL-Points) |
| `member_count` | Approximate active member count |

### Content (JSON)

```json
{
  "dao_id": "<dao-id>",
  "founding_pubkey": "<founder-pubkey>",
  "created_at": 1234567890,
  "codex_url": "<https-url-to-codex>",
  "voting_rules": {
    "quorum_percent": 10,
    "majority_percent": 50
  }
}
```

---

## Kind 30421 — DAO Codex (Governance Rules)

**Status:** [IMPLEMENTED] (ADR-111)
**Layer:** Governance
**Purpose:** The authoritative record of a DAO's governance rules, dispute resolution procedures, and operational codex. Versioned and content-addressed.

### Schema

```json
{
  "kind": 30421,
  "pubkey": "<dao-admin-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<dao-id>-<codex-version>"],
    ["dao_id", "<dao-id>"],
    ["codex_version", "<semantic-version>"],
    ["title", "<codex-title>"],
    ["effective_date", "<unix-timestamp>"],
    ["previous_codex_version", "<prior-version-if-amendment>"]
  ],
  "content": "<full-codex-markdown-or-html>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Description |
|-----|-------------|
| `d` | Composite: `<dao-id>-<version>` enables version history queries |
| `dao_id` | Reference to the DAO |
| `codex_version` | Full semantic version (e.g., "1.0.0", "1.1.0-amendment-1") |
| `effective_date` | Unix timestamp when this codex version becomes active |

### Content

The full text of the DAO's governance codex — procedures for dispute resolution, voting quorum, proposal types, sanctions, etc. Max 65536 characters (Markdown or structured text).

---

## Kind 30422 — DAO Membership / Registration

**Status:** [IMPLEMENTED] (ADR-112, M2 multi-member council)
**Layer:** Governance
**Purpose:** A member registers with a DAO, or admin records member approval. Links a pubkey to a DAO with optional role/status.

### Schema

```json
{
  "kind": 30422,
  "pubkey": "<member-or-admin-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<dao-id>-<member-pubkey>"],
    ["dao_id", "<dao-id>"],
    ["member", "<member-pubkey>"],
    ["status", "active|pending|revoked"],
    ["role", "member|mediator|arbitrator|admin"],
    ["joined_at", "<unix-timestamp>"]
  ],
  "content": "<optional-member-profile-json>",
  "sig": "<schnorr-signature>"
}
```

### Required Tags

| Tag | Values | Description |
|-----|--------|-------------|
| `d` | `<dao-id>-<pubkey>` | Addressable by DAO + member pubkey |
| `dao_id` | DAO ID | Which DAO this member belongs to |
| `member` | Pubkey | The member's Nostr pubkey |
| `status` | `active` \| `pending` \| `revoked` | Membership state |
| `role` | `member` \| `mediator` \| `arbitrator` \| `admin` | DAO role |

### Content (JSON, optional)

```json
{
  "member_pubkey": "<pubkey>",
  "display_name": "<optional-name>",
  "hid_level": 0|1|2|3,
  "joined_at": 1234567890,
  "contribution_notes": "<optional-notes>"
}
```

### Authorization Rules

- **Self-registration**: A member can publish Kind 30422 with their own pubkey in `member` tag, setting `status=pending`.
- **Admin approval**: DAO admin publishes Kind 30422 with `status=active` or `revoked` to finalize membership.
- **Role assignment**: Only admin pubkey can set `role` to `arbitrator` or `mediator`.

---

## NIP Registration Pathway

The event kinds 30017–30027 and 30414–30415 are currently an **internal Colabonate convention** or draft NIP standard (see status markers above). They are used in production but have not been formally submitted to the Nostr protocol's NIP (Nostr Improvement Proposal) process.

**Plan for NIP registration:**

1. Stabilize all event schemas (complete this document to v1.0.0-stable)
2. Submit a NIP draft covering at minimum Kind 30017 (Offer) as the most widely useful event
3. Seek community review from the Nostr developer community
4. Submit remaining kinds as separate NIPs or as a single "Colabonate Protocol Extension" NIP
5. Update this document to reflect official NIP numbers once assigned

Until NIP registration is complete, implementers should use the kind numbers as specified here. Client software should treat these as **unofficial** kinds and not assume other non-Colabonate relays will index or retain them.

---

## Versioning

### Adding New Event Kinds

To add a new Colabonate event kind:
1. Create an ADR proposing the new kind and its rationale
2. Define the full schema following this document's format
3. Add the kind to this document with status `[CONCEPT]`
4. Implement in a non-breaking way (new kind = no schema change to existing kinds)
5. Update kind to `[PHASE N]` when implementation begins

### Breaking Changes

A change to an existing kind's required tags or content format is a **major version bump** of the protocol (`v1.x.x → v2.0.0`). This requires:
- An ADR documenting the change
- A deprecation period of at least 90 days
- Client-side version negotiation support

### Client-Side Version Declaration

Clients implementing Colabonate events should include the supported protocol version in their NIP-01 profile metadata (Kind 0):

```json
{
  "name": "My Colabonate Client",
  "colabonate_protocol_version": "1.0.0-draft"
}
```

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
