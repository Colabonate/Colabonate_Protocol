# 021 – Smart Order Escrow Flow: OfferType SERVICE vs PRODUCT

**Status:** accepted
**Date:** 2026-03-25
**Affects:** `prisma/schema.prisma`, `apps/server/index.ts`, `apps/playground/src/pages/Simulator.tsx`, `apps/colabonate-app/src/components/tickets/TicketDetailView.tsx`, `apps/colabonate-app/src/components/offers/OfferDetailView.tsx`, `docs/protocols/workflows/buy-protocol.md`, `docs/protocols/core/ticket-system.md`

---

## Context

### Aktuelle Situation (Phase 1)

Das aktuelle Offer-Schema kennt keine Unterscheidung nach Verfügbarkeit:
- Jedes Offer kann beliebig oft gekauft werden
- Jedes Ticket startet im Status `PENDING`
- Buyer kann sofort "Buy Now" klicken und Ticket erstellen
- Keine Unterscheidung zwischen Dienstleistung (1x pro Buyer) und Produkt (unbegrenzt)

**Problem:** Dieser Ansatz passt nicht für alle Use Cases:
1. **Dienstleistungen (SERVICE):** Seller möchte jeden Kunden individuell prüfen und zustimmen (z.B. Beratung, Coaching, Custom Development)
2. **Produkte (PRODUCT):** Seller möchte unbegrenzte Stückzahlen ohne manuelle Zustimmung verkaufen (z.B. E-Books, digitale Assets, Templates)

### Anforderung

Unterscheidung zwischen zwei Offer-Typen mit unterschiedlichem Flow:

| Aspekt | SERVICE | PRODUCT |
|--------|---------|---------|
| **Verfügbarkeit** | 1x pro Buyer | Unbegrenzt |
| **Seller-Zustimmung** | Erforderlich vor Payment | Automatisch |
| **Ticket Status nach "Buy Now"** | `PENDING` | `IN_PROGRESS` |
| **UI für Buyer** | "Warte auf Seller-Zustimmung" | "Payment starten" |
| **UI für Seller** | "Annehmen" / "Ablehnen" Buttons | Direkt Invoice erstellen |

---

## Decision

### D1 – OfferType Enum im Schema

```prisma
enum OfferType {
  SERVICE   // 1x pro Buyer, Seller muss zustimmen
  PRODUCT   // Unbegrenzt, automatisch angenommen
}

model Offer {
  // ... bestehende Felder ...
  offerType    OfferType  @default(SERVICE)
  tickets      Ticket[]
}

model Ticket {
  // ... bestehende Felder ...
  offerType  OfferType  // Von Offer kopiert bei Erstellung
}
```

**Begründung:** 
- Einfache, klare Unterscheidung auf Datenbank-Ebene
- OfferType wird an Ticket vererbt → Flow-Logik kann am Ticket entscheiden
- Default `SERVICE` für Backwards-Kompatibilität

---

### D2 – Backend: Ticket-Erstellung mit OfferType-Logik

**Endpoint:** `POST /api/tickets`

```typescript
// 1. Verfügbarkeit prüfen (nur bei SERVICE)
if (offer.offerType === 'SERVICE') {
  const existingTicket = await prisma.ticket.findFirst({
    where: {
      offerId: offer.id,
      buyerPubkey: buyerPubkey,
      status: { notIn: ['CANCELLED', 'COMPLETED'] }
    }
  })
  if (existingTicket) {
    return res.status(400).json({ 
      error: 'Sie haben dieses Service-Angebot bereits gekauft. Ein aktives Ticket existiert bereits.' 
    })
  }
}

// 2. Ticket erstellen mit offerType und Status-Logik
const ticket = await prisma.ticket.create({
  data: {
    offerId,
    buyerPubkey,
    sellerPubkey: offer.creatorPubkey,
    amountSats,
    offerType: offer.offerType,  // Von Offer kopieren
    // Status-Logik:
    // - SERVICE → PENDING (Seller muss zustimmen)
    // - PRODUCT → IN_PROGRESS (automatisch angenommen)
    status: offer.offerType === 'PRODUCT' ? 'IN_PROGRESS' : 'PENDING'
  },
  include: { offer: true }
})
```

**Begründung:**
- SERVICE-Tickets erfordern Seller-Akzeptanz vor Payment-Start
- PRODUCT-Tickets starten direkt im Payment-Flow
- Ein Buyer kann nur ein aktives SERVICE-Ticket pro Offer haben

---

