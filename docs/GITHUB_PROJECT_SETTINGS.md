# GitHub Project Settings

### Project name
```
Colabonate Protocol — Spec Development
```

### Short description
```
Open protocol specification for decentralized trade and cooperation via Bitcoin Lightning. Tracks spec drafts, open design questions, and milestone progress toward v1.0.0.
```

### README (Project-level, shown on the project overview page)
```
## Colabonate Protocol — Spec Development

This project tracks the development of the [Colabonate Protocol](https://github.com/Colabonate/colabonate-protocol) specification from `v0.1.0-draft` to `v1.0.0` — the first stable, implementer-ready open standard.

### How this board works

| Column | Meaning |
|--------|---------|
| **Backlog** | Planned work not yet started |
| **Open Question** | Unresolved design decisions — blocking progress |
| **In Spec** | Actively being drafted or revised |
| **In Review** | PR open — awaiting community feedback |
| **Done** | Closed and merged |

### Key documents
- [SPECIFICATION_STATUS.md](https://github.com/Colabonate/colabonate-protocol/blob/main/SPECIFICATION_STATUS.md) — status of every spec document
- [README.md](https://github.com/Colabonate/colabonate-protocol/blob/main/README.md) — milestone roadmap
- [CHANGELOG.md](https://github.com/Colabonate/colabonate-protocol/blob/main/CHANGELOG.md) — version history

### Website
[colabonate.com](https://colabonate.com)
```

### Visibility
```
Public
```

---

## Settings → Manage access

**Who can view:** Public (everyone)
**Who can edit:** Only repository collaborators with write access

---

## Board — Columns (Board View)

Create these columns **in this order** (drag to reorder after creation):

### Column 1
```
Backlog
```

### Column 2
```
Open Question
```

### Column 3
```
In Spec
```

### Column 4
```
In Review
```

### Column 5
```
Done
```

---

## Board — Automation Rules

Set these up under **Workflows** in the project settings.

### Rule 1
```
Trigger:  Item added to project
Action:   Set status → Backlog
```
*(Default rule — moves all newly added items to Backlog)*

### Rule 2
```
Trigger:  Item reopened
Action:   Set status → Backlog
```

### Rule 3
```
Trigger:  Pull request opened
Action:   Set status → In Review
```

### Rule 4
```
Trigger:  Pull request merged
Action:   Set status → Done
```

### Rule 5
```
Trigger:  Issue closed
Action:   Set status → Done
```

> **Note:** Label-based automation (open-question → column "Open Question") requires GitHub Actions, not built-in Project automation. See the GitHub Actions workflow at the bottom of this document.

---

## Views

Create these additional views alongside the default Board view:

### View 1 — By Milestone (Table)
- **Type:** Table
- **Name:** `By Milestone`
- **Group by:** Milestone
- **Fields shown:** Title, Status, Labels, Assignees, Milestone
- **Sort:** Milestone ascending

### View 2 — Open Questions (Board filtered)
- **Type:** Board
- **Name:** `Open Questions`
- **Filter:** `label:open-question`
- **Fields shown:** Title, Labels, Milestone

### View 3 — Spec Candidates (Table)
- **Type:** Table
- **Name:** `Spec Candidates`
- **Filter:** `label:spec-promotion`
- **Fields shown:** Title, Status, Milestone, Assignees

---

## Custom Fields (optional, add after basic setup)

These fields add structured metadata to issues on the board:

### Field 1
- **Name:** `Spec Document`
- **Type:** Text
- **Purpose:** Link to the affected spec file (e.g. `docs/protocols/core/escrow-protocol.md`)

### Field 2
- **Name:** `Blocking`
- **Type:** Single select
- **Options:** `v0.2.0` · `v0.3.0` · `v0.4.0` · `v1.0.0` · `post-v1.0`
- **Purpose:** Which milestone is this blocking?

### Field 3
- **Name:** `Area`
- **Type:** Single select
- **Options:** `core` · `identity` · `governance` · `workflows` · `payments` · `meta`

---

## GitHub Actions — Label-based Column Automation

Create this file in the repository to auto-move issues to the correct column based on labels.

**File:** `.github/workflows/project-automation.yml`

```yaml
name: Project Board Automation

on:
  issues:
    types: [labeled]
  pull_request:
    types: [labeled, opened, closed]

jobs:
  move-to-open-question:
    if: github.event.label.name == 'open-question'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/add-to-project@v0.5.0
        with:
          project-url: https://github.com/users/Colabonate/projects/1
          github-token: ${{ secrets.ADD_TO_PROJECT_PAT }}

  move-to-in-spec:
    if: github.event.label.name == 'spec-proposal'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/add-to-project@v0.5.0
        with:
          project-url: https://github.com/users/Colabonate/projects/1
          github-token: ${{ secrets.ADD_TO_PROJECT_PAT }}
```

> **Required secret:** Add `ADD_TO_PROJECT_PAT` (a Personal Access Token with `project` scope) to the repository secrets at:
> `https://github.com/Colabonate/colabonate-protocol/settings/secrets/actions`

---

## Labels — Create in Repository

