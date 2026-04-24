# Role Onboarding — Colabonate Arbitration Council

**Version:** 1.0.0
**Published:** 2026-04-17
**Related Documents:** [Arbitration Rubric](./arbitration-rubric.md) · [DAO Codex](./dao-codex.md) · [Dispute Protocol](../workflows/dispute-protocol.md) · [Arbitration Council Spec](./arbitration-council.md)

---

## For Arbitrators

### Who You Are

You are a member of the Colabonate Arbitration Council, the operative arm of the Foundation DAO. Your authority derives from your Kind 30021 credential (signed and published on the Nostr relay) and your status as ACTIVE in the DAO member registry.

**Your role:** Review dispute evidence, apply the Arbitration Rubric, and issue a binding verdict (RELEASE or CANCEL). Your verdict triggers the escrow state transition — it is consequential and permanent.

**You are not:** A judge who enforces law; a mediator who negotiates settlement; an advocate for either party.

---

### Before Your First Verdict

Read the following in order before submitting your first verdict:

1. **[Arbitration Rubric v1.0.0](./arbitration-rubric.md)** — Read §1–§4 completely. You need to know the Decision Matrix and when deviation is appropriate.
2. **[DAO Codex Appendix A](./dao-codex.md#appendix-a-foundation-dao-operation-adr-125-adr-126)** — Understand the bootstrap context; you operate under M2 rules (SINGLE quorum until M3).
3. **[Dispute Protocol](../workflows/dispute-protocol.md)** — Understand the 3-level system and when a dispute is eligible for Council review.
4. **[Arbitration Council Spec](./arbitration-council.md)** — Technical operational details: credential types, event kinds, quorum evaluation.

Estimated reading time: 20–30 minutes.

---

### Workflow

```
1. Receive email notification → new dispute arrived
2. Open /council/disputes/:id  → review the evidence
3. Check the rubric link (📖 Arbitration Rubric) → apply relevant row
4. Write reasoning (optional but encouraged) → cite rubric row or deviation
5. Submit verdict (RELEASE or CANCEL) → immutable once submitted
6. Quorum evaluation runs automatically → if met, escrow transitions
```

---

### Response-Time Expectation

Resolve disputes within **48 hours** of receiving the notification email. Time-sensitive cases (e.g., hold-invoice expiry) may require faster response. The Council page shows dispute age — prioritize older cases.

If you cannot respond within 48 hours, you may withdraw your verdict submission before quorum is met (dispute must still be in COUNCIL_QUEUE or IN_REVIEW status). Once quorum is met and escrow fires, the verdict is final.

---

### Immutability Warning

Verdicts are **permanent once submitted.** You cannot edit or retract them. Before submitting:

- [ ] Have you read the evidence?
- [ ] Have you applied the rubric (or documented a deviation)?
- [ ] Is your reasoning field complete enough that another arbitrator could follow your logic?
- [ ] Are you confident the outcome matches the evidence?

If any answer is no, wait. There is no undo.

---

### Revocation

Your membership can be revoked by a DAO Admin via a Kind 5 event. If revoked:

- Your Kind 30021 credential is invalidated
- Any pending verdicts you have not yet submitted cannot be submitted under your credential
- Verdicts already submitted remain valid and on-chain (immutable)
- Your reputation score is affected by any unresolved disputes you were assigned

---

## For Admins

### Who You Are

You are a DAO Admin: you manage membership, configuration, and serve as tie-breaker in M2 (this changes in M3 when rotating assignments are introduced). You have privileges that arbitrators do not: invite members, revoke credentials, update DAO config.

---

### Before Your First Admin Action

1. Read **[DAO Codex Chapters I–IV](./dao-codex.md)** — Understand voting models, amendment procedure, and the full governance structure.
2. Read **[Arbitration Council Spec](./arbitration-council.md)** — Understand member lifecycle (INVITE → PENDING → ACTIVE → REVOKED).
3. Confirm you have your NIP-07 extension (nos2x or getalby) configured — Admin actions require signed challenges.

---

### Invite Flow

When inviting a new arbitrator:

1. Collect their pubkey (Nostr npub format, e.g., `npub1...`).
2. Send an invite out-of-band (email or Nostr DM) containing:
   - Their pubkey
   - Link to [Arbitrator Onboarding section](#for-arbitrators) (first call-to-action after they accept)
   - Expected response time (48 hours)
   - Any context about the Council's current workload
3. The invitee signs Kind 30021 with their NIP-07 extension → credential published to relay
4. You approve → status set to ACTIVE → they appear in the Council dashboard

**Invite email template (M2):**
```
Subject: Colabonate Arbitration Council — Invitation

You have been invited to serve as an arbitrator on the Colabonate Arbitration Council.

Your pubkey: <pubkey>
Accept your invitation: <accept-link>

Before your first verdict, please read the Arbitrator Onboarding guide:
<onboarding-link>

Expected response time: 48 hours from notification.

— Colabonate Foundation DAO
```

---

### Revocation (Kind 5)

To revoke a member's credential:

1. Go to the Admin panel → Members → select member.
2. Click Revoke.
3. Sign the Kind 5 event with your NIP-07 extension.
4. The revocation is published to the relay; the member's status changes to REVOKED.

Kind 5 is permanent. A revoked member cannot be reinstated under the same pubkey without a new invite and credential.

---

### Config Changes

**Quorum type** (`SINGLE` / `MAJORITY` / `THRESHOLD`): Change only when the Council membership changes materially. Document the rationale in meeting notes or an ADR.

**Cooling-off days**: Default is 7 days. Changes affect new disputes only — in-flight disputes retain the cooling-off snapshot from escalation time.

**Threshold N** (for THRESHOLD type): Set to match the number of active arbitrators at time of configuration. Reassess when membership changes.

---

### Audit Log Review

Review the Council audit trail monthly:

- `/council` page → dispute history
- `NotificationLog` table → email delivery success rates
- Check for disputes stuck in COUNCIL_QUEUE with no arbitrator action

---

## For Disputants

### Before Opening a Dispute

You must have attempted **Level 1 resolution first.** This means:

- [ ] At least 7 days of documented direct communication with the other party
- [ ] A clear description of the breach you are disputing
- [ ] Evidence that you attempted to resolve the issue (messages, timestamps)

Opening a dispute without a documented L1 attempt will be noted by the arbitrator and may reduce the weight of your claims, though it will not result in outright dismissal.

---

### What Makes a Well-Formed Dispute

A good dispute description includes:

1. **Timeline:** What happened, in chronological order
2. **Evidence:** Links to relevant Nostr events, chat logs, or screenshots with timestamps
3. **Desired outcome:** RELEASE (funds to seller) or CANCEL (refund to buyer) — you are requesting one; the arbitrator makes the final decision
4. **Ticket ID:** The ticket in question

**Evidence that carries weight:**
- Timestamped Nostr events (most reliable)
- Chat logs attributable to both parties' pubkeys
- Screenshots with metadata visible

**Evidence that carries little weight:**
- Vague claims ("they never delivered")
- One party's word against another without corroboration
- Evidence that cannot be tied to the relevant pubkeys or ticket

---

### What Happens Next

```
1. You open dispute → ticket transitions to DISPUTED
2. DAO arbitrators receive email notification
3. Arbitrator reviews evidence, applies rubric, issues verdict
4. If quorum met → escrow transitions (RELEASE or CANCEL)
5. Both parties receive email notification of verdict
6. Verdict is immutable → no contest in M2 (Phase 4 Appeal planned)
```

Timeline: Typically 24–48 hours from escalation to resolution, depending on arbitrator availability.

---

### Outcomes

| Outcome | Meaning | What Happens |
|---------|---------|-------------|
| **RELEASE** | Funds released to seller | Escrow settled; ticket COMPLETED |
| **CANCEL** | Funds refunded to buyer | Escrow cancelled; ticket CANCELLED |
| **SPLIT** | (Not yet implemented) | Deferred per Rubric §4 |

---

### Immutability

Once the arbitrator issues a verdict and quorum is met, the verdict is **final and immutable.** You cannot contest it within M2. The Phase 4 Appeal mechanism is planned but not yet implemented.

---

## Role Matrix Quick Reference

| Role | Can Do | Cannot Do |
|------|--------|-----------|
| ADMIN | Invite/revoke members, update config, tie-break (M2), issue verdicts | Self-revoke; override other admin decisions |
| ARBITRATOR | Submit verdicts, read disputes, review evidence | Invite/revoke members, change config |
| MEDIATOR | (reserved — Phase 3) | N/A |
| DISPUTANT | Open dispute, withdraw (before IN_REVIEW), receive verdict | Submit verdict, see pre-publication reasoning |

---

## Links

- [DAO Codex](./dao-codex.md)
- [Arbitration Rubric](./arbitration-rubric.md)
- [Dispute Protocol](../workflows/dispute-protocol.md)
- [Arbitration Council Spec](./arbitration-council.md)
- [ADR-125: Bootstrap](../../decisions/125-arbitration-council-bootstrap.md)

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