### D3 – Frontend: TicketDetailView Flow-Anpassung

**Für SERVICE-Tickets (Status PENDING):**

```tsx
{ticket.offerType === 'SERVICE' && ticket.status === 'PENDING' && (
  <>
    {/* Buyer-Sicht */}
    {isBuyer && (
      <Alert severity="warning">
        <Typography variant="body2" fontWeight={600}>
          Warte auf Seller-Zustimmung
        </Typography>
        <Typography variant="caption">
          Der Verkäufer muss Ihre Anfrage zuerst annehmen. Sie erhalten eine Benachrichtigung, 
          sobald das Ticket angenommen wurde.
        </Typography>
      </Alert>
    )}
    
    {/* Seller-Sicht */}
    {isSeller && (
      <Box>
        <Typography variant="subtitle2" fontWeight={600} mb={1}>
          Anfrage prüfen
        </Typography>
        <Box sx={{ display: 'flex', gap: 1 }}>
          <Button
            fullWidth
            variant="contained"
            color="success"
            startIcon={<ThumbUpIcon />}
            onClick={() => handleSellerAction('accept')}
          >
            Annehmen
          </Button>
          <Button
            fullWidth
            variant="outlined"
            color="error"
            startIcon={<ThumbDownIcon />}
            onClick={() => handleSellerAction('reject')}
          >
            Ablehnen
          </Button>
        </Box>
      </Box>
    )}
  </>
)}
```

**Für PRODUCT-Tickets (Status IN_PROGRESS):**

```tsx
{ticket.offerType === 'PRODUCT' && (
  <Alert severity="success">
    <Typography variant="caption">
      ✓ Produkt automatisch angenommen – Payment kann starten
    </Typography>
  </Alert>
)}
```

**Begründung:**
- Klare visuelle Unterscheidung für User
- Buyer weiß sofort, ob er warten muss oder handeln kann
- Seller sieht kontext-spezifische Aktionen

---

### D4 – Frontend: OfferDetailView "Buy Now" Anpassung

**Button-Text und Hinweis dynamisch:**

```tsx
<Button
  variant="contained"
  size="large"
  fullWidth
  startIcon={offer.offerType === 'SERVICE' ? <FlagIcon /> : <ShoppingCartIcon />}
  onClick={handleBuyNow}
>
  {offer.offerType === 'SERVICE' ? 'Anfrage stellen' : 'Jetzt kaufen'}
</Button>

<Alert 
  severity={offer.offerType === 'SERVICE' ? 'warning' : 'success'}
  icon={offer.offerType === 'SERVICE' ? <InfoOutlinedIcon /> : <CheckCircleIcon />}
  sx={{ mt: 2 }}
>
  {offer.offerType === 'SERVICE' 
    ? '⚠️ Der Verkäufer muss Ihre Anfrage zuerst annehmen. Sie erhalten eine Benachrichtigung nach der Zustimmung.'
    : '✓ Sofort verfügbar – unbegrenzte Stückzahl. Payment startet direkt nach dem Kauf.'}
</Alert>
```

**Begründung:**
- Erwartungsmanagement vor dem Kauf
- Buyer weiß vor "Buy Now" was passiert
- Verhindert Verwirrung über "Warte auf Zustimmung"-Status

---

### D5 – Playground Protocol Simulator Erweiterungen

**Neue UI-Elemente:**

1. **OfferType Auswahl** (Radio Buttons / Toggle)
   - Service (1x pro Buyer)
   - Produkt (unbegrenzt)

2. **Template-Erweiterung** mit `offerType` Feld

3. **Status-Log-Anzeige** mit OfferType-Info:
   ```
   [FLOW] SERVICE Ticket erstellt: "Logo Design" → Status: PENDING
   [FLOW] PRODUCT Ticket erstellt: "E-Book" → Status: IN_PROGRESS
   ```

4. **Seller-Aktionen Alert** für SERVICE-Tickets im PENDING-Status

**Begründung:**
- Simulator muss neuen Flow abbilden können
- Entwickler können beide Flows testen
- Visuelles Feedback für Status-Unterschiede

---

### D6 – Status Flow Diagramm (Dokumentation)

