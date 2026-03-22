# Cooperation Protocol

**Status:** [PHASE 2] — Concept finalized, schema extensions defined, not yet implemented

## Purpose

Enables structured collaboration between multiple parties with defined milestones,
payment plans, and transparent tracking.

Unlike the Buy Protocol (1 buyer + 1 seller, one-time), this covers
**longer-term, multi-step collaboration**.

---

## 4-Phase Flow

### Phase A — Initiation and Matching

```
1. Initiator describes cooperation goal
   → Title, description, required skills, budget (sats), timeline

2. Published via Nostr (Kind 30018, planned)
   or sent directly to known partner (Nostr DM)

3. Partner responds with interest or counter-offer

4. Both parties agree on terms
   (chat integration, Phase 3)
```

### Phase B — Contract Formation

```
5. Initiator creates Milestone Ticket
   POST /api/tickets  { ticketType: "MILESTONE", ... }  [PHASE 2]

6. Milestones are defined:
   { title, description, dueDate, amountSats, acceptanceCriteria }

7. Partner signs via pubkey (Nostr Event as confirmation)

8. Escrow deposit by Initiator (full amount as Hold Invoice)
```

### Phase C — Execution and Tracking

```
9. Milestone work begins (Status: IN_PROGRESS)

10. Partner reports milestone completion + evidence

11. Initiator reviews and confirms
    → Partial payment released (proportional from escrow)

12. Continue with step 9 for the next milestone
```

### Phase D — Completion and Reputation

```
13. Final milestone completed
    → Full ticket: Status COMPLETED

14. Mutual reviews
    → Nostr Events (Soulbound, Phase 3)

15. Reputation bound to pubkeys
```

### Phase E — Conflict Resolution (optional)

```
16. On dispute: → Dispute Protocol (dispute-protocol.md)
    (3 levels: self-resolution → mediation → DAO court)
```

---

## Comparison: Smart Order vs. Cooperation

| | Smart Order (Buy) | Cooperation |
|-|-------------------|-------------|
| Parties | 1 buyer + 1 seller | n parties possible |
| Time structure | One-time | Multiple milestones |
| Payment | Escrow phases (25/50/25) | Per milestone |
| Outcome | Product / service | Joint deliverable |
| Review | One-sided | Mutual |
| Ticket type | `SMART_ORDER` | `MILESTONE` |

---

## Required Schema Extensions (Phase 2)

```prisma
enum TicketType {
  SMART_ORDER    // Buy/sell (current default)
  MILESTONE      // Cooperation with milestones
  DISPUTE        // Conflict resolution
  GOVERNANCE     // DAO vote
  VERIFICATION   // Identity / skill verification
  ROYALTY        // Royalty distribution
}

enum MilestoneStatus {
  PENDING
  IN_PROGRESS
  COMPLETED
  DISPUTED
}

model Milestone {
  id                 String          @id @default(cuid())
  ticketId           String
  ticket             Ticket          @relation(fields: [ticketId], references: [id])
  title              String
  description        String?
  acceptanceCriteria String?
  dueDate            DateTime?
  status             MilestoneStatus @default(PENDING)
  invoice            String?
  paymentHash        String?
  amountSats         Int?
  completedAt        DateTime?
  createdAt          DateTime        @default(now())
  updatedAt          DateTime        @updatedAt
}

// Ticket gains new fields:
// ticketType  TicketType  @default(SMART_ORDER)
// milestones  Milestone[]
```

---

## Cooperation Types

All cooperation types share the same technical foundation (Milestone Ticket).
The difference lies in the social context:

| Type | Description | Example |
|------|-------------|---------|
| Transactional | Short-term, one-off | Freelance task |
| Production | Joint manufacturing | Two craftspeople building together |
| Sales | Joint distribution | Cross-promotion |
| Service | Service bundle | Pizza shop + delivery partner |
| Educational | Knowledge sharing | Workshop, course |
| Crowdfunding | Collective financing | Community project |
| Barter | Without money | Skill exchange |

---

## Recommended Entry Path for New Users

1. **Barter rings** — minimal effort, quick wins
2. **Simple services** — e.g. "Math tutoring", "Bike repair"
3. **Small projects** — 2–3 milestones, manageable budget
4. **Complex cooperations** — multiple partners, governance elements
