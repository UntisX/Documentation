# Planner-Screen (Stundenplan)

> Haupt-Bildschirm (Start-Ziel nach Login): Wochenübersicht des Stundenplans.

**Route:** `planner` (Bottom-Navigation: Tab 1 „Stundenplan")
**Datei:** `ui/screens/PlannerScreen.kt`
**ViewModel:** `PlannerViewModel` (`@HiltViewModel`)
**Backing:** `TimetableRepository`

---

## Zweck

- Zeigt den wöchentlichen Stundenplan als Wochenraster.
- Tages- und Wochenansicht umschaltbar.
- Detail-Informationen zu einzelnen Stunden.

## UI-Bausteine

| Composable | Funktion |
|-----------|----------|
| `PlannerScreen` | Haupt-UI, verwaltet Tabs (`WeekDayTab`) und Raster (`WeekGridView`) |
| `WeekDayTab` | Tab-Leiste für die Wochentage (Mo–So) |
| `WeekGridView` | Wochenraster mit Stunden-Blöcken |
| `LessonDetailDialog` | Detail-Dialog bei Klick auf eine Stunde |

## Funktionen

- **10 Perioden** (07:30–15:45) pro Tag.
- **Farbcodierte Fächer-Blöcke** (Fach-Farbe).
- **Ausgefallene Stunden:** durchgestrichen dargestellt (TimetableCancellation).
- **Klick auf Stunde:** `LessonDetailDialog` zeigt Lehrer, Raum, Woche-Typ (A/B), Zeit etc.
- **Wochen-Navigation:** Woche vor/zurück (`formatWeekLabel`, `findLessonDate`).

## Daten

- Monatsdaten: `GET /api/timetable/month-data?year=…&month=…` (gespeist vom `TimetableViewModel` über `TimetableRepository.getMonthData`).
- Wochen-Tabelle: `GET /api/timetable` (`getTimetable`).
- Custom Events (Feiertage/Ereignisse): `GET /api/custom-events?year=…&month=…`.

## Verwandt

- [TimetableRepository](../repositories/timetable-repository.md)
- [Realtime SSE (Backend)](../../08-realtime-sse.md)