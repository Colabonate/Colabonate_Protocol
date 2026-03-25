# 009 – Identity Level Model (0–3)

**Status:** accepted
**Date:** 2026-03-22
**Decider:** Deniz Yilmaz (Founder & Protocol Architect)

## Context

Colabonate is a protocol for decentralized trade and cooperation between people. Unlike traditional platforms that require centralized KYC (Know Your Customer) verification, Colabonate must balance three competing requirements:

1. **Low barrier to entry** — anyone with a Lightning wallet should be able to use the protocol immediately
2. **Sybil resistance for high-trust interactions** — governance voting, dispute mediation, and high-value trades require some assurance that participants are unique humans
3. **Privacy preservation** — the protocol should not require disclosure of government-issued identity documents to a central authority

The question: how should the protocol handle identity verification in a way that is permissionless at the base level but can provide meaningful trust guarantees for high-stakes interactions?

Several approaches were considered, ranging from no identity (fully anonymous) to mandatory KYC.

## Decision

We use a **four-level tiered identity model (Level 0–3)** where each level unlocks additional protocol capabilities.

| Level | Name | Method | Sybil Resistance |
|-------|------|--------|-----------------|
| 0 | Anonymous | LNURL-Auth | Low |
| 1 | Nostr Profile | Nostr Kind 0 + NIP-05 | Low |
| 2 | Peer Verified | 2 in-person verifiers + Proximity Proof | Medium |
| 3 | HID Verified | Humanode Biomapper ZK biometric | High |

Higher levels are opt-in and unlock additional capabilities (governance voting, mediation, DAO founding membership). No capability is permanently gated for all users — the protocol can be used productively at Level 0.

**Key design choices:**
- Level 2 uses human social proof (peer verification) rather than government documents
- Level 3 uses cryptographic biometric proof without storing or transmitting raw biometric data
- No central authority decides identity — the verification is distributed across peers and the Humanode network
- All identity credentials are published as Nostr events (soulbound, non-transferable)

## Alternatives

### 1. Binary: Verified / Unverified

Two states only: anonymous (no verification) or verified (single verification step).

**Why rejected:** A binary model forces a trade-off. If the verification is strong (e.g. Humanode biometric), the barrier to entry is high and many legitimate users who don't want biometric verification cannot participate in governance. If the verification is weak (e.g. email), it provides little Sybil resistance. The four-level model allows progressive trust without forcing all users to the highest level.

### 2. KYC via Identity Provider

Require users to submit government documents to a third-party KYC provider (e.g. Stripe Identity, Jumio).

**Why rejected:** This introduces a central authority that holds user PII, contradicts the decentralization principle, creates regulatory liability for the Foundation, and excludes users in jurisdictions where official documents are difficult to obtain. It also creates a honeypot of personal data.

### 3. Humanode-Only (Skip Peer Verification)

Use only Humanode biometric verification, skipping the peer verification step.

**Why rejected:** Humanode requires specific hardware and software support. In regions where Humanode access is limited, users would be permanently excluded from governance. The peer verification Level 2 provides meaningful Sybil resistance without hardware requirements and creates a trust web through in-person social attestation — a valuable property independent of biometrics.

### 4. Social Graph Verification (Web of Trust)

Use an existing social web-of-trust (e.g. Nostr WoT, PGP keyservers) as the verification layer.

**Why rejected:** Web-of-trust systems require existing trust networks, which creates a chicken-and-egg problem for a new protocol. They also do not provide Sybil resistance against well-coordinated actors who create multiple identities within the trust graph. The in-person encounter requirement of Level 2 is more resistant to automated Sybil attacks than web-of-trust graphs.

### 5. Three Levels (Remove Level 1)

Skip the "Nostr Profile" level and go directly from Level 0 to peer verification.

**Why rejected:** Level 1 (Nostr Profile) provides a meaningful intermediate state: a user who has published a persistent Nostr identity and optionally linked a domain via NIP-05 is more accountable than a purely ephemeral LNURL-Auth session. This level costs nothing but a one-time key generation and is a natural step for users already using Nostr.

## Consequences

### Positive

- Zero barrier to entry — Level 0 works with any Lightning wallet, no account creation needed
- Progressive trust — users choose their verification level based on their participation goals
- Privacy-preserving — Level 3 uses ZK proofs; no biometric data ever stored on any server
- Decentralized — Level 2 uses human peers; Level 3 uses Humanode's distributed network
- Community bootstrapping — verifier pool creates an incentive for Level 3 users to help Level 2 candidates
- Protocol can function with majority Level 0 users while maintaining governance integrity via Level 3 1P1V

### Negative

- Level 2 peer verification requires real-world logistics (finding verifiers, scheduling meetings) — this can be slow in low-density areas
- Level 3 depends on Humanode as an external dependency — if Humanode changes or fails, Level 3 verification is affected
- Four levels add complexity to client implementations — clients must handle each level differently
- Users who lose their private key lose their reputation (identity is pubkey-bound) — this is a fundamental property of the model but creates UX risk

## Affected Documents

- `docs/protocols/identity/identity-protocol.md` — full specification
- `docs/protocols/core/nostr-events.md` — Kind 30021, 30023, 30026
- `docs/protocols/core/reputation-protocol.md` — identity level multipliers
- `docs/protocols/governance/dao-codex.md` — 1P1V governance requirement
- `docs/protocols/core/ticket-system.md` — Verification Ticket type

## References

- [Humanode Biomapper](https://humanode.io)
- [NIP-05: DNS Identity Verification](https://github.com/nostr-protocol/nostr/blob/master/05.md)
- [LNURL-Auth LUD-04](https://github.com/lnurl/luds/blob/luds/04.md)
- [docs/protocols/identity/identity-protocol.md](../protocols/identity/identity-protocol.md)

---

*Decision made by Deniz Yilmaz (Founder) | 2026-03-22*
