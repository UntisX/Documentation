# Audit-Protokoll-Screen

> Ansicht des administrativen Audit-Protokolls mit zwei Darstellungsmodi.

**Route:** `audit_log` (Sub-Screen, Admin)
**Datei:** `ui/screens/AuditLogScreen.kt`
**ViewModel:** `AuditLogViewModel` (`@HiltViewModel`)
**Backing:** `AdminRepository`

---

## UI-Bausteine

| Composable | Funktion |
|-----------|----------|
| `AuditLogScreen` | Haupt-UI mit Umschalter |
| `TimelineView` | Chronologische Timeline-Darstellung der Einträge |
| `TableView` | Tabellarische Darstellung |
| `getActionColor(action)` | Farbkodierung je Aktion (CREATE/UPDATE/DELETE/…) |

## Funktionen

- Umschaltbar zwischen **Timeline**- und **Tabellen**-Ansicht.
- Listet `AuditEntry`-Einträge (Aktion, Entität, User, Zeit, Details).
- Farbcodierung der Aktionen für schnelle Erfassung.

## Daten

- `GET /api/administration/audit?limit=100&offset=0&action=…` (`adminRepository.getAuditLog`).
- `AuditLogResponse { entries: List<AuditEntry>, total: Long }`.

## Verwandt

- [AdminRepository](../repositories/admin-repository.md) (getAuditLog)
- [Audit-DTOs](../07-datenmodelle.md)