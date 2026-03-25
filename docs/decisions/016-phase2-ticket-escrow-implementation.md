# 011 – Phase 2 Ticket- & Escrow-Systeme: Implementierungsentscheidungen

**Status:** accepted
**Date:** 2026-03-25
**Affects:** `apps/server/index.ts`, `apps/server/services/nostr-ticket-listener.ts`, `apps/colabonate-app/src/lib/nostr.ts`, `apps/colabonate-app/src/components/tickets/`, `apps/colabonate-app/src/components/offers/`, `docs/protocols/core/ticket-system.md`, `docs/protocols/core/escrow-protocol.md`

## Context

Phase 2 der Colabonate-Implementierung erfordert drei Kernfähigkeiten, die bisher fehlten:
1. Eine Möglichkeit, **Milestone-Tickets** via Nostr-Events (Kind 30018) zu erzeugen und in der DB zu indexieren.
2. Einen **User-Flow** (direkter Ticket-Kauf) aus der `OfferDetailView`, der den User entscheiden lässt, ob er per Smart Order oder per Milestone kauft.
3. Die **Lightning Escrow Phasen** (25%/50%/25%) in der `TicketDetailView` sequenziell abbilden zu können.

## Decision

### D1 – Milestone Payload als JSON im `content`-Feld (Kind 30018)

Milestones werden **nicht** als einzelne Nostr-Tags kodiert, sondern als JSON-Array im `content`-Feld des Kind 30018 Events:

```json
content: "[{\"title\":\"Phase 1\",\"amountSats\":5000},{...]"
```

Ein zusätzlicher Tag `["type", "MILESTONE"]` ermöglicht das schnelle Filtern ohne Content-Parsing.

**Begründung:** Maximale Flexibilität für die Datenstruktur (beliebig viele Felder pro Milestone), schnelleres Backend-Processing, JSON-Standard.

### D2 – Progressive Escrow Invoice Generierung

Die drei Hold Invoices (25%/50%/25%) werden **nicht** auf einmal beim Ticket-Erstellen generiert, sondern **progressiv** — jede Phase erst wenn die vorherige abgeschlossen ist:

- `POST /api/tickets/:id/escrow-phase { phase: 1|2|3 }` generiert die jeweilige Invoice.
- Die Invoice wird in `phase1Invoice`, `phase2Invoice`, `phase3Invoice` auf dem Ticket gespeichert.
- Der `escrowStatus` wird auf `PHASE_X_PENDING` gesetzt.
- Ein Kind 30019 Nostr-Event wird als Audit-Trail publiziert.

**Begründung:** Entspricht dem `escrow-protocol.md` State Machine. Verhindert LNBits-Hold-Invoice-Timeouts für Phasen, die noch nicht fällig sind.

### D3 – Mock-First für LNBits Hold Invoices

Solange keine echte LNBits-Instanz konfiguriert ist (`LNBITS_ADMIN_KEY` + `LNBITS_INVOICE_KEY` fehlen in `.env`), generiert das Backend **Mock-BOLT11-Strings**. Das Pattern basiert auf dem bestehenden `LNBitsService.generatePaymentHash()`-Mechanismus.

**Begründung:** Vollständig lokale Entwicklung ohne externe Abhängigkeiten, konsistent mit ADR-010 (Mock Nostr Relay).

### D4 – Strikte Summen-Validierung im OfferConfirmationDialog

Die Summe aller Milestone-Beträge muss **exakt** dem Angebotspreis (`offer.priceSats`) entsprechen, bevor das Ticket erstellt werden kann.

**Begründung:** Verhindert wirtschaftliche Inkonsistenz. Das Nostr-Netzwerk und die DB-Validierung kennen den Originalpreis; Abweichungen würden zu Escrow-Release-Konflikten führen.

## Rejected Alternatives

- **Milestones als Nostr-Tags** (`["milestone", "title", "5000"]`) – abgelehnt weil: keine native Unterstützung für verschachtelte Objekte in Nostr-Tags, mehr verbosity, schwerer zu parsen.
- **Alle 3 Invoices auf einmal generieren** – abgelehnt weil: HTLC-Timeouts würden Phasen-Invoices entwerten bevor sie fällig sind.
- **Frontend-Summen-Validierung weglassen / Warnung statt Block** – abgelehnt weil: Protokollkonsistenz muss erzwungen werden, nicht optiert werden.

## Consequences

- Agenten, die Kind 30018 Events verarbeiten, **müssen** das `type`-Tag prüfen und bei `MILESTONE` das `content`-Feld als JSON parsen.
- Die `publishTicketRequest`-Funktion in `lib/nostr.ts` ist die einzige kanonische Stelle für Kind 30018 Events — immer dort anpassen.
- Neue Escrow-States (`PHASE_1_PENDING`, `RESERVED`, `PHASE_2_PENDING`, `DELIVERY_STARTED`, `PHASE_3_PENDING`, `DELIVERY_CONFIRMED`, `RELEASED`) sind als `EscrowStatus`-Enum in `prisma/schema.prisma` zu ergänzen (teilweise noch offen).
- Anpassungen an `docs/protocols/core/` erfordern immer ein PDC (Protocol Decision Change) als ADR-Eintrag.

## References

- Protokoll: [`docs/protocols/core/ticket-system.md`](../protocols/core/ticket-system.md)
- Protokoll: [`docs/protocols/core/escrow-protocol.md`](../protocols/core/escrow-protocol.md)
- Backend Indexer: [`apps/server/services/nostr-ticket-listener.ts`](../../apps/server/services/nostr-ticket-listener.ts)
- Frontend Offer Confirmation: [`apps/colabonate-app/src/components/offers/OfferConfirmationDialog.tsx`](../../apps/colabonate-app/src/components/offers/OfferConfirmationDialog.tsx)
- Frontend Escrow UI: [`apps/colabonate-app/src/components/tickets/TicketDetailView.tsx`](../../apps/colabonate-app/src/components/tickets/TicketDetailView.tsx)
- Backend Endpoint: [`apps/server/index.ts`](../../apps/server/index.ts) — `POST /api/tickets/:id/escrow-phase`
- Nostr-Lib: [`apps/colabonate-app/src/lib/nostr.ts`](../../apps/colabonate-app/src/lib/nostr.ts)
- Verwandte ADRs: [ADR-009](009-decentralized-nostr-ticket-system.md), [ADR-010](010-local-mock-nostr-relay.md)
