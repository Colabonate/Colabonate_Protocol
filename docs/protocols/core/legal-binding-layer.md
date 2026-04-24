# Legal Binding Layer – Opt-in DAO+Codex Binding

**Normativity:** Descriptive

**Version:** 1.0.0-draft
**Date:** 2026-03-22
**Status:** [PHASE 4]

---

## Overview

The Legal Binding Layer is an **opt-in mechanism** that allows participants to bind a specific transaction to a DAO's Codex for enhanced accountability and dispute resolution.

**The base Colabonate Protocol operates without any legal binding.** This layer is strictly optional and is designed for participants who want:
- Community-specific governance of their interactions
- A private arbitration framework for higher-value transactions
- Reference to a defined set of rules in case of dispute

---

## Protocol Philosophy

The Colabonate Protocol is a **neutral protocol infrastructure**. It does not:
- Guarantee regulatory compliance with any jurisdiction
- Replace state-court litigation
- Constitute a legal agreement between participants

The legal binding layer provides a **private arbitration framework** — similar to how parties in commercial contracts agree to arbitration clauses. The DAO Codex serves as the rule set for that private arbitration.

**Participants are responsible for determining whether a DAO+Codex binding satisfies their local legal requirements.** The protocol makes no representations about legal compliance.

---

## The Binding Mechanism

### Ticket-Level Binding

Legal binding is specified at the individual ticket level using three fields in the ticket schema:

| Field | Type | Description |
|-------|------|-------------|
| `daoId` | String (UUID) | Identifier of the DAO whose Codex governs this ticket |
| `codexHash` | String (SHA-256) | Hash of the exact Codex version at time of binding |
| `codexVersion` | String (semver) | Human-readable version of the Codex (e.g. "1.2") |

These fields appear in the ticket creation event (Nostr Kind 30018):

```json
{
  "kind": 30018,
  "tags": [
    ["d", "<ticket-id>"],
    ["offer", "<offer-id>"],
    ["buyer", "<buyer-pubkey>"],
    ["seller", "<seller-pubkey>"],
    ["dao_id", "<dao-id>"],
    ["codex_hash", "<sha256-of-codex-at-binding>"],
    ["codex_version", "1.2"]
  ]
}
```

### Codex Version Freeze

When a ticket is created with a DAO binding, the `codexHash` captures the **exact version of the Codex at that moment**.

**Subsequent Codex amendments do not affect bound tickets.** If the DAO amends its Codex from v1.2 to v1.3 after a ticket is created:
- The ticket created with v1.2 hash continues to operate under v1.2 rules
- Only tickets created after the amendment use v1.3

This immutability protects participants from retroactive rule changes during an active transaction.

---

## What Binding Means for Each Party

### For Buyers and Sellers

| Aspect | Unbound (Base Protocol) | Bound to DAO Codex |
|--------|------------------------|-------------------|
| Dispute Level 1 | Base protocol defaults | DAO's Level 1 procedure |
| Dispute Level 2 | Foundation mediator pool | DAO's mediator pool (if defined) |
| Dispute Level 3 | Foundation DAO arbitration | DAO's arbitrators (if defined) |
| Arbitration rules | Base protocol rules | DAO Codex rules (additive only) |
| Fee structure | Base protocol fees | DAO fee structure (may differ) |
| Community accountability | None | DAO membership accountability |

### For DAO Members

When a transaction is bound to their DAO's Codex:
- DAO members may be called to serve as mediators or arbitrators
- Verdicts are published on Nostr and affect the DAO's community reputation
- The DAO treasury may receive a portion of dispute resolution fees

---

## Implications of Binding: Legal Considerations

### What a Codex Binding IS

A Codex binding creates a **private arbitration agreement** between the two parties:
- Both parties agree to resolve disputes according to the DAO's Codex rules
- Decisions by DAO mediators and arbitrators are binding within the Colabonate protocol (escrow execution follows the verdict)
- The DAO acts as a private dispute resolution body, similar to commercial arbitration services

### What a Codex Binding IS NOT