```
┌─────────────────────────────────────────────────────────────────┐
│                    SERVICE Flow (1x pro Buyer)                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Buyer: "Anfrage stellen"                                       │
│              ↓                                                  │
│  Ticket erstellt (Status: PENDING)                              │
│              ↓                                                  │
│  Seller: "Annehmen" ✓  oder  "Ablehnen" ✗                       │
│              ↓                                                  │
│  Bei Annehmen: Status → IN_PROGRESS                             │
│              ↓                                                  │
│  Seller: Phase 1 Invoice erstellen (25%)                        │
│              ↓                                                  │
│  Buyer: Phase 1 Invoice bezahlen                                │
│              ↓                                                  │
│  Escrow: HELD (25% im Escrow)                                   │
│              ↓                                                  │
│  Seller: Lieferung + Phase 2 Invoice (50%)                      │
│              ↓                                                  │
│  Buyer: Phase 2 Invoice bezahlen                                │
│              ↓                                                  │
│  Escrow: PARTIALLY_RELEASED (75% total)                         │
│              ↓                                                  │
│  Buyer: "Zahlung freigeben" + Phase 3 Invoice (25%)             │
│              ↓                                                  │
│  Buyer: Phase 3 Invoice bezahlen                                │
│              ↓                                                  │
│  Escrow: FULLY_RELEASED (100%) → Ticket: COMPLETED              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   PRODUCT Flow (unbegrenzt)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Buyer: "Jetzt kaufen"                                          │
│              ↓                                                  │
│  Ticket erstellt (Status: IN_PROGRESS) ← Automatisch angenommen │
│              ↓                                                  │
│  Seller: Phase 1 Invoice erstellen (25%)                        │
│              ↓                                                  │
│  Buyer: Phase 1 Invoice bezahlen                                │
│              ↓                                                  │
│  Escrow: HELD (25% im Escrow)                                   │
│              ↓                                                  │
│  (Identisch zu SERVICE ab hier)                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Rejected Alternatives

### Alternative A: Separate Endpoints für SERVICE/PRODUCT

```typescript
POST /api/tickets/service  // Mit Pending-Status
POST /api/tickets/product  // Mit InProgress-Status
```

**Abgelehnt weil:**
- Code-Duplikation
- Zwei Endpoints warten statt einem
- OfferType ist eine Eigenschaft des Offers, kein separater Use Case

### Alternative B: Boolean-Feld `requiresApproval` statt Enum

```prisma
model Offer {
  requiresApproval  Boolean  @default(true)
}
```

**Abgelehnt weil:**
- Nicht erweiterbar (z.B. für zukünftige Typen wie `LIMITED`)
- Enum ist semantisch klarer (`SERVICE` vs `PRODUCT` statt `true` vs `false`)
- Bessere Lesbarkeit im Code (`offer.offerType === 'SERVICE'`)

### Alternative C: Alle Tickets starten in PENDING

```typescript
// Immer PENDING, auch für PRODUCT
status: 'PENDING'
```

**Abgelehnt weil:**
- PRODUCT-Käufer müssen unnötig warten
- Schlechte UX für digitale Produkte
- Seller muss jeden PRODUCT-Kauf manuell bestätigen (Skalierungsproblem)

### Alternative D: Availability-Count statt Typen

```prisma
model Offer {
  maxQuantity  Int?  // null = unbegrenzt, 1 = SERVICE, >1 = LIMITED
}
```

**Abgelehnt weil:**
- Zu komplex für Phase 2
- SERVICE-Logik (Seller-Zustimmung) nicht direkt abbildbar
- Kann später als Erweiterung `LIMITED` hinzugefügt werden

---

## Consequences

### Was Agents/Developer wissen müssen

1. **Bei Ticket-Erstellung:**
   - OfferType wird von Offer an Ticket vererbt
   - Status wird server-seitig gesetzt (nicht im Frontend!)
   - SERVICE: Verfügbarkeit prüfen (ein aktives Ticket pro Buyer)

2. **Bei UI-Entwicklung:**
   - TicketDetailView muss OfferType prüfen
   - SERVICE + PENDING → Seller-Aktionen anzeigen
   - PRODUCT → Direkt Payment-Flow anzeigen

3. **Bei Protocol-Änderungen:**
   - buy-protocol.md muss SERVICE vs PRODUCT unterscheiden
   - ticket-system.md muss OfferType im Ticket-Schema dokumentieren

### Protocol-Dokumentation muss aktualisiert werden

**docs/protocols/workflows/buy-protocol.md:**
- Phase 1 Abschnitt in SERVICE/PRODUCT unterteilen
- Status-Flow Diagramm anpassen
- Seller-Akzeptanz als optionaler Schritt kennzeichnen

**docs/protocols/core/ticket-system.md:**
- OfferType im Ticket-Schema ergänzen
- Status-Übergänge für SERVICE/PRODUCT dokumentieren
- "Active API endpoints" um OfferType-Logik erweitern

### Testing im Simulator

| OfferType | TicketType | Erwarteter Status | Test-Case |
|-----------|------------|-------------------|-----------|
| SERVICE   | SMART_ORDER | PENDING | Seller muss zustimmen |
| SERVICE   | MILESTONE   | PENDING | Seller muss zustimmen |
| PRODUCT   | SMART_ORDER | IN_PROGRESS | Direkt Payment |
| PRODUCT   | MILESTONE   | IN_PROGRESS | Direkt Payment |

### Backwards-Kompatibilität

- Bestehende Tickets ohne `offerType` → Default `SERVICE` annehmen
- Migration: Alle existierenden Offers/Tickets auf `SERVICE` setzen
- Keine Breaking Changes für bestehende Clients

---

## Implementation Checklist

### Phase 1: Schema & Backend

- [ ] `OfferType` Enum zu `prisma/schema.prisma` hinzufügen
- [ ] `offerType` Feld zu `Offer` Model hinzufügen (default: SERVICE)
- [ ] `offerType` Feld zu `Ticket` Model hinzufügen
- [ ] Migration erstellen: `npx pnpm prisma migrate dev --name add_offer_type`
- [ ] `POST /api/offers`: offerType aus Request body übernehmen
- [ ] `POST /api/tickets`: 
  - SERVICE: Verfügbarkeit prüfen
  - Status-Logik: SERVICE→PENDING, PRODUCT→IN_PROGRESS
  - offerType von Offer kopieren

### Phase 2: Playground Simulator

- [ ] Typen erweitern: `MockOfferType = 'SERVICE' | 'PRODUCT'`
- [ ] `OFFER_TEMPLATES` um `offerType` Feld erweitern
- [ ] UI: OfferType Auswahl-Buttons hinzufügen
- [ ] UI: Draft Preview mit OfferType-Hinweis (Alert)
- [ ] `handleConfirmCreate`: offerType an API senden
- [ ] UI: Seller-Aktionen Alert für SERVICE-PENDING Tickets
- [ ] Log: OfferType in Flow-Logs anzeigen

### Phase 3: App Frontend

- [ ] `OfferDetailView`: Button-Text dynamisch ("Anfrage stellen" vs "Jetzt kaufen")
- [ ] `OfferDetailView`: Hinweis-Alert unter Buy Button
- [ ] `TicketDetailView`: SERVICE-PENDING Buyer-Sicht (Warte-Alert)
- [ ] `TicketDetailView`: SERVICE-PENDING Seller-Sicht (Annehmen/Ablehnen)
- [ ] `TicketDetailView`: PRODUCT Info-Alert (automatisch angenommen)
- [ ] `TicketDetailView`: Button-Logik anpassen (Invoice nur nach Akzeptanz)

### Phase 4: Protocol Documentation

- [ ] `docs/protocols/workflows/buy-protocol.md`: SERVICE/PRODUCT Flows
- [ ] `docs/protocols/core/ticket-system.md`: OfferType im Schema
- [ ] `docs/protocols/core/ticket-system.md`: Status-Flow Diagramm aktualisieren
- [ ] PDC (Protocol Decision Change) als ADR-Eintrag hinzufügen

### Phase 5: Testing

- [ ] Alle 4 Kombinationen im Simulator testen
- [ ] SERVICE: Buyer kann nicht zweites Ticket erstellen
- [ ] SERVICE: Seller kann ablehnen → Ticket CANCELLED
- [ ] PRODUCT: Ticket startet direkt in IN_PROGRESS
- [ ] Escrow-Flow für beide Typen testen

---

## References

- Protokoll: [`docs/protocols/workflows/buy-protocol.md`](../protocols/workflows/buy-protocol.md)
- Protokoll: [`docs/protocols/core/ticket-system.md`](../protocols/core/ticket-system.md)
- Protokoll: [`docs/protocols/core/escrow-protocol.md`](../protocols/core/escrow-protocol.md)
- Schema: [`prisma/schema.prisma`](../../prisma/schema.prisma)
- Backend: [`apps/server/index.ts`](../../apps/server/index.ts)
- Simulator: [`apps/playground/src/pages/Simulator.tsx`](../../apps/playground/src/pages/Simulator.tsx)
- Frontend: [`apps/colabonate-app/src/components/tickets/TicketDetailView.tsx`](../../apps/colabonate-app/src/components/tickets/TicketDetailView.tsx)
- Frontend: [`apps/colabonate-app/src/components/offers/OfferDetailView.tsx`](../../apps/colabonate-app/src/components/offers/OfferDetailView.tsx)
- Verwandte ADRs: [ADR-016](016-phase2-ticket-escrow-implementation.md), [ADR-018](018-offer-buy-now-confirmation-flow.md)

---

## Protocol Validation Notes

**Protocol-Dokumente wurden aktualisiert (2026-03-25):**

### 1. docs/protocols/workflows/buy-protocol.md ✅

**Aktualisiert mit:**
- Offer Types Tabelle (SERVICE vs PRODUCT)
- SERVICE Flow (Seller must approve)
- PRODUCT Flow (Auto-accepted)
- Phase 1 als Legacy/Deprecated markiert
- Database State (Phase 2) mit offerType
- Error Cases Tabelle erweitert

**PDC-Notiz hinzugefügt:**
```markdown
> (PDC: see ADR-021) – Added OfferType SERVICE vs PRODUCT distinction
```

---

### 2. docs/protocols/core/ticket-system.md ✅

**Aktualisiert mit:**
- `offerType` Feld im Ticket Model
- `OfferType` Enum (SERVICE, PRODUCT)
- `EscrowStatus` Enum vollständig dokumentiert
- Status Flow Diagramme für SERVICE und PRODUCT
- OfferType Behavior Tabelle

**PDC-Notiz hinzugefügt:**
```markdown
> (PDC: see ADR-021) – Added OfferType SERVICE vs PRODUCT distinction
```

---

### 3. docs/protocols/core/escrow-protocol.md

**Keine Änderungen erforderlich** – Escrow-Flow ist für beide Typen identisch nach Akzeptanz.

---

## PDC (Protocol Decision Change) Implementation ✅

**Abgeschlossen:**
1. [x] buy-protocol.md aktualisiert
2. [x] ticket-system.md aktualisiert
3. [x] PDC-Notiz in beiden Dateien hinzugefügt
4. [x] Dieses ADR als "accepted" markiert (Status: accepted)
5. [x] Bereit für Code-Implementierung

---

## Next Steps: Code Implementation

**Reihenfolge gemäß ADR-021 Implementation Checklist:**

### Phase 1: Schema & Backend
- [ ] `OfferType` Enum zu `prisma/schema.prisma` hinzufügen
- [ ] `offerType` Feld zu `Offer` Model hinzufügen (default: SERVICE)
- [ ] `offerType` Feld zu `Ticket` Model hinzufügen
- [ ] Migration erstellen: `npx pnpm prisma migrate dev --name add_offer_type`
- [ ] `POST /api/offers`: offerType aus Request body übernehmen
- [ ] `POST /api/tickets`: 
  - SERVICE: Verfügbarkeit prüfen
  - Status-Logik: SERVICE→PENDING, PRODUCT→IN_PROGRESS
  - offerType von Offer kopieren

### Phase 2: Playground Simulator
- [ ] Typen erweitern: `MockOfferType = 'SERVICE' | 'PRODUCT'`
- [ ] `OFFER_TEMPLATES` um `offerType` Feld erweitern
- [ ] UI: OfferType Auswahl-Buttons hinzufügen
- [ ] UI: Draft Preview mit OfferType-Hinweis (Alert)
- [ ] `handleConfirmCreate`: offerType an API senden
- [ ] UI: Seller-Aktionen Alert für SERVICE-PENDING Tickets
- [ ] Log: OfferType in Flow-Logs anzeigen

### Phase 3: App Frontend
- [ ] `OfferDetailView`: Button-Text dynamisch
- [ ] `OfferDetailView`: Hinweis-Alert unter Buy Button
- [ ] `TicketDetailView`: SERVICE-PENDING Buyer-Sicht
- [ ] `TicketDetailView`: SERVICE-PENDING Seller-Sicht (Annehmen/Ablehnen)
- [ ] `TicketDetailView`: PRODUCT Info-Alert
- [ ] `TicketDetailView`: Button-Logik anpassen

### Phase 4: Testing
- [ ] Alle 4 Kombinationen im Simulator testen
- [ ] SERVICE: Buyer kann nicht zweites Ticket erstellen
- [ ] SERVICE: Seller kann ablehnen → Ticket CANCELLED
- [ ] PRODUCT: Ticket startet direkt in IN_PROGRESS
- [ ] Escrow-Flow für beide Typen testen
