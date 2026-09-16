# TimetableRepository

> Stundenplan-Daten für den Planner-Screen: Wochen-/Monatsdaten und Custom Events.

**Datei:** `data/repository/TimetableRepository.kt`
**Singleton:** ja (`@Singleton`)

---

## Methoden

### `getMonthData(year: Int, month: Int): ApiResult<TimetableMonthData>`

- `GET /api/timetable/month-data?year=…&month=…`
- Liefert die Monatsansicht (Liste der Stunden + verfügbare Monate/Jahre).
- Grundlage für den Planner.

### `getTimetable(): ApiResult<List<TimetableEntry>>`

- `GET /api/timetable`
- Komplette Wochen-Tabelle aller Timetable-Einträge.

### `createEntry(request: TimetableEntryCreateRequest): ApiResult<TimetableEntry>`

- `POST /api/timetable`

### `deleteEntry(id: Long): ApiResult<Unit>`

- `DELETE /api/timetable/{id}`

### `getCustomEvents(year: Int, month: Int): ApiResult<List<CustomEvent>>`

- `GET /api/custom-events?year=…&month=…`

### `createCustomEvent(request: CustomEventCreateRequest): ApiResult<CustomEvent>`

- `POST /api/custom-events`

### `deleteCustomEvent(id: Long): ApiResult<Unit>`

- `DELETE /api/custom-events/{id}`

> Hinweis: Anlegen/Ändern von Stundenplan-Einträgen (`createEntry`) ist im Interface vorhanden, wird aber aktuell nicht von admin-Screens genutzt (Timetable wird v. a. gelesen).

## Verwandt

- [Planner-Screen](../screens/planner.md)