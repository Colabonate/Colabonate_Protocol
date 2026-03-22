# GitHub Project Setup – Colabonate Protocol

Setup reference for the public project management of the Colabonate Protocol repository.
This document describes **what to set up**, **why**, and **how the parts work together**.

---

## Overview: The Four Levels

```
GitHub Project (Kanban)           ← strategic view: where do we stand?
    │
    ├── Milestones (v0.2, v0.3…)  ← tactical view: what belongs to which release?
    │       │
    │       └── Issues (with Labels) ← operational view: what is the concrete work?
    │
    └── SPECIFICATION_STATUS.md   ← document view: status of every spec file
            │
            └── spec/             ← outcome: what has passed review
```

Each level has a distinct role. They do not duplicate each other — they complement each other.

---

## 1. GitHub Project (Kanban)

### Purpose

The **strategic bird's-eye view**: which specifications are in which state? What is blocking the next milestone? For outside contributors, this is the first signal of how active and organized the project is.

### Setup

**Name:** `Colabonate Protocol — Spec Development`

**Board Columns:**

| Column | Meaning | What goes here |
|--------|---------|----------------|
| `Backlog` | Planned, not yet active | Issues with `Planned` status in SPECIFICATION_STATUS.md |
| `Open Question` | Unresolved design questions — blocking | Issues with label `open-question` |
| `In Spec` | Actively being worked on / draft phase | Issues with label `spec-proposal` in progress |
| `In Review` | Complete, awaiting community feedback | PR open, review round in progress |
| `Done` | Milestone-reviewed, moved to `spec/` | Closed issues of the respective milestone |

**Automation (GitHub Project Automation):**

| Trigger | Action |
|---------|--------|
| Issue opened with label `open-question` | → Column `Open Question` |
| Issue opened with label `spec-proposal` | → Column `Backlog` |
| Pull Request opened | → Column `In Review` |
| Issue / PR closed | → Column `Done` |

**Additional Views (beyond the board):**

- **By Milestone** (table) — all issues grouped by v0.2 / v0.3 / v0.4 / v1.0
- **Open Questions** (filtered) — label `open-question` only, sorted by priority
- **Spec Candidates** (filtered) — issues preparing a document promotion to `spec/`

### Link to SPECIFICATION_STATUS.md

Every document in `SPECIFICATION_STATUS.md` that has a `Blocking Issue` gets a corresponding GitHub Issue. The issue appears on the board. When the issue is closed, `SPECIFICATION_STATUS.md` is updated (blocking issue removed, status updated accordingly).

**Rule:** `SPECIFICATION_STATUS.md` is the **source of truth** for document status. The GitHub Project board is the **view on the work process**.

---

## 2. GitHub Milestones

### Purpose

Milestones group issues into a **release goal**. They define what `v0.2.0` means — not as a date, but as a criterion. A milestone closes when all associated issues are closed.

### Milestones to Create

#### Milestone: `v0.1.0-draft` — Public First Draft

**Status:** ✅ Closed at repository publication
**Criterion:** All core specs exist as readable drafts

```
Deliverables (all already fulfilled):
✅ vision.md, roles.md, ticket-system.md
✅ nostr-events.md (Kinds 30017–30026 + NIP-57)
✅ escrow-protocol.md, reputation-protocol.md
✅ identity-protocol.md
✅ payment-architecture.md
✅ dao-codex.md, economic-protocol.md, dao-creation-protocol.md
✅ ADRs 007–012
✅ README.md, SPECIFICATION_STATUS.md, CHANGELOG.md
```

---

#### Milestone: `v0.2.0` — Stable Schemas

**Goal:** Implementers can build Phase 1 clients — Nostr schemas and escrow state machine no longer change without a major version bump.

**Criteria (all must be met):**
- [ ] Nostr Event Schemas 30017–30026 frozen (no breaking tag changes)
- [ ] Escrow state machine finalized
- [ ] Open Question #1 (Stablecoin Escrow Denomination) closed or formally deferred to v1.0
- [ ] `vision.md` and `roles.md` promoted to `spec/` (first stable documents)
- [ ] First community review round completed (min. 1 issue/discussion per core document)
- [ ] GitHub Project and Issue Labels set up

