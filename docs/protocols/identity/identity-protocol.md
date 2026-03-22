# Identity Protocol – Colabonate Human Identity (HID)

**Version:** 1.0.0-draft
**Date:** 2026-03-22
**Status:** [IMPLEMENTED] Level 0 | [PHASE 3] Levels 1–3

---

## Overview

Colabonate uses a **tiered identity system** that allows participants to operate at varying levels of verified trust. The system is designed around three core principles:

1. **Sovereignty** — users control their own identity; no central KYC authority
2. **Progressive trust** — higher trust levels unlock higher-value interactions
3. **Privacy by default** — higher levels provide more trust without requiring disclosure of personally identifiable information (PII)

The identity system is built on:
- **LNURL-Auth** (LUD-04) for pseudonymous authentication
- **Nostr Protocol** (NIP-01) for identity events and attestations
- **Humanode Biomapper** for biometric Sybil resistance (Level 3)

---

## Identity Level Overview

| Level | Name | Authentication | Verification | Sybil Resistance | Governance |
|-------|------|---------------|-------------|-----------------|------------|
| 0 | Anonymous | LNURL-Auth | None | Low | None |
| 1 | Nostr Profile | LNURL-Auth + Nostr | Self-declared | Low | Read-only |
| 2 | Peer Verified | LNURL-Auth + Nostr | 2 in-person verifiers | Medium | Observe + Vote (rep-weighted) |
| 3 | HID Verified | LNURL-Auth + Nostr + Humanode | Biometric ZK proof | High | Full (1P1V eligible) |

---

## Level 0: Anonymous (LNURL-Auth)

**Status:** [IMPLEMENTED]

### Description

The base identity layer. A user authenticates by signing a challenge (`k1`) with their Lightning wallet's private key. This produces a **pubkey** that serves as their permanent pseudonymous identifier on Colabonate.

### Authentication Flow

```
1. Client requests challenge:
   GET /api/auth/lnurl
   ← { lnurl: "lnurl1...", k1: "abc123..." }

2. User opens LNURL in Lightning wallet
   Wallet signs k1 with secp256k1 key (deterministic per domain)

3. Wallet calls callback:
   GET /api/auth/lnurl/callback?k1=...&sig=...&key=...
   ← Server verifies Schnorr signature

4. Client polls status:
   GET /api/auth/lnurl/status/:k1
   ← { status: "OK", pubkey: "02a1b2c3..." }
```

### Properties

- **No password, no email, no registration**
- The pubkey is derived deterministically from the Lightning wallet seed and the service domain — same wallet, same domain = same pubkey
- The private key never leaves the user's wallet
- A user can generate different pubkeys for different services by design (privacy feature)
- The server only stores the pubkey — never the private key

### Capabilities at Level 0

| Capability | Available |
|-----------|-----------|
| Browse offers | Yes |
| Create offers | Yes |
| Create tickets | Yes |
| Pay Lightning invoices | Yes |
| Leave reviews | Yes (but unweighted) |
| Dispute resolution | Yes (Level 1 self-service only) |
| Governance voting | No |
| Mediation | No |

---

## Level 1: Nostr Profile

**Status:** [PHASE 2]

### Description

The user publishes a Nostr profile (NIP-01 Kind 0 metadata event) and optionally adds NIP-05 DNS verification. This links their LNURL-Auth pubkey to a persistent Nostr identity with optional social proof.

### Upgrade Procedure

1. User generates or connects a Nostr keypair (Ed25519 or secp256k1 per NIP-01)
2. User publishes Kind 0 metadata event including their Lightning address (`lud16`) to link LNURL-Auth identity to Nostr identity
3. (Optional) User sets up NIP-05 DNS verification (`_nip05.json` on their domain)
4. User publishes Kind 30021 credential (`credential_type: identity_level_1`) signed with their own pubkey

### Nostr Profile (Kind 0) Required Fields for Colabonate

```json
{
  "kind": 0,
  "pubkey": "<nostr-pubkey>",
  "content": {
    "name": "<display-name>",
    "lud16": "<lightning-address>",
    "colabonate_pubkey": "<lnurl-auth-pubkey>",
    "colabonate_protocol_version": "1.0.0-draft"
  }
}
```

The `colabonate_pubkey` field links the Nostr identity to the LNURL-Auth identity. This link is self-declared — it requires no verification but provides continuity across sessions.

### Capabilities Added at Level 1

| Capability | Change |
|-----------|--------|
| Reviews | Weighted by profile completeness |
| Cooperation tickets | Milestone-based cooperation enabled |
| Dispute Level 2 | Mediation access (as party) |

---

## Level 2: Peer Verified

**Status:** [PHASE 3]

### Description