- **Not a state-court-enforceable contract** — Nostr-published Codex bindings have no legal standing in most jurisdictions without additional legal agreements between parties
- **Not KYC or regulatory compliance** — binding to a DAO Codex does not constitute AML/KYC compliance
- **Not a guarantee of legal enforcement** — if a party violates a verdict, the protocol can sanction their Colabonate account but cannot enforce physical-world compliance

### Regulatory Compliance Note

**The protocol explicitly does not guarantee regulatory compliance.** Participants who require regulatory compliance (e.g. for licensed financial services, consumer protection, data protection) are responsible for:
1. Consulting legal advisors in their jurisdiction
2. Determining whether a DAO Codex binding satisfies applicable requirements
3. Adding any required supplementary legal agreements outside of the protocol

Some community DAOs may be structured to satisfy specific regulatory requirements (e.g. a DAO with a choice-of-law clause pointing to Swiss arbitration law). This is a community DAO design choice, not a protocol-level guarantee.

---

## Cross-Jurisdiction Considerations

Community DAOs can optionally include a **choice-of-law clause** in their Codex:

```markdown
## Choice of Law (Optional)
Disputes arising from transactions bound to this Codex shall be resolved
in accordance with the laws of [jurisdiction], with [arbitration body] as
the seat of arbitration. This clause supplements the Colabonate Protocol's
dispute resolution procedure but does not replace it.
```

This clause has no technical effect within the Colabonate protocol — it is a contractual declaration for legal purposes only. The protocol processes all disputes equally regardless of choice-of-law clauses.

---

## Foundation DAO Binding (Special Case)

Transactions bound to the **Foundation DAO Codex** (`daoId: "colabonate-foundation"`) benefit from:
- The established Foundation governance structure
- Access to the Foundation's full mediator and arbitrator pool
- Published, auditable governance history on Nostr
- Higher public accountability (Foundation DAO operates transparently)

Foundation DAO binding is the default recommendation for high-value transactions where community DAO infrastructure is not available or where cross-community arbitration is needed.

---

## Interaction with Base Protocol Guarantees

The DAO Codex may **add requirements** but cannot **remove base protocol guarantees**:

| Base Protocol Guarantee | Modifiable by DAO Codex? |
|------------------------|--------------------------|
| Escrow mechanism (Lightning Hold Invoice) | No — escrow is always protocol-layer |
| Ticket status machine (PENDING → COMPLETED) | No — status flow is fixed |
| Nostr event immutability | No — published events are permanent |
| Private key custody policy | No — protocol never holds keys |
| Bitcoin-only payments (sats) | No — no altcoin payments permitted |
| Level 1 dispute right (self-service) | No — always available |
| DAO dispute fee minimum | Yes — DAO can set higher fees |
| Dispute resolution timeline | Yes — DAO can set shorter windows |
| Additional membership criteria | Yes — DAO can add requirements |

---

## Binding Lifecycle

```
Ticket Created (unbound)
     │
     ▼
Binding Fields Added (daoId, codexHash, codexVersion)
     │
     ▼
Ticket Active — Codex version frozen at creation hash
     │
     ├── [Dispute occurs] → DAO Codex governs dispute
     │        │
     │        ▼
     │   Verdict published (Nostr Kind 30019)
     │        │
     │        ▼
     │   Escrow executed per verdict
     │
     ├── [Completion] → Ticket COMPLETED, Codex binding ends
     │
     └── [Cancellation] → Ticket CANCELLED, Codex binding ends

Codex Amendment (DAO-internal) → only affects tickets created AFTER amendment
```

---

## References

- [docs/protocols/governance/dao-creation-protocol.md](../governance/dao-creation-protocol.md) — how DAOs and Codexes are created
- [docs/protocols/governance/dao-codex.md](../governance/dao-codex.md) — Foundation DAO Codex
- [docs/protocols/core/escrow-protocol.md](./escrow-protocol.md) — escrow execution following verdicts
- [docs/protocols/workflows/dispute-protocol.md](../workflows/dispute-protocol.md) — dispute resolution procedure
- [docs/protocols/core/ticket-system.md](./ticket-system.md) — ticket schema including daoId field
- [docs/protocols/core/nostr-events.md](./nostr-events.md) — Kind 30018 (ticket creation with binding)

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
