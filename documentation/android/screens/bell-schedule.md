# Klingelzeiten-Screen (Admin)

> Verwaltung der Stundenplan-/Klingelzeiten – aktuell nur Platzhalter.

**Route:** `bell_schedule` (Sub-Screen, Admin)
**Datei:** `ui/screens/ResourceScreens.kt` (`BellScheduleScreen`, `BellScheduleViewModel`)
**ViewModel:** `BellScheduleViewModel` (`@HiltViewModel`)
**Backing:** `AdminRepository`

---

## Aktueller Stand (WIP)

- Die API-Funktionen existieren im `UntisXApi`-Interface (siehe [API-Client](../03-api-client.md)):
  - `GET /api/school/bell-schedule`
  - `POST /api/school/bell-schedule`
  - `PUT/DELETE /api/school/bell-schedule/{id}`
- Das `AdminRepository` hat **noch keine** BellSchedule-Methoden.
- `BellScheduleViewModel.loadEntries()` ist ein **Platzhalter** (leere try/catch-Blockade, lädt nichts).
- Der Screen zeigt nur den `EmptyState` („Stundenplan-Verwaltung").

> Der Screen ist damit rein funktional **noch nicht umgesetzt** – erwartete Umsetzung: Liste der `BellScheduleEntry` (kind, label, start_time, end_time) mit Anlegen/Bearbeiten/Löschen.

## Verwandt

- [BellSchedule-DTOs](../07-datenmodelle.md)