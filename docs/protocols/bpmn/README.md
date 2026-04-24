# BPMN Process Documentation

Business Process Model and Notation (BPMN 2.0) diagrams for Colabonate workflows.

**Standard:** BPMN 2.0 (OMG Open Standard) – royalty-free
**Tooling:** [bpmn.io](https://bpmn.io) / [Camunda Modeler](https://camunda.com/download/modeler/) / draw.io
**Reference:** ADR-059 (D3 – BPMN for Process Documentation)

---

> **⚠ May be out of date**
>
> These diagrams are **derived artifacts** from the protocol specs under `workflows/`, `core/`, and `identity/`. They have not been fully re-validated against the recent protocol separation from the reference implementation.
>
> Specs that changed since these diagrams were last synced:
>
> - `workflows/cooperation-protocol.md` — 4-step flow (ADR-114)
> - `workflows/buy-protocol.md` — escrow API paths removed, status names aligned with `escrow-protocol.md`
> - `core/ticket-system.md` — EscrowStatus vocabulary unified, extended statuses documented
> - `core/nostr-events.md` — status tags updated + Kinds 30027, 30414–30415, 30420–30422 added
>
> **High-risk diagrams:** `cooperation-flow.bpmn`, `milestone-service-flow.bpmn`
> **Medium-risk:** `order-product-flow.bpmn`, `escrow-service-flow.bpmn`
> **Low-risk:** `onboarding-flow.bpmn`
>
> Before relying on a diagram as authoritative, cross-check against the linked source protocols.

---

## Diagrams

| File | Process | Status |
|------|---------|--------|
| `order-product-flow.bpmn` | Order Product Flow (PRODUCT: auto-accepted, 3-phase escrow) | ✅ |
| `escrow-service-flow.bpmn` | Smart Order Escrow Flow (SERVICE: seller approval, 3-phase escrow, dispute) | ✅ |
| `milestone-service-flow.bpmn` | Milestone-based Service Delivery (cooperation milestone loop with escrow) | ✅ |
| `cooperation-flow.bpmn` | Cooperation / Joint Project Flow (initiation → milestones → dispute protocol) | ✅ |
| `onboarding-flow.bpmn` | Creator Onboarding Flow (LNURL-Auth, role selection, identity L0→L2) | ✅ |

### Protocol Sources

Each diagram is derived from the corresponding protocol specification:

| Diagram | Source Protocol |
|---------|----------------|
| `order-product-flow.bpmn` | `workflows/buy-protocol.md` — PRODUCT flow, `core/escrow-protocol.md` — 3-phase escrow |
| `escrow-service-flow.bpmn` | `workflows/buy-protocol.md` — SERVICE flow, `core/escrow-protocol.md` — Hold Invoice state machine |
| `milestone-service-flow.bpmn` | `workflows/cooperation-protocol.md` — 4-phase cooperation flow, `core/ticket-system.md` — Milestone Ticket |
| `cooperation-flow.bpmn` | `workflows/cooperation-protocol.md` — full cooperation + `workflows/dispute-protocol.md` — 3-level dispute |
| `onboarding-flow.bpmn` | ADR-036 (Scope: Onboarding), `core/nostr-events.md` — Kind 0/30027, `identity/identity-protocol.md` — L0–L2 |

---

## Naming Convention

`<domain>-<variant>-flow.bpmn`

Examples: `order-product-flow.bpmn`, `escrow-service-flow.bpmn`

## Related ADRs

- ADR-021: Smart Order Escrow Flow
- ADR-023: Offer defines Ticket Type and Milestones
- ADR-059: Standardized Category Protocol (introduces BPMN standard)


---

*Part of the Colabonate Protocol Specification | [docs/protocols/](../README.md)*
