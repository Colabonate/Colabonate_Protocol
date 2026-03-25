# 020 – Database Wipe Function for Protocol Simulator

**Status:** accepted
**Date:** 2026-03-25
**Affects:** `apps/server/index.ts`, `apps/playground/src/pages/Simulator.tsx`

---

## Context

Da Colabonate sich noch in der Entwicklung befindet und keine Live-Anbindung existiert, werden im Protocol Simulator (Playground) kontinuierlich Test-Tickets erstellt. Bisher gab es keine einfache Möglichkeit, diese Test-Daten aus der Datenbank zu entfernen.

**Problem:**
- Beim Testen von Ticket-Flows (SMART_ORDER, MILESTONE) sammeln sich viele Test-Tickets an
- Die DB kann nur manuell über SQLite-Commands zurückgesetzt werden
- Kein "Reset"-Button für schnelle Iterationen während der Entwicklung

---

## Decision

Eine **Wipe-Funktion** wird im Protocol Simulator hinzugefügt:

### Backend: Neuer API Endpoint

```typescript
// Wipe All Tickets Endpoint (For Simulator – includes milestones via cascade)
app.delete('/api/tickets/wipe', async (req, res) => {
  try {
    console.log('[WIPE] Starting database wipe...')
    // Disable foreign key checks for SQLite
    await prisma.$executeRawUnsafe('PRAGMA foreign_keys = OFF')
    
    // Delete in correct order for SQLite foreign keys
    await prisma.milestone.deleteMany({})
    console.log('[WIPE] Milestones deleted')
    await prisma.ticket.deleteMany({})
    console.log('[WIPE] Tickets deleted')
    await prisma.review.deleteMany({})
    console.log('[WIPE] Reviews deleted')
    await prisma.offerMilestone.deleteMany({})
    console.log('[WIPE] OfferMilestones deleted')
    await prisma.offer.deleteMany({})
    console.log('[WIPE] Offers deleted')
    
    // Re-enable foreign key checks
    await prisma.$executeRawUnsafe('PRAGMA foreign_keys = ON')
    console.log('[WIPE] Foreign keys re-enabled')
    
    res.json({ success: true, message: 'All tickets and offers wiped from database' })
  } catch (error) {
    // Ensure foreign keys are re-enabled even on error
    await prisma.$executeRawUnsafe('PRAGMA foreign_keys = ON').catch(() => {})
    console.error('Failed to wipe tickets:', error)
    res.status(500).json({ error: 'Failed to wipe database', details: error instanceof Error ? error.message : 'Unknown error' })
  }
})
```

**Lösch-Reihenfolge (wegen Foreign Keys):**
1. `Milestone` (Ticket-Milestones)
2. `Ticket` (Alle Tickets)
3. `Review` (Ticket-Reviews)
4. `OfferMilestone` (Offer-Milestones)
5. `Offer` (Alle Angebote)

### Frontend: UI-Button im Simulator

**Position:** Oberhalb der Ticket-Erstellen-Panels, rechtsbündig

**Design:**
- Roter outlined Button mit Warnsymbol ⚠️
- Beschriftung: "Database Wipe (Alle Tickets löschen)"
- Deaktiviert wenn keine Tickets vorhanden oder Loading-Zustand

**Sicherheit:** `window.confirm()`-Dialog vor Ausführung

```typescript
async function handleWipeDatabase() {
  if (!window.confirm('⚠️ Alle Tickets und Offers aus der DB löschen? Diese Aktion kann nicht rückgängig gemacht werden!')) {
    return
  }
  setLoading(true)
  try {
    const res = await fetch(`${API}/api/tickets/wipe`, { method: 'DELETE' })
    if (!res.ok) throw new Error('Wipe failed')
    await fetchAllTickets()
    setSelectedTicketId('')
    setSelectedMilestoneId('')
    log('[WIPE] Database wiped successfully – all tickets and offers removed')
  } catch (e: any) {
    log(`[WIPE] Error: ${e?.message || 'Unknown'}`)
  } finally {
    setLoading(false)
  }
}
```

---

## Rejected Alternatives

### Alternative A: Nur Tickets löschen, Offers behalten

**Abgelehnt weil:** Offers ohne Tickets sind nutzlos; kompletter Wipe ist sauberer.

### Alternative B: Wipe ohne Confirmation-Dialog

**Abgelehnt weil:** Versehentliches Löschen wäre zu riskant.