Go to: **https://github.com/Colabonate/colabonate-protocol/labels**

Delete defaults that don't apply (`bug`, `enhancement`, `help wanted`, `invalid`), then create:

### Type Labels

| Name | Color | Description |
|------|-------|-------------|
| `spec-proposal` | `#0075ca` | Proposed addition or change to a specification |
| `open-question` | `#e4e669` | Unresolved design question blocking progress |
| `needs-research` | `#c5def5` | Requires external research or expert input |
| `editorial` | `#d4c5f9` | Typo, clarity, or formatting — no content issue |
| `adr-needed` | `#e99695` | Decision large enough to warrant an ADR |
| `spec-promotion` | `#0e8a16` | Candidate for promotion from docs/ to spec/ |
| `good-first-issue` | `#7057ff` | Well-defined, limited scope — good entry point |
| `question` | `#cc317c` | General question about the protocol |
| `duplicate` | `#cfd3d7` | Already captured as another issue |
| `wontfix` | `#cfd3d7` | Deliberate decision not to resolve |

### Area Labels

| Name | Color | Description |
|------|-------|-------------|
| `area: core` | `#f9d0c4` | Core specs: nostr-events, escrow, tickets |
| `area: identity` | `#f9d0c4` | Identity and HID protocol |
| `area: governance` | `#f9d0c4` | DAO Codex, economics, COLA token |
| `area: workflows` | `#f9d0c4` | Buy, sell, cooperation, dispute flows |
| `area: payments` | `#f9d0c4` | Payment architecture, Lightning, RSK |

### Milestone Labels

| Name | Color | Description |
|------|-------|-------------|
| `milestone: v0.2` | `#bfd4f2` | Must be resolved for v0.2.0 |
| `milestone: v1.0` | `#bfd4f2` | Must be resolved for v1.0.0 |
| `milestone: post-v1` | `#eeeeee` | Deliberately deferred to after v1.0 |

---

## Milestones — Create in Repository

Go to: **https://github.com/Colabonate/colabonate-protocol/milestones/new**

### Milestone 1

**Title:**
```
v0.1.0-draft
```
**Description:**
```
Public first draft — all core specs exist as readable drafts. Closed at initial publication.
```
**Due date:** `2026-03-22`
**Status:** Close immediately after creation ✓

---

### Milestone 2

**Title:**
```
v0.2.0
```
**Description:**
```
Stable Schemas — implementers can build Phase 1 clients.

Criteria:
- Nostr Event Schemas 30017–30026 frozen (no breaking tag changes)
- Escrow state machine finalized
- Open Question #1 (Stablecoin Escrow Denomination) closed or formally deferred
- vision.md and roles.md promoted to spec/
- First community review round completed
- GitHub Project and Issue Labels set up
```
**Due date:** *(leave blank — criteria-based, not date-based)*

---

### Milestone 3

**Title:**
```
v0.3.0
```
**Description:**
```
Identity & Reputation Complete — Levels 0–3 fully specified, COL-Points formula finalized.

Criteria:
- proximity-proof.md created as draft
- Humanode integration API documented
- COL-Points cap defined or formalized as open question
- identity-protocol.md promoted to spec/
- reputation-protocol.md promoted to spec/
- cooperation-protocol.md draft finalized
```
**Due date:** *(leave blank)*

---

### Milestone 4

**Title:**
```
v0.4.0
```
**Description:**
```
Governance Layer Complete — DAO Codex ratified, Economic Protocol finalized, Dispute Protocol complete.

Criteria:
- dao-codex.md community-reviewed and promoted to spec/
- economic-protocol.md community-reviewed
- Open Question #2 (Codex Fork RSK Contract Standard) closed
- Open Question #3 (HID Linkage Threshold) closed
- dispute-protocol.md Level 3 arbitration fully specified
- ADR 013 and ADR 014 created
```
**Due date:** *(leave blank)*

---

### Milestone 5

**Title:**
```
v1.0.0
```
**Description:**
```
Final Standard — all core documents in spec/, all open questions closed, NIP registration submitted.

Criteria:
- All documents in SPECIFICATION_STATUS.md either Stable (in spec/) or formally deferred
- security-model.md and protocol-versioning.md created and in spec/
- NIP registration for Kind 30017 submitted
- No open open-question issues without a decision
- SPECIFICATION_STATUS.md shows 0 Planned documents for Core / Identity / Workflows
```
**Due date:** *(leave blank)*

---

## First Issues to Open (immediately after setup)

Go to: **https://github.com/Colabonate/colabonate-protocol/issues/new**

### Issue 1

**Title:**
```
open-question: Stablecoin Escrow Denomination — exchange rate risk handling
```
**Body:**
```
**Affected document:** docs/protocols/core/escrow-protocol.md, docs/protocols/core/payment-architecture.md

**Question:**
When Spark Stablecoins (USD/EUR) are used for payment, how is the exchange rate risk handled in escrow?

**Context:**
The escrow holds funds in sats (Lightning). If a buyer pays in a stablecoin equivalent, and the BTC/USD rate moves during the escrow period, who absorbs the difference?

**Options:**
1. Escrow amount is always fixed in sats at ticket creation time — stablecoin amount shown is informational only
2. Escrow amount is fixed in fiat (USD/EUR) — sats amount recalculated at settlement
3. Both parties choose denomination at ticket creation — locked for the ticket lifetime

**Blocking:** Yes — blocks escrow-protocol.md promotion to spec/ (v0.2.0)

**Reference:** docs/protocols/core/payment-architecture.md — Open Issue #1
```
**Labels:** `open-question` · `area: payments` · `area: core`
**Milestone:** `v0.2.0`

