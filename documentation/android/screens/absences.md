# Fehlstunden-Screen (Absences)

> Ansicht der Fehlzeiten eines Schülers inkl. Statistik.

**Route:** `absences` (Bottom-Navigation: Tab 2 „Fehlstunden")
**Datei:** `ui/screens/AbsencesScreen.kt`
**ViewModel:** `AbsencesViewModel` (`@HiltViewModel`)
**Backing:** `AbsenceRepository`

---

## UI-Bausteine

| Composable | Funktion |
|-----------|----------|
| `AbsencesScreen` | Haupt-UI: Liste der Fehlzeiten + Statistik |
| `AbsenceCard` | Einzelne Fehlzeit (Zeitraum, Fach, Grund, entschuldigt-Badge) |
| `CreateAbsenceDialog` | Neuen Fehlzeiten-Eintrag anlegen |

## Funktionen

- **Fehlzeiten-Liste** mit Karten (`AbsenceCard`).
- **Statistik-Karten** oben: Gesamt / Entschuldigt / Unentschuldigt / Verspätet (aus `AbsenceStats`).
- Swipe-/Button zum Löschen einzelner Einträge (`onDelete` → `delete`).

## Daten

- `GET /api/students/absences?studentId=…` (Schüler-ID optional, Standard = eigener Account).
- `GET /api/students/absences/stats?studentId=…`.

> **WIP:** `CreateAbsenceDialog` zeigt zwar den Dialog (Fach, Zeitraum, Grund, entschuldigt), der „Bestätigen"-Button ruft aber die API **nicht** auf.

## Verwandt

- [AbsenceRepository](../repositories/absence-repository.md)
- [Absence-DTOs](../07-datenmodelle.md)