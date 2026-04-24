# Arbitration Council — Operational Specification

**Status:** `[ACTIVE]` — Multi-Member Operation (ADR-126 M2)
**Applies to:** Foundation DAO Level 3 dispute resolution
**Supersedes:** dispute-protocol.md Section on Level 3 (bootstrap mode only)

---

## 1. Mandate

The Colabonate Arbitration Council ("the Council") is the operative body of the Colabonate Foundation DAO that issues binding verdicts in Level 3 disputes. Its mandate derives from the DAO Codex Chapter II.1.

The Council's verdicts trigger Lightning escrow state transitions via the escrow router (`apps/server/routes/escrow.ts`), which maps verdict outcomes to escrow actions: `RELEASE` → funds to seller, `CANCEL` → funds to buyer.

---

## 2. Member Lifecycle (M2)

### 2.1 Membership States

| State | Description |
|-------|-------------|
| `PENDING` | Invited by admin; must sign Kind 30021 credential to activate |
| `ACTIVE` | Full arbitrator privileges; can issue verdicts |
| `REVOKED` | Cannot issue verdicts; prior verdicts remain in audit trail |

### 2.2 Invite Flow

1. Admin calls `POST /api/dao/foundation/members` with target pubkey + role
2. Invitee receives accept link: `/council/invite/accept?pubkey=<pubkey>`
3. Invitee signs Kind 30021 credential via NIP-07
4. Server verifies signature and activates member

### 2.3 Revocation

1. Admin calls `PATCH /api/dao/foundation/members/:pubkey/revoke`
2. Server marks member `REVOKED` and returns Kind 5 deletion template
3. Admin signs and publishes Kind 5 to revoke the Kind 30021 credential

---

## 3. Quorum Configuration

### 3.1 Quorum Types

| Type | Behavior | Use Case |
|------|----------|----------|
| `SINGLE` | Any single verdict resolves (M0 bootstrap default) | Fast resolution, single arbitrator |
| `MAJORITY` | >50% of active arbitrators required | Balanced governance |
| `THRESHOLD` | Exactly N verdicts required | Predictable timing |

### 3.2 Config Snapshot

When a dispute is escalated to `COUNCIL_QUEUE`, the current `quorumType` and `thresholdN` are snapshotted into the dispute record. In-flight disputes continue under their snapshot config even if the DAO config changes later.

---

## 4. Verdict Procedure

### 4.1 Who Can Issue Verdicts

Only arbitrators whose:
- `DaoMember.status = 'ACTIVE'`
- `DaoMember.role ∈ {'ADMIN', 'ARBITRATOR'}`
- Have not previously voted on this dispute

### 4.2 Verdict Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET` | `/api/dao/foundation` | Public | DAO metadata + config |
| `GET` | `/api/dao/foundation/members` | Public | All members (filter by ?status=) |
| `GET` | `/api/dao/foundation/config` | Public | Quorum + timing config |
| `POST` | `/api/dao/foundation/members` | Admin (NIP-07) | Invite a member |
| `PATCH` | `/api/dao/foundation/members/:pubkey/revoke` | Admin (NIP-07) | Revoke a member |
| `POST` | `/api/dao/foundation/members/:pubkey/accept` | Invitee-signed | Activate membership |
| `PATCH` | `/api/dao/foundation/config` | Admin (NIP-07) | Update quorum config |
| `POST` | `/api/disputes/:id/escalate` | Disputant | OPEN → COUNCIL_QUEUE |
| `POST` | `/api/disputes/:id/withdraw` | Opener | → WITHDRAWN |
| `POST` | `/api/disputes/:id/verdict` | Arbitrator | Submit verdict |

### 4.3 Quorum Aggregation

1. Verdict is persisted immediately upon submission
2. Quorum is evaluated after each verdict
3. If quorum met: dispute resolves, escrow transitions
4. If tie (equal RELEASE/CANCEL): status stays `IN_REVIEW`, admin tie-break required

### 4.4 Verdict Outcomes

| Outcome | Escrow Action | Description |
|---------|--------------|-------------|
| `RELEASE` | DISPUTED → RELEASED | Funds to seller |
| `CANCEL` | DISPUTED → CANCELLED | Funds to buyer |
| `SPLIT` | Deferred | Invoice split not yet implemented |

### 4.5 Dissent

Dissent is a feature, not a leak. All submitted verdicts are public and immutable. The audit trail retains dissenting votes even when the majority outcome resolves the dispute.

---

## 5. Enforcement Hook

Verdicts are connected to the escrow state machine through the `verifyActor(ARBITRATOR, ...)` function in `apps/server/routes/escrow.ts`. For M2:

1. Request includes `callerPubkey` in the request body
2. Server loads `DaoMember` and asserts `status = 'ACTIVE'`, role ∈ {'ADMIN', 'ARBITRATOR'}
3. Server checks no prior verdict exists for this caller on this dispute
4. On success: verdict is persisted; quorum check triggers escrow transition
5. On failure: HTTP 403

---

## 6. Dispute Status State Machine

```
OPEN
  └── (cooling-off elapsed + disputant calls escalate)
      → COUNCIL_QUEUE

COUNCIL_QUEUE
  └── (first verdict submitted)
      → IN_REVIEW

IN_REVIEW
  └── (quorum met + majority outcome)
      → RESOLVED
  └── (tie → equal RELEASE/CANCEL votes)
      → IN_REVIEW (admin tie-break required)

RESOLVED
  └── (terminal state)

WITHDRAWN
  └── (opener withdraws before any verdict)
      → terminal state
```

---

## 7. Nostr Events

### 7.1 Kind 30022 — `arbitration_verdict` sub-type

Published by the arbitrator (client-side signed) when issuing a verdict.

```json
{
  "kind": 30022,
  "pubkey": "<arbitrator-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "<verdict-id>"],
    ["sub_type", "arbitration_verdict"],
    ["proposal", "<dispute-kind-30020-event-id>"],
    ["vote", "yes"],
    ["voting_model", "1p1v"]
  ],
  "content": "{\"sub_type\":\"arbitration_verdict\",\"dao_id\":\"colabonate-foundation\",\"dispute_id\":\"<kind-30020-id>\",\"outcome\":\"RELEASE\",\"reasoning\":\"<text>\"}"
}
```

### 7.2 Kind 30021 — Arbitrator Credential

Issued by the member (self-issued after invite acceptance) to establish arbitration authority.

```json
{
  "kind": 30021,
  "pubkey": "<member-pubkey>",
  "created_at": 1234567890,
  "tags": [
    ["d", "colabonate-foundation-arbitrator-<pubkey>"],
    ["credential_type", "arbitrator"],
    ["dao_id", "colabonate-foundation"],
    ["role", "ARBITRATOR"],
    ["issued_by", "<admin-pubkey>"]
  ],
  "content": "{\"credential_type\":\"arbitrator\",\"dao_id\":\"colabonate-foundation\",\"role\":\"ARBITRATOR\",\"issued_by\":\"<pubkey>\"}"
}
```

### 7.3 Kind 5 — Credential Revocation

Published by admin when revoking a member. References the Kind 30021 event ID to revoke.

```json
{
  "kind": 5,
  "pubkey": "<admin-pubkey>",
  "created_at": 1234567890,
  "tags": [["e", "<kind-30021-event-id>"]],
  "content": "Credential revocation due to membership revoke"
}
```

---

## 8. Audit Trail

The `/council/audit` page (public) lists all resolved disputes with:
- Dispute ID + ticket reference
- Final outcome (RELEASE/CANCEL)
- Total verdict count
- Dissent count (verdicts that differed from outcome)

Drilling into a resolved dispute shows each individual verdict:
- Arbitrator pubkey
- Outcome (RELEASE/CANCEL)
- Reasoning (if provided)
- Timestamp
- Kind 30022 event ID

---

## 9. References

- [DAO Codex](./dao-codex.md) — Foundation constitution
- [Dispute Protocol](../workflows/dispute-protocol.md) — 3-level dispute system
- [Nostr Events](../core/nostr-events.md) — Kind 30020, 30021, 30022, 30023 schemas
- [Escrow Protocol](../core/escrow-protocol.md) — Verdict → settlement mapping
- [ADR-125](../../decisions/125-arbitration-council-bootstrap.md) — M0+M1 bootstrap record
- [ADR-126](../../decisions/126-arbitration-council-multi-member.md) — M2 multi-member implementation
- [PRD: M2 Multi-Member Operation](../../prd/ARBITRATION-COUNCIL-M2-PRD.md) — Product specification

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