### Alternative C: Separate Wipe-Buttons für Tickets/Offers

**Abgelehnt weil:** Komplexität unnötig; Entwickler wollen meist alles auf einmal löschen.

### Alternative D: Soft-Delete mit `isTest` Flag

**Abgelehnt weil:** Overengineering für Test-Umgebung; echte DB bleibt unberührt.

---

## Consequences

### Positive Effekte

- **Schnelleres Iterieren:** Entwickler können DB mit einem Klick zurücksetzen
- **Sauberer Zustand:** Nach Wipe ist die DB leer, keine verwaisten Datensätze
- **Sicherheit:** Confirmation-Dialog verhindert versehentliches Löschen
- **Nur Development:** Diese Funktion ist nur im Playground verfügbar, nicht in der Production-App

### Logging

Bei erfolgreichem Wipe wird in der Event Console angezeigt:
```
[WIPE] Starting database wipe...
[WIPE] Milestones deleted
[WIPE] Tickets deleted
[WIPE] Reviews deleted
[WIPE] OfferMilestones deleted
[WIPE] Offers deleted
[WIPE] Foreign keys re-enabled
[WIPE] Database wiped successfully – all tickets and offers removed
```

---

## Implementation Details

### Server (`apps/server/index.ts`)

```typescript
app.delete('/api/tickets/wipe', async (req, res) => {
  try {
    console.log('[WIPE] Starting database wipe...')
    await prisma.$executeRawUnsafe('PRAGMA foreign_keys = OFF')
    
    await prisma.milestone.deleteMany({})
    await prisma.ticket.deleteMany({})
    await prisma.review.deleteMany({})
    await prisma.offerMilestone.deleteMany({})
    await prisma.offer.deleteMany({})
    
    await prisma.$executeRawUnsafe('PRAGMA foreign_keys = ON')
    
    res.json({ success: true, message: 'All tickets and offers wiped from database' })
  } catch (error) {
    await prisma.$executeRawUnsafe('PRAGMA foreign_keys = ON').catch(() => {})
    console.error('Failed to wipe tickets:', error)
    res.status(500).json({ error: 'Failed to wipe database', details: error instanceof Error ? error.message : 'Unknown error' })
  }
})
```

### Simulator (`apps/playground/src/pages/Simulator.tsx`)

```typescript
async function handleWipeDatabase() {
  if (!window.confirm('⚠️ Alle Tickets und Offers aus der DB löschen? Diese Aktion kann nicht rückgängig gemacht werden!')) {
    return
  }
  setLoading(true)
  try {
    const res = await fetch(`${API}/api/tickets/wipe`, { method: 'DELETE' })
    if (!res.ok) throw new Error('Wipe failed')
    await fetchAllTickets()
    setSelectedTicketId('')
    setSelectedMilestoneId('')
    log('[WIPE] Database wiped successfully – all tickets and offers removed')
  } catch (e: any) {
    log(`[WIPE] Error: ${e?.message || 'Unknown'}`)
  } finally {
    setLoading(false)
  }
}

// UI Render
<Box sx={{ display: 'flex', justifyContent: 'flex-end', mb: 2 }}>
  <Button
    size="small"
    variant="outlined"
    color="error"
    onClick={handleWipeDatabase}
    disabled={loading || liveTickets.length === 0}
    startIcon={<ReportProblemIcon />}
  >
    ⚠️ Database Wipe (Alle Tickets löschen)
  </Button>
</Box>
```

---

## Usage

### Via UI
1. Playground öffnen (Port 5174)
2. "Protocol Simulator" → "Ticket Flow Simulator"
3. "⚠️ Database Wipe" Button klicken (oben rechts)
4. Confirmation-Dialog bestätigen

### Via API
```bash
curl -X DELETE http://localhost:4001/api/tickets/wipe
```

**Response:**
```json
{
  "success": true,
  "message": "All tickets and offers wiped from database"
}
```

---

## Note

**Nur für Development/Testing.** Diese Funktion ist nur im Playground verfügbar, nicht in der Production-App. Die Production-Datenbank wird nicht beeinflusst.

---

## References

- Simulator Page: [`apps/playground/src/pages/Simulator.tsx`](../../apps/playground/src/pages/Simulator.tsx)
- Server Routes: [`apps/server/index.ts`](../../apps/server/index.ts)
- Verwandte ADRs: [ADR-019](019-protocol-simulator-cleanup.md) – Protocol Simulator Cleanup
