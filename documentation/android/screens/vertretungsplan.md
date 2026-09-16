# Vertretungsplan-Screen

> Tagesansicht des Vertretungsplans mit Vertretungen und Ausfällen.

**Route:** `vertretungsplan` (Sub-Screen, Admin)
**Datei:** `ui/screens/VertretungsplanScreen.kt`
**ViewModel:** `VertretungsplanViewModel` (`@HiltViewModel`)
**Backing:** `AdminRepository`

---

## ViewModel-Zustand

| State | Typ | Beschreibung |
|-------|-----|--------------|
| `selectedDate` | `StateFlow<LocalDate>` | Aktuell gewähltes Datum (Default heute) |
| `vertretung` | `StateFlow<VertretungsplanResponse?>` | Daten (null bei Fehler) |
| `isLoading` | `StateFlow<Boolean>` | Ladeindikator |
| `selectedTab` | `StateFlow<Int>` | Aktiver Reiter (Ausfälle / Vertretungen …) |

## Funktionen

- Datum wählen (`selectDate`), vor/zurück blättern (`previousDay`/`nextDay`).
- Beim Datumswechsel automatisch neu laden.
- Zwei Ansichten (Tabs) für:
  - `VertretungCancellation` (Ausfälle)
  - `VertretungSubstitution` (Vertretungen)
- Fehler: bei `ApiResult.Error` wird `vertretung = null` gesetzt (leere Sicht).

## Daten

- `GET /api/timetable/vertretungsplan?date=YYYY-MM-DD` (`adminRepository.getVertretungsplan(date)`).

## Verwandt

- [AdminRepository](../repositories/admin-repository.md) (getVertretungsplan, createCancellation, deleteCancellation)
- [Vertretung-DTOs](../07-datenmodelle.md)