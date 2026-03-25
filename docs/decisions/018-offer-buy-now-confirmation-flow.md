# 018 – Offer "Buy Now" Confirmation Flow

**Status:** accepted
**Date:** 2026-03-25
**Affects:** `apps/colabonate-app/src/components/offers/OfferDetailView.tsx`, `apps/colabonate-app/src/components/offers/OfferConfirmationDialog.tsx`, `docs/protocols/workflows/buy-protocol.md`

## Context

Beim Kauf eines Angebots über den "Buy Now"-Button in der `OfferDetailView` wurde zuvor der `TicketCreationWizard` verwendet. Dieser Dialog war als "Ticket erstellen" betitelt und suggerierte dem Nutzer, dass er gerade ein neues Ticket manuell erstellt – nicht, dass er einen Kauf abschließt.

**Problem:** Der Dialog-Text war inkonsistent mit dem tatsächlichen User-Flow gemäß `buy-protocol.md`:
1. User klickt "Buy Now" → soll Kauf bestätigen
2. Ticket wird im Hintergrund erstellt (Status: PENDING)
3. User wird zur Zahlung aufgefordert

Die Formulierung "Ticket erstellen" war verwirrend, da der Nutzer ein Angebot **kaufen** möchte – kein Ticket **erstellen**.

## Decision

Der `TicketCreationWizard` wird durch einen `OfferConfirmationDialog` ersetzt mit folgenden Änderungen:

### Dialog-Titel und -Texte
- **Titel:** "Kauf bestätigen" (statt implizite Ticket-Erstellung)
- **Button:** "Kaufen & Bezahlen" (statt "Ticket erstellen")
- **Zusatzinfo:** Hinweis auf nächste Schritte (Lightning-Rechnung, Zahlung, Escrow-Sicherung)

### Komponente
- **Neu:** `apps/colabonate-app/src/components/offers/OfferConfirmationDialog.tsx`
- **Gelöscht:** `apps/colabonate-app/src/components/tickets/TicketCreationWizard.tsx`

### API-Call Dokumentation
Der `POST /api/tickets`-Aufruf im Dialog wird explizit kommentiert:
```typescript
// Erstellt ein Ticket im Status PENDING
// Der Käufer wird anschließend zur Zahlung der Lightning Invoice aufgefordert
```

## Rejected Alternatives

- **TicketCreationWizard beibehalten und nur Texte ändern** – abgelehnt weil: Der Name "Wizard" suggeriert einen mehrstufigen Prozess; tatsächlich ist es ein einfacher Bestätigungsdialog.
- **Separaten "PurchaseFlow"-Component erstellen** – abgelehnt weil: Overengineering; ein Dialog reicht für Phase 1 aus.
- **Direkte Ticket-Erstellung ohne Bestätigung** – abgelehnt weil: Kaufabschluss erfordert explizite Nutzerbestätigung (rechtlich und UX).

## Consequences

- **UX-Klarheit:** Nutzer verstehen, dass sie ein Angebot kaufen – kein Ticket "erstellen".
- **Protokoll-Konsistenz:** Der Flow entspricht nun dem `buy-protocol.md` Phase 1 (Ticket wird im PENDING-Status erstellt).
- **Komponenten-Ordnung:** `OfferConfirmationDialog` liegt im `offers/`-Ordner (nicht `tickets/`), da er angebotsbezogen ist.
- **ADR-Update:** ADR-016 wurde aktualisiert, um den neuen Komponentennamen zu referenzieren.

## References

- Protokoll: [`docs/protocols/workflows/buy-protocol.md`](../protocols/workflows/buy-protocol.md)
- Protokoll: [`docs/protocols/core/ticket-system.md`](../protocols/core/ticket-system.md)
- Frontend: [`apps/colabonate-app/src/components/offers/OfferConfirmationDialog.tsx`](../../apps/colabonate-app/src/components/offers/OfferConfirmationDialog.tsx)
- Frontend: [`apps/colabonate-app/src/components/offers/OfferDetailView.tsx`](../../apps/colabonate-app/src/components/offers/OfferDetailView.tsx)
- Verwandte ADR: [ADR-016](016-phase2-ticket-escrow-implementation.md)
