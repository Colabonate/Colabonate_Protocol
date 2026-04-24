# Proximity Proof Protocol

> **Normativity:** Normative (when implemented)
> **Stability:** Experimental
> **Status:** [PHASE 3 — not yet implemented]

Part of the Colabonate Protocol Specification · [← Back to Protocols](../README.md)

---

## Purpose

Proximity Proof is the Identity Level 2 verification mechanism. It enables a pubkey to move from Level 1 (self-declared) to Level 2 (peer-attested) without requiring biometric ZK verification (Level 3 / HID). Two independent, previously-verified peers attest — via signed Nostr events — that they met the subject in person.

Level 2 gates participation in medium-value tickets, community mediation panels, and DAO voting in models where 1P1V is unavailable.

---

## Mechanism (Normative Summary)

A Proximity Proof is a **Kind 30026** Nostr event. Two proofs from **two different verifier pubkeys**, each already at Level 2 or higher, are required to elevate a subject to Level 2.

Minimum required tags (full schema belongs in [nostr-events.md § Kind 30026](../core/nostr-events.md) once this phase ships):

| Tag | Value | Purpose |
|-----|-------|---------|
| `subject` | subject pubkey (hex) | Who is being attested |
| `verifier_level` | `2` or `3` | Claimed verifier level at time of signing |
| `encounter_ts` | Unix timestamp | When the in-person encounter took place |
| `location_hint` | optional coarse geohash or city | Sybil-pair detection; never precise coordinates |
| `d` | addressable tag `<subject>:<encounter_ts>` | Deduplicates replays |

The subject's identity handler aggregates valid Kind 30026 events; upon observing two from distinct verifiers within a rolling 180-day window, it issues a Kind 30021 (Credential) event elevating the subject to Level 2.

---

## Anti-Sybil Constraints

- A single verifier pubkey may issue at most **1 proximity proof per subject per 90 days**.
- Two verifier pubkeys that co-attest more than **5 subjects together in 30 days** are flagged for DAO review (possible Sybil ring).
- A verifier demoted below Level 2 invalidates their outstanding proximity proofs **prospectively** (existing Level 2 credentials are not retroactively revoked).

---

## Why This Is Phase 3

Phase 1 (V1) ships Level 0 (self-declared pubkey) and Level 1 (LNURL-Auth attested). Level 2 requires:

1. A stable Level 2+ verifier population (chicken-and-egg — bootstrapped by Foundation DAO members).
2. Nostr infrastructure maturity around Kind 30026 relay acceptance and replay protection.
3. DAO governance to handle verifier misconduct reports.

None of these are V1 blockers, so the mechanism is specified at the level needed for third-party implementers to plan, but the handler, endpoints, and UI are deferred.

---

## References

- [Identity Protocol](./identity-protocol.md) — Level 0–3 overview
- [Nostr Events § Kind 30026](../core/nostr-events.md) — event schema (placeholder until this phase ships)
- [Nostr Events § Kind 30021 — Credential Event](../core/nostr-events.md) — credential issuance
- [DAO Codex](../governance/dao-codex.md) — sanction process for misbehaving verifiers
- [ADR-010 — Identity Level Model](../../decisions/010-identity-level-model.md)

---

*Part of the Colabonate Protocol Specification · [← Back to Protocols](../README.md)*