The user's identity is attested by **two independent human verifiers** through an in-person encounter. The verifiers confirm:
- The person is a real human (physical presence)
- Partial document inspection (not full KYC — verifiers do not record full document details)
- A joint biometric scan producing a privacy-preserving proximity proof

This level provides meaningful Sybil resistance through social proof without requiring any central authority.

### Upgrade Procedure

#### Step 1: Request Verification

1. User applies for Level 2 verification through the Colabonate app
2. User posts a security deposit (amount determined by DAO, default: 5,000 sats) via Lightning escrow
3. The deposit is returned after successful verification or after 30 days if no verifiers are available

#### Step 2: Verifier Assignment

1. Two independent verifiers are randomly selected from the **verified verifier pool**
2. Verifiers must have:
   - Identity Level 3 (HID verified) themselves
   - Active status in the verifier pool (opted in)
   - No prior connection to the subject (no shared completed tickets within 90 days)
3. Verifiers are notified via Nostr DM

#### Step 3: In-Person Encounter

The encounter must happen in person (physical co-location). Both verifiers must be present simultaneously.

**Encounter checklist (verifiers confirm all):**
- [ ] Subject presents a physical identity document (passport, national ID, or driver's license)
- [ ] Partial document inspection: verifier confirms document type and checks that the face matches the person present (verifiers do NOT record document numbers or full name)
- [ ] Joint biometric scan: subject and both verifiers complete a fingerprint proximity scan on the Colabonate app — the app generates a privacy-preserving biotoken (hash) locally; no raw biometric data is transmitted
- [ ] Verifiers sign the Proof Record with their Nostr private key

#### Step 4: Proof Record Creation

Each verifier publishes a Kind 30026 (Proximity Proof) event:

```json
{
  "kind": 30026,
  "pubkey": "<verifier-pubkey>",
  "tags": [
    ["d", "<proof-id>"],
    ["proximity_verifier", "<verifier-pubkey>"],
    ["p", "<subject-pubkey>"],
    ["location_hash", "<sha256-of-location-plus-salt>"]
  ],
  "content": "{\"attestation_version\":\"1.0\",\"verifier_pubkey\":\"...\",\"subject_pubkey\":\"...\",\"document_partial_match\":true,\"joint_scan_validated\":true,\"attested_at\":1234567890}"
}
```

#### Step 5: Confirmation

When two independent Kind 30026 events exist for the same subject pubkey from two different verifier pubkeys, the protocol issues:
- A Kind 30021 credential (`credential_type: identity_level_2`) signed by the verifiers' combined attestation
- Return of the security deposit to the subject

### Verifier Pool Participation

To join the verifier pool, a user must:
1. Hold Identity Level 3 (Humanode biometric verified)
2. Have a minimum reputation score of 4.0/5.0 with at least 10 completed tickets
3. Opt in via a Kind 30021 event (`credential_type: verifier_pool_member`)
4. Agree to complete verification within 14 days of receiving a request

Verifiers receive COL-Points for completed verifications.

### Capabilities Added at Level 2

| Capability | Change |
|-----------|--------|
| DAO governance | Observation + reputation-weighted voting |
| Dispute mediation | Eligible to join mediator pool (after meeting reputation threshold) |
| High-value tickets | Required for tickets above protocol-defined threshold (DAO-governed) |
| DAO creation | Minimum identity level required for DAO founding members |

---

## Level 3: HID Verified (Humanode Biomapper)

**Status:** [PHASE 3]

### Description

The highest identity level. The user's Nostr pubkey is cryptographically linked to a unique human being via Humanode's biometric ZK proof system. This provides strong Sybil resistance: each unique human can hold only one Level 3 HID.

Level 3 is the prerequisite for **One Person One Vote (1P1V)** governance participation.

### Upgrade Procedure

#### Step 1: Prerequisites

- User must already hold Level 2 identity
- User must have the Colabonate app with Humanode Biomapper SDK integration

#### Step 2: On-Device Biotoken Generation

1. The Humanode Biomapper SDK scans a pair of fingerprints locally on the user's device
2. The device generates a **biotoken** — an irreversible cryptographic hash of the biometric data
3. **No raw biometric data ever leaves the device**
4. The biotoken is combined with the user's Nostr pubkey to create a **ZK commitment**

#### Step 3: Humanode Uniqueness Proof

1. The user submits the anonymous ZK commitment to Humanode's network
2. Humanode verifies that the ZK commitment does not match any existing biotoken in its network (uniqueness check)
3. If unique: Humanode issues a uniqueness credential — a ZK proof that "this biotoken belongs to a unique human not previously registered"
4. The ZK proof contains **no biometric data** — only a cryptographic certificate of uniqueness

#### Step 4: HID Attestation Event

The app publishes a Kind 30023 event:

```json
{
  "kind": 30023,
  "pubkey": "<subject-pubkey>",
  "tags": [
    ["d", "<hid-attestation-id>"],
    ["hid_level", "3"],
    ["humanode_proof", "<sha256-of-zk-commitment>"],
    ["verifier_1", "<peer-verifier-1-pubkey>"],
    ["verifier_2", "<peer-verifier-2-pubkey>"],
    ["validity_years", "7"]
  ],
  "content": "{...attestation metadata...}"
}
```

#### Step 5: Level 3 Credential Issued

A Kind 30021 credential (`credential_type: identity_level_3`, `hid_level: 3`) is issued and published.

### Validity and Renewal

- HID Level 3 is valid for **7 years** from the date of biometric verification (Humanode's validity window)
- After expiry, the user must repeat the biometric verification (in-person peer verification is not required again — only the Humanode scan)
- If a pubkey is compromised (private key exposure), the user can request a Humanode re-verification and re-bind to a new pubkey (requires contacting Foundation DAO with proof of the old Level 2 credentials)

### Privacy Guarantees

| Guarantee | Implementation |
|-----------|---------------|
| No raw biometrics stored | On-device hashing only; hash is one-way |
| No biometrics transmitted | Only the ZK commitment hash is sent to Humanode |
| No central biometric database | Humanode uses distributed ZK proof architecture |
| Pubkey-binding without identity disclosure | ZK proof proves uniqueness, not who the person is |
| Selective disclosure | Level 3 credential is visible on Nostr, but the underlying biometric data is not |

### Capabilities Added at Level 3

| Capability | Change |
|-----------|--------|
| 1P1V governance voting | Full access (one vote per verified human) |
| Verifier pool membership | Eligible to verify others' Level 2 |
| Arbitrator eligibility | DAO court membership (via DAO governance vote) |
| Maximum trust tier | Access to all protocol features |

---

## Identity Level Comparison

| Feature | Level 0 | Level 1 | Level 2 | Level 3 |
|---------|---------|---------|---------|---------|
| Marketplace (buy/sell) | Yes | Yes | Yes | Yes |
| Cooperation tickets | Yes | Yes | Yes | Yes |
| Dispute Level 1-2 | Yes | Yes | Yes | Yes |
| Dispute Level 3 | No | No | Yes | Yes |
| Review weight | Low | Medium | High | Highest |
| Governance observation | No | No | Yes | Yes |
| Token-weighted voting | No | No | Yes | Yes |
| 1P1V voting | No | No | No | Yes |
| Mediator eligibility | No | No | Yes (rep threshold) | Yes |
| Arbitrator eligibility | No | No | No | Yes (DAO vote) |
| Verifier pool | No | No | No | Yes |
| DAO founding member | No | No | Yes (min) | Yes |

---

## Migration Path

Users upgrade their identity level by completing the procedures described above. Upgrades are irreversible at the time of publication (a Level 2 user cannot "downgrade" to Level 1 — the Level 2 credential persists on Nostr relays).

A user can hold only **one active identity** per Nostr pubkey. If a user needs to change their pubkey (e.g., wallet migration), they must:
1. Create a Nostr NIP-38 key migration event linking old pubkey to new pubkey
2. Re-request verification at the appropriate level for the new pubkey

---

## Sybil Resistance Analysis

| Attack | Level 0 | Level 1 | Level 2 | Level 3 |
|--------|---------|---------|---------|---------|
| Multiple accounts (different wallets) | Trivial | Easy | Hard | Impossible* |
| Fake identity (fake documents) | N/A | N/A | Moderate risk | Very hard |
| Verifier collusion | N/A | N/A | Requires 2 colluders | Requires 2 colluders + Humanode bypass |
| Biotoken spoofing | N/A | N/A | N/A | Cryptographically infeasible |

*Level 3 resistance depends on Humanode network security. The protocol treats Humanode as a trusted external component in Phase 3. The DAO can vote to add alternative biometric providers in later phases.

---

## References

- [LNURL-Auth (LUD-04)](https://github.com/lnurl/luds/blob/luds/04.md)
- [NIP-01: Basic Nostr protocol](https://github.com/nostr-protocol/nostr/blob/master/01.md)
- [NIP-05: DNS identity verification](https://github.com/nostr-protocol/nostr/blob/master/05.md)
- [Humanode Biomapper](https://humanode.io)
- [docs/protocols/core/nostr-events.md](../core/nostr-events.md) — Kind 30021, 30023, 30026
- [docs/protocols/identity/proximity-proof.md](./proximity-proof.md) — Level 2 encounter procedure

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
