# Ticket System – Colabonate

The ticket is the core of Colabonate. Every interaction between parties is
represented as a ticket — it is simultaneously a contract, a payment instruction,
and a communication channel.

> (PDC: see ADR-021) – Added OfferType SERVICE vs PRODUCT distinction

## Current State (Phase 1 — implemented)

The schema has a single ticket type with these fields:

```prisma
model Ticket {
  id               String       @id @default(cuid())
  offerId          String
  buyerPubkey      String       // = Partner
  sellerPubkey     String       // = Initiator
  status           TicketStatus @default(PENDING)
  lightningInvoice String?
  paymentHash      String?
  amountSats       Int?
  disputeReason    String?
  createdAt        DateTime     @default(now())
  updatedAt        DateTime     @updatedAt
}
```

**Active API endpoints:**
```
POST   /api/tickets                       Create ticket
GET    /api/tickets?pubkey=...            Get tickets for a user
GET    /api/tickets/:id                   Ticket details
PATCH  /api/tickets/:id                   Update status
GET    /api/tickets/:id/payment-status    Payment polling (LNBits)
```

## Status Flow (Phase 1)

```
PENDING
  ├─► (accept) ─► IN_PROGRESS ─► COMPLETED
  ├─► (reject) ─► CANCELLED
  └─► (either) ─► DISPUTED ─► COMPLETED | CANCELLED
```

> `ACCEPTED` is in the schema but not yet active in routing.
> The transition currently goes directly: PENDING → IN_PROGRESS via `action: accept`.

## Current Schema — Additional Fields (Phase 2+)

The Phase 1 schema is extended in Phase 2 with legal binding fields, ticket type, and offer type:

```prisma
model Ticket {
  id               String       @id @default(cuid())
  offerId          String
  buyerPubkey      String
  sellerPubkey     String
  status           TicketStatus @default(PENDING)
  offerType        OfferType    @default(SERVICE)  // NEW: SERVICE | PRODUCT
  ticketType       TicketType   @default(SMART_ORDER)
  daoId            String?      // DAO binding (see legal-binding-layer.md)
  codexHash        String?      // SHA-256 of Codex at binding time
  codexVersion     String?      // Human-readable Codex version (e.g. "1.2")
  escrowStatus     EscrowStatus?
  
  // Escrow phase invoices
  phase1Invoice    String?
  phase2Invoice    String?
  phase3Invoice    String?
  deliveryConfirmedAt DateTime?
  
  createdAt        DateTime     @default(now())
  updatedAt        DateTime     @updatedAt
}

enum OfferType {
  SERVICE   // 1x per buyer, seller must accept
  PRODUCT   // unlimited quantity, auto-accepted
}

enum TicketType {
  SMART_ORDER
  MILESTONE
  DISPUTE
  GOVERNANCE
  VERIFICATION
  ROYALTY
}

enum EscrowStatus {
  FUNDED
  RELEASE_PHASE_1
  RELEASE_PHASE_2
  RELEASE_PHASE_3
  RELEASED_FULL
  REFUNDED
}
```

**OfferType Behavior:**

| OfferType | Status after creation | Seller approval required | UI Action |
|-----------|----------------------|-------------------------|-----------|
| SERVICE   | PENDING              | Yes                     | "Request" |
| PRODUCT   | IN_PROGRESS          | No (auto-accepted)      | "Buy Now" |

**Status Flow (Phase 2):**

```
SERVICE Flow:
PENDING ──(accept)──► IN_PROGRESS ──► PAID ──► COMPLETED
          (reject)──► CANCELLED

PRODUCT Flow:
IN_PROGRESS ──► PAID ──► COMPLETED  (no PENDING state)

Both Flows:
Any state ──► DISPUTED ──► COMPLETED | CANCELLED
```

When `daoId` + `codexHash` are set, the ticket is bound to a DAO Codex.
See [legal-binding-layer.md](./legal-binding-layer.md) for full specification.

---

## 8 Ticket Types (Whitepaper v6 — Full Definition)

### 1. Smart Order Ticket [PHASE 2]

Buy/sell with Lightning escrow phases.

**Planned escrow structure:**
- 25% on ticket acceptance (reservation)
- 50% on delivery start / milestone
- 25% on completion and review

**Required schema changes:**
```prisma
ticketType       TicketType    @default(SMART_ORDER)
escrowStatus     EscrowStatus?
phase1Invoice    String?
phase2Invoice    String?
phase3Invoice    String?
deliveryConfirmedAt DateTime?
```

### 2. Milestone Ticket [PHASE 2]

Project progress tracking for cooperations. Each milestone has its own
acceptance criteria and a separate Lightning invoice for partial payment.

**Requires new Prisma model:**
```prisma
model Milestone {
  id          String          @id @default(cuid())
  ticketId    String
  ticket      Ticket          @relation(...)
  title       String
  description String?
  dueDate     DateTime?
  status      MilestoneStatus @default(PENDING)
  invoice     String?
  amountSats  Int?
  completedAt DateTime?
}
```

### 3. Dispute Ticket [PHASE 4]

Three-level conflict resolution — see `workflows/dispute-protocol.md`.