---

### Issue 2

**Title:**
```
open-question: Codex Fork RSK contract interface definition
```
**Body:**
```
**Affected document:** docs/protocols/core/payment-architecture.md

**Question:**
What is the minimum interface a Codex Fork RSK smart contract must implement to be recognized by the Colabonate protocol?

**Context:**
Codex Forks are RSK-based economic sub-units created by HID-verified users (Whitepaper v6, Section 3.2). The protocol references them but no contract standard exists yet.

**Options:**
1. Define a minimal RRC-20 extension interface (functions: createFork, linkCodex, transferGovernance)
2. Define it as a separate ADR and dedicated spec document (Phase 4)
3. Leave as implementation-defined until Phase 4 — document as explicitly out of scope for v1.0

**Blocking:** Yes — blocks payment-architecture.md promotion to spec/ (v0.4.0)

**Reference:** docs/protocols/core/payment-architecture.md — Open Issue #2
```
**Labels:** `open-question` · `area: governance` · `area: payments`
**Milestone:** `v0.4.0`

---

### Issue 3

**Title:**
```
open-question: HID Linkage enforcement threshold for high-value tickets
```
**Body:**
```
**Affected document:** docs/protocols/core/payment-architecture.md

**Question:**
At what ticket value (in sats) does HID Linkage become required rather than optional?

**Context:**
Per ADR 012, HID Linkage is optional in Phase 1–2 and required for governance/mediator actions in Phase 3. For high-value marketplace tickets, enforcement is described as a DAO governance decision. That decision has not yet been made.

**Options:**
1. Threshold set by DAO vote at Phase 4 launch (no fixed value in protocol)
2. Protocol defines a default threshold (e.g. 1,000,000 sats / ~$1000) that DAO can override
3. No threshold — HID Linkage always voluntary for marketplace tickets, mandatory only for governance

**Blocking:** Yes — blocks payment-architecture.md promotion to spec/ (v0.4.0)

**Reference:** docs/protocols/core/payment-architecture.md — Open Issue #3
```
**Labels:** `open-question` · `area: identity` · `area: payments`
**Milestone:** `v0.4.0`

---

### Issue 4

**Title:**
```
spec-promotion: vision.md → spec/
```
**Body:**
```
**Document:** docs/protocols/core/vision.md
**Target:** spec/vision.md
**Milestone:** v0.2.0

**Promotion checklist:**
- [ ] No open blocking issues for this document
- [ ] Content is complete (no [PLANNED] sections)
- [ ] 14-day community review period completed
- [ ] No unresolved objections
- [ ] SPECIFICATION_STATUS.md updated to Stable
- [ ] Banner added to docs/protocols/core/vision.md

**Reference:** SPECIFICATION_STATUS.md — first spec/ candidate
```
**Labels:** `spec-promotion` · `area: core`
**Milestone:** `v0.2.0`

---

### Issue 5

**Title:**
```
spec-promotion: roles.md → spec/
```
**Body:**
```
**Document:** docs/protocols/core/roles.md
**Target:** spec/roles.md
**Milestone:** v0.2.0

**Promotion checklist:**
- [ ] No open blocking issues for this document
- [ ] Content is complete (no [PLANNED] sections)
- [ ] 14-day community review period completed
- [ ] No unresolved objections
- [ ] SPECIFICATION_STATUS.md updated to Stable
- [ ] Banner added to docs/protocols/core/roles.md

**Reference:** SPECIFICATION_STATUS.md — first spec/ candidate
```
**Labels:** `spec-promotion` · `area: core`
**Milestone:** `v0.2.0`

---

### Issue 6

**Title:**
```
needs-research: NIP-57 Zap COL-Points conversion rate
```
**Body:**
```
**Affected document:** docs/protocols/core/nostr-events.md, docs/protocols/core/reputation-protocol.md

**Question:**
At what rate do NIP-57 Zaps (Lightning payments to a pubkey) contribute to COL-Points?

**Context:**
nostr-events.md documents that "zaps on completed tickets or reviews may contribute to COL-Points. The DAO sets the conversion rate per governance vote." Before the DAO exists, a default or placeholder is needed for the spec.

**Research needed:**
- What conversion rates do comparable protocols use?
- Should there be a minimum zap threshold (currently: 1 sat / 1000 msats)?
- Should zap weight differ by the sender's identity level?

**Blocking:** No — informational, improves completeness of reputation-protocol.md
```
**Labels:** `needs-research` · `area: core`
**Milestone:** `v0.2.0`

---

*Colabonate Protocol — GitHub Project Settings Reference | 2026-03-22*