**Associated Issues:**
- `open-question`: Stablecoin Escrow Denomination (#1)
- `spec-proposal`: Nostr Kind Schema freeze
- `editorial`: Finalize vision.md and roles.md
- `spec-promotion`: vision.md → spec/
- `spec-promotion`: roles.md → spec/

---

#### Milestone: `v0.3.0` — Identity & Reputation Complete

**Goal:** Identity Levels 0–3 fully specified, COL-Points formula finalized, reputation mechanism stable.

**Criteria:**
- [ ] `proximity-proof.md` created and present as a draft
- [ ] Humanode integration API documented (or opened as `open-question`)
- [ ] COL-Points cap defined via DAO governance process (or placeholder formalized)
- [ ] `identity-protocol.md` promoted to `spec/`
- [ ] `reputation-protocol.md` promoted to `spec/`
- [ ] `cooperation-protocol.md` draft finalized

**Associated Issues:**
- `needs-research`: Humanode API Integration Details
- `spec-proposal`: Create proximity-proof.md
- `open-question`: COL-Points Cap Governance Decision
- `spec-promotion`: identity-protocol.md → spec/

---

#### Milestone: `v0.4.0` — Governance Layer Complete

**Goal:** DAO Codex community-reviewed, Economic Protocol finalized, Dispute Protocol complete.

**Criteria:**
- [ ] `dao-codex.md` community-reviewed and promoted to `spec/`
- [ ] `economic-protocol.md` community-reviewed
- [ ] Open Question #2 (Codex Fork RSK Contract Standard) closed
- [ ] Open Question #3 (HID Linkage Threshold) closed
- [ ] `dispute-protocol.md` Level 3 arbitration fully specified
- [ ] ADR 013 and ADR 014 created

**Associated Issues:**
- `open-question`: Codex Fork RSK Contract Interface (#2)
- `open-question`: HID Linkage Enforcement Threshold (#3)
- `adr-needed`: Legal Binding Layer Opt-in (ADR 014)
- `spec-proposal`: dispute-protocol.md Level 3 Arbitration Verdict

---

#### Milestone: `v1.0.0` — Final Standard

**Goal:** All core documents in `spec/`, all open questions closed, NIP registration submitted.

**Criteria:**
- [ ] All documents in `SPECIFICATION_STATUS.md` either `Stable` (in `spec/`) or formally deferred as `Phase N`
- [ ] `security-model.md` created and in `spec/`
- [ ] `protocol-versioning.md` created and in `spec/`
- [ ] NIP registration for Kind 30017 submitted
- [ ] No open `open-question` issues without a decision
- [ ] `SPECIFICATION_STATUS.md` shows 0 `Planned` documents for Core / Identity / Workflows

---

## 3. GitHub Issues — Labels

### Purpose

Labels classify issues by **type** (what is this?) and **area** (which part of the spec?). They enable filtered views in the GitHub Project and help contributors find the right entry point.

### Label Definitions

#### Type Labels (what kind of issue is this?)

| Label | Color | Definition | Who opens it |
|-------|-------|-----------|--------------|
| `spec-proposal` | `#0075ca` blue | Proposed addition or change to a specification | Community, Core Team |
| `open-question` | `#e4e669` yellow | Unresolved design question blocking progress | Core Team when identified |
| `needs-research` | `#c5def5` light blue | Requires external research, expert input, or prototype experience | Anyone |
| `editorial` | `#d4c5f9` purple | Typo, clarity, or formatting — no content issue | Anyone |
| `adr-needed` | `#e99695` red-orange | Decision large enough to warrant an ADR | Core Team |
| `spec-promotion` | `#0e8a16` green | Candidate for promotion from `docs/protocols/` to `spec/` | Core Team after milestone |
| `good-first-issue` | `#7057ff` violet | Well-defined, limited scope — good entry point for new contributors | Core Team |
| `question` | `#cc317c` pink | General question about the protocol (no actionable item) | Community |
| `duplicate` | `#cfd3d7` grey | Already captured as another issue | Anyone |
| `wontfix` | `#cfd3d7` grey | Deliberate decision not to resolve this issue | Core Team |

#### Area Labels (which part of the spec?)

| Label | Color | Scope |
|-------|-------|-------|
| `area: core` | `#f9d0c4` salmon | Core specs (nostr-events, escrow, tickets…) |
| `area: identity` | `#f9d0c4` salmon | Identity & HID protocol |
| `area: governance` | `#f9d0c4` salmon | DAO Codex, Economics, COLA Token |
| `area: workflows` | `#f9d0c4` salmon | Buy / Sell / Cooperation / Dispute flows |
| `area: payments` | `#f9d0c4` salmon | Payment Architecture, Lightning, RSK |

#### Milestone Labels (optional, supplementing GitHub Milestones)

| Label | Usage |
|-------|-------|
| `milestone: v0.2` | Explicitly relevant for v0.2.0 |
| `milestone: v1.0` | Must be resolved before v1.0.0 |
| `milestone: post-v1` | Deliberately deferred to after v1.0 |

---

### Issue Templates

Three issue templates should be created in `.github/ISSUE_TEMPLATE/`:

**1. `spec-proposal.md`**
```
Affected document: docs/protocols/...
Type of change: [ ] New section  [ ] Change to existing section  [ ] New document
Problem being solved:
Proposed solution:
Alternatives considered:
Affected ADRs:
```

**2. `open-question.md`**
```
Affected document:
Question:
Why is this an open question (non-trivial)?
Possible answers / options:
What happens if we do not make a decision?
Blocking: [ ] yes — blocks Milestone ___  [ ] no
```

**3. `editorial.md`**
```
File:
Line / Section:
Problem:
Proposed fix:
```

---

## 4. `spec/` Directory — Usage Rules

### Purpose

`spec/` is the **quality seal** of the repository. What lives there has gone through a defined process and is considered stable enough to build against.

### Promotion Process (docs/ → spec/)

```
1. Document in docs/protocols/ is draft-complete
         │
         ↓
2. Open a GitHub Issue with label `spec-promotion`
   (checklist: no open blocking issues, milestone criteria met)
         │
         ↓
3. Community review period: 14 days (issue open for comments)
         │
         ↓
4. Core Team decides (no objections or objections addressed)
         │
         ↓
5. Pull Request:
   - Copy file from docs/protocols/X.md to spec/X.md
   - SPECIFICATION_STATUS.md: status Draft → Stable
   - docs/protocols/X.md: add banner "Stable version in spec/"
         │
         ↓
6. PR merged → close milestone issue
```

### File Structure After First Promotions (target: v0.2.0)

```
spec/
├── README.md        ← explains promotion criteria (already present)
├── vision.md        ← first candidate
└── roles.md         ← first candidate
```

### Header Convention for Files in `spec/`

Every file promoted to `spec/` carries this header block:

```markdown
**Spec Status:** Stable
**Promoted in:** v0.2.0
**Source:** docs/protocols/core/vision.md
```

---

## 5. How It All Works Together — Example Workflow

**Example: Open Question #1 is resolved (Stablecoin Escrow Denomination)**

```
1. Open Issue #1 "open-question: Stablecoin Escrow Denomination"
   Labels: open-question, area: payments, milestone: v0.2
   → automated: board column "Open Question"

2. Discussion in the issue (community input, research)

3. Decision reached: "Escrow always in sats; stablecoin amount is
   converted to sats at ticket creation time and frozen"

4. Open spec-proposal issue: "escrow-protocol.md: add stablecoin
   denomination rule"
   Labels: spec-proposal, area: payments, area: core
   → board column "In Spec"

5. PR: update escrow-protocol.md
   → board column "In Review"

6. PR merged → close Issue #1, update SPECIFICATION_STATUS.md
   (remove blocking issue for escrow-protocol.md)
   → board column "Done"

7. When all v0.2.0 criteria are met: close Milestone v0.2.0
   → write CHANGELOG.md entry for v0.2.0
```

---

## Setup Checklist (at Repository Publication)

### GitHub Repository Settings

- [ ] Set repository to `public`
- [ ] Description: `Open protocol standard for decentralized trade and cooperation via Bitcoin Lightning`
- [ ] Topics: `bitcoin`, `lightning-network`, `nostr`, `protocol`, `open-standard`, `decentralized`
- [ ] Website: `https://colabonate.com`
- [ ] Social links in repository About: X, YouTube, Instagram, OpenCollective, Patreon
- [ ] Enable Discussions
- [ ] Enable Issues
- [ ] Enable Projects

### Create Labels

- [ ] All type labels: `spec-proposal`, `open-question`, `needs-research`, `editorial`, `adr-needed`, `spec-promotion`, `good-first-issue`, `question`
- [ ] All area labels: `area: core`, `area: identity`, `area: governance`, `area: workflows`, `area: payments`
- [ ] Milestone labels: `milestone: v0.2`, `milestone: v1.0`, `milestone: post-v1`
- [ ] Remove or repurpose default labels (`bug` → not applicable, `enhancement` → `spec-proposal`)

### Create Milestones

- [ ] `v0.1.0-draft` — mark as closed (already fulfilled)
- [ ] `v0.2.0` — include criteria description
- [ ] `v0.3.0` — include criteria description
- [ ] `v0.4.0` — include criteria description
- [ ] `v1.0.0` — include criteria description

### Create GitHub Project

- [ ] New project: `Colabonate Protocol — Spec Development`
- [ ] Board type: Kanban
- [ ] Create columns: Backlog / Open Question / In Spec / In Review / Done
- [ ] Configure automation rules
- [ ] Create views: By Milestone, Open Questions, Spec Candidates
- [ ] Link repository to project

### Open First Issues (immediately after publication)

- [ ] Issue #1: `open-question: Stablecoin Escrow Denomination` — Labels: `open-question`, `area: payments`, `milestone: v0.2`
- [ ] Issue #2: `open-question: Codex Fork RSK Contract Interface` — Labels: `open-question`, `area: governance`, `milestone: v0.4`
- [ ] Issue #3: `open-question: HID Linkage Enforcement Threshold` — Labels: `open-question`, `area: identity`, `milestone: v0.4`
- [ ] Issue #4: `spec-promotion: vision.md → spec/` — Labels: `spec-promotion`, `area: core`, `milestone: v0.2`
- [ ] Issue #5: `spec-promotion: roles.md → spec/` — Labels: `spec-promotion`, `area: core`, `milestone: v0.2`
- [ ] Issue #6: `needs-research: NIP-57 Zap COL-Points conversion rate` — Labels: `needs-research`, `area: core`, `milestone: v0.2`

### Create Issue Templates

- [ ] `.github/ISSUE_TEMPLATE/spec-proposal.md`
- [ ] `.github/ISSUE_TEMPLATE/open-question.md`
- [ ] `.github/ISSUE_TEMPLATE/editorial.md`

---

*Colabonate Protocol — GitHub Project Setup Reference | 2026-03-22*