**Trigger:**
- Manual: `PATCH /api/tickets/:id { status: "DISPUTED", disputeReason: "..." }`
- Automatic: timeout without completion [PHASE 2]

### 4. Governance Ticket [PHASE 4+]

DAO votes via Nostr Events. Initial voting weight: 1 pubkey = 1 vote.

### 5. Verification Ticket [PHASE 3]

Identity and skill verification. Non-transferable (Soulbound).
Technically: a Nostr Event co-signed by a verified pubkey.

**No NFT, no EVM** — purely Bitcoin/Nostr-based.

### 6. Royalty Ticket [PHASE 5]

Automatic royalty distribution when community protocols and workflows are used.
Based on Lightning Keysend or a similar mechanism.

### 7. Rating Ticket [PHASE 3]

**Source:** Whitepaper v6, Section 3.3

Dedicated ticket type for capturing structured feedback and building reputation scores. Distinct from the general review system (Kind 30024) — a Rating Ticket is a formal, protocol-level rating artifact that is:

- Linked to a completed Smart Order or Milestone Ticket
- Structured (star rating + dimensions + free text)
- Published as a Nostr event (Kind 30024)
- Triggers COL-Points award upon mutual submission

**Relationship to reputation-protocol.md:** The Rating Ticket is the protocol-level trigger for a Kind 30024 event. It is not a new Nostr event kind — it is the formal process that creates one.

**Schema additions:**

```
ticketType     TicketType    // RATING
parentTicketId String        // reference to completed ticket being rated
ratingFor      String        // pubkey of the person being rated
stars          Int           // 1-5
dimensions     Json?         // optional structured rating dimensions
reviewText     String?       // free text (max 2048 chars)
nostrEventId   String?       // Kind 30024 event published on completion
```

**Status flow:**
```
PENDING → SUBMITTED (rating text entered)
        → PUBLISHED (Kind 30024 event published on Nostr)
        → MUTUAL_COMPLETE (both parties have submitted)
```

### 8. Return Ticket [PHASE 4]

**Source:** Whitepaper v6, Section 3.3

Handles product returns, service complaints, and refund processes. Initiated by the buyer after a ticket is COMPLETED (within a return window defined in the original offer).

**Properties:**
- Buyer-initiated only
- Linked to a COMPLETED Smart Order Ticket
- Opens a structured return/refund process independent of the main dispute protocol
- Lower friction than a Dispute Ticket — designed for cases where return is agreed upon, not contested
- If return is contested → escalates to Dispute Ticket (Level 1)

**Return window:** Defined by seller in the offer metadata (`["return_window", "<days>"]` tag on Kind 30017). Default: 0 days (no returns) unless explicitly set.

**Schema additions:**

```
ticketType       TicketType    // RETURN
parentTicketId   String        // reference to COMPLETED ticket
returnReason     String        // reason slug: defective | not_as_described | changed_mind | other
returnDescription String?      // free text explanation
evidenceRefs     String[]      // references to evidence (hashes of photos, etc.)
refundAmountSats Int?          // proposed refund amount (may be partial)
returnStatus     ReturnStatus  // REQUESTED | SELLER_ACCEPTED | IN_TRANSIT | COMPLETED | DISPUTED
```

**Status flow:**
```
REQUESTED
  ├── Seller accepts → SELLER_ACCEPTED
  │     └── Buyer confirms return sent → IN_TRANSIT
  │           └── Seller confirms receipt → COMPLETED (refund released from escrow)
  └── Seller rejects → DISPUTED (escalates to Dispute Ticket Level 1)
```

**Note on escrow:** For a Return Ticket to trigger an automatic refund, the original ticket must have used escrow (Phase 2+). Phase 1 returns are handled off-protocol.

**Note on Reputation Tickets (Section 3.3):** The Whitepaper lists "Reputation Tickets" separately. In the protocol architecture, reputation is managed through Rating Tickets (Type 7) and the Verification Ticket (Type 5, existing). A separate Reputation Ticket type may be introduced in Phase 5 for documenting community contributions and SBT management. Flagged as [CONCEPT] pending further specification.

## Dependency Map

```
Smart Order Ticket
  └─► can escalate to ────► Dispute Ticket

Milestone Ticket
  └─► extends ────────────► Smart Order Ticket (parent)
  └─► can escalate to ────► Dispute Ticket

Governance Ticket
  └─► can affect ─────────► Dispute Ticket (arbitrator election)

Verification Ticket
  └─► required for ───────► Mediator role (Phase 4)
```

## Use Cases by Area

| Area | Ticket Type | Phase |
|------|-------------|-------|
| Buy / Sell | Smart Order Ticket | Phase 2 (base: Phase 1) |
| Project / Cooperation | Milestone Ticket | Phase 2 |
| Conflict | Dispute Ticket | Phase 4 |
| Voting | Governance Ticket | Phase 4+ |
| Identity / Skills | Verification Ticket | Phase 3 |
| Protocol usage | Royalty Ticket | Phase 5 |
| Structured feedback | Rating Ticket | Phase 3 |
| Returns / Refunds | Return Ticket | Phase 4 |
