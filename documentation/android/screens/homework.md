# Hausaufgaben-Screen (Homework)

> Liste und Verwaltung von Hausaufgaben.

**Route:** `homework` (Bottom-Navigation: Tab 3 „Hausaufgaben")
**Datei:** `ui/screens/HomeworkScreen.kt`
**ViewModel:** `HomeworkViewModel` (`@HiltViewModel`)
**Backing:** `HomeworkRepository`

---

## UI-Bausteine

| Composable | Funktion |
|-----------|----------|
| `HomeworkScreen` | Haupt-UI: gefilterte Liste + Erstellen-Button |
| `HomeworkCard` | Einzelne Hausaufgabe mit Status-Badge |
| `CreateHomeworkDialog` | Neue Hausaufgabe anlegen |

## Funktionen

- **Hausaufgaben-Liste** mit Karten (`HomeworkCard`).
- **Status-Badges:** Überfällig / Heute / Nächste Woche / OK (berechnet aus `dueDate`).
- **Löschen** über die Karte (`onDelete` → `delete`).
- **Filter** nach Fach möglich (`subjectId`-Parameter).

## Daten

- `GET /api/timetable/homework?subjectId=…`.

> **WIP:** `CreateHomeworkDialog` ist zwar vorhanden, die eigentliche Erstellung aber noch `// TODO` (kein API-Call).

## Verwandt

- [HomeworkRepository](../repositories/homework-repository.md)
- [Homework-DTOs](../07-datenmodelle.md)