# Arbitration Rubric — Colabonate Foundation DAO

**Version:** 1.0.0
**Published:** 2026-04-17
**Hash Algorithm:** SHA-256
**Supersedes:** none
**Status:** ADVISORY — Arbitrators retain discretion; deviation must be justified in verdict reasoning

---

## 1. Purpose & Advisory Status

This rubric is the decision framework for Colabonate Arbitration Council arbitrators resolving Level 3 disputes. It makes verdicts consistent, defensible, and auditable.

**Advisory only.** The rubric does not bind arbitrators mechanically. Arbitrators must evaluate evidence on its merits and may deviate from a row when the specifics of a case warrant it. In such cases, the verdict reasoning field must name the deviation explicitly (e.g., "Deviation from Rubric §2, row 7 — evidence asymmetry was not present because Seller provided timestamped logs").

The rubric is versioned and hash-addressed via the DAO Codex Appendix A reference. Changes require an ADR and a version bump.

---

## 2. Decision Matrix

| # | Case | Recommended Outcome | Primary Reasoning |
|---|------|---------------------|-------------------|
| 2.1 | Delivery confirmed + acceptance documented | **RELEASE** | Contract fulfilled per Ticket terms; both parties consented |
| 2.2 | Non-delivery past deadline + no seller response | **CANCEL** | Contract breach; buyer protected by default |
| 2.3 | Non-delivery past deadline + seller shows delivery proof | **RELEASE** | Evidence supersedes claim; delivery documented |
| 2.4 | Quality dispute + seller offers remedy | **RELEASE with note** | L1 resolution attempt succeeded; escalate only if remedy fails within stated timeframe |
| 2.5 | Quality dispute + seller refuses remedy | **CANCEL** | Buyer's reasonable rejection upheld; seller breached implied quality obligation |
| 2.6 | Partial delivery | **RELEASE (bootstrap; SPLIT deferred)** | Note in reasoning that partial delivery acknowledged; track case for SPLIT prioritization when SPLIT is implemented |
| 2.7 | Evidence asymmetry — one party silent after reasonable attempts | **Rule for responding party** | *Audiatur et altera pars* — silence without explanation forfeits the argument; responding party benefits |
| 2.8 | Spurious dispute — no prima facie breach | **RELEASE** | Disputant bears burden of proof; no breach demonstrated |
| 2.9 | Both parties document mutual breach | **Admin tie-breaker (M2) or balance-of-harm (M1)** | Document reasoning explicitly; weight harm proportionally |

---

## 3. Evidence Evaluation Principles

**3.1 Evidence Weight Hierarchy**

| Rank | Evidence Type | Notes |
|------|-------------|-------|
| 1 | Timestamped Nostr events | Signed, relay-published, immutable |
| 2 | Chat/message logs with timestamps | Must be attributable to the relevant pubkeys |
| 3 | Screenshots | Acceptable; cross-referenced where possible |
| 4 | Unattributed claims | Minimal weight; require corroboration |

**3.2 Communication First**

Disputes filed without a documented L1 cooling-off attempt (7 days of direct communication) are disfavored. The arbitrator should note in reasoning if L1 was skipped or trivially bypassed, and may reduce the weight of the disputant's claims accordingly without outright dismissing them.

**3.3 Proportional Response**

Minor delays are not breaches. A deadline missed by less than 24 hours without communication is a minor issue. A deadline missed by more than 48 hours with no response from the seller constitutes a material breach. The arbitrator must evaluate proportionality before recommending a verdict.

**3.4 Transparency**

Every verdict must name which rubric row applies, or explicitly justify deviation. A verdict that says only "RELEASE — buyer not entitled to refund" without referencing evidence or rubric rows is harder to audit and should be avoided. The reasoning field exists precisely for this.

**3.5 Immutability**

Once submitted, verdicts cannot be edited or retracted. Errors are corrected through the Phase 4 Appeal mechanism. This is why reasoning matters — it is the public record of the arbitrator's judgment and must stand up to scrutiny.

---

## 4. SPLIT Handling (Deferred)

The SPLIT verdict outcome is not yet implemented. When both parties present credible but conflicting claims and neither is clearly in the right, arbitrators should:

- Issue **RELEASE** with a note in the reasoning field: "SPLIT deferred per rubric v1.0.0; partial delivery acknowledged; funds released to seller with notation for future SPLIT recovery if protocol supports it."
- Record the dispute as a candidate for SPLIT reassessment in a future protocol version.

Do NOT hold a dispute in limbo waiting for SPLIT. Issue a binary verdict and note it.

---

## 5. Deviation Procedure

When a case falls outside the matrix or the evidence strongly suggests a different outcome than the matrix recommends:

1. State which rubric row most closely applies (if any).
2. State the reason the deviation is warranted (specific evidence, context).
3. Reference "Deviation from Rubric §X" explicitly in the reasoning field.

Deviating verdicts are valid and will not be overridden. The purpose of recording the deviation is auditability, not approval.

---

## 6. Amendment Procedure

Changes to this rubric require:

1. An ADR documenting the proposed change and rationale
2. A version bump (e.g., 1.0.0 → 1.1.0 for minor edits, 2.0.0 for breaking changes)
3. A new SHA-256 hash computed over the updated UTF-8 bytes
4. Update to DAO Codex Appendix A.3 with the new hash
5. Announced on the Colabonate public relay with the new version tag

Amendment is not retroactive. In-flight disputes continue under the rubric version active at the time of escalation.

---

## 7. References

- [Colabonate DAO Codex](./dao-codex.md)
- [Dispute Protocol](../workflows/dispute-protocol.md)
- [Arbitration Council Spec](./arbitration-council.md)
- [Nostr Events Reference](../core/nostr-events.md)
- [ADR-125: Arbitration Council Bootstrap](../../decisions/125-arbitration-council-bootstrap.md)

---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
