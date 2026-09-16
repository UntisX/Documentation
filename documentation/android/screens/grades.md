# Noten-Screen (Grades)

> Notenübersicht mit Fach-Durchschnitten und Gesamtdurchschnitt.

**Route:** `grades` (Sub-Screen, über Profil erreichbar)
**Datei:** `ui/screens/GradesScreen.kt`
**ViewModel:** `GradesViewModel` (`@HiltViewModel`)
**Backing:** `GradeRepository`

---

## UI-Bausteine

| Composable | Funktion |
|-----------|----------|
| `GradesScreen` | Haupt-UI: Liste + Durchschnitts-Karten |
| `SubjectAverageCard` | Fach-Durchschnitt mit farbigem Noten-Badge |
| `GradeCard` | Einzelne Note (Fach, Wert, Art `getGradeKindLabel`, Datum) |
| `getGradeColor(grade)` | Farbkodierung der Note (grün/gelb/rot) |

## Funktionen

- **Noten-Liste** (`GradeCard` je Eintrag, Löschen-Button optional).
- **Fach-Durchschnitte** (`SubjectAverage`) als Karten.
- **Gesamtdurchschnitt** (`OverallAverage`).
- Delete-Dialog `onDelete` → `GradeRepository.delete(id)`.

## Daten

- `GET /api/students/grades?studentId=…` bzw. `GET /api/students/grades/average?studentId=…`.

> **WIP:** Der Delete-Dialog schließt sich, die Note wird aktuell aber **nicht** wirklich gelöscht (kein echter API-Call bzw. UI-only).

## Verwandt

- [GradeRepository](../repositories/grade-repository.md)
- [Grade-DTOs](../07-datenmodelle.md)