# API: Kalender (`/custom-events`)

> Eigene/Einträge von Benutzern im Monatskalender mit Wiederholung und Farbe.

---

## GET /custom-events

**Eingeloggt.** Nur die Ereignisse des aufrufenden Benutzers.

Query: `?year=2026&month=10` (optional, sonst aktueller Monat).

### Response (200)

```json
[
  {
    "id": 7,
    "user_id": 1,
    "title": "Elternabend",
    "description": "Beginn im Lehrerzimmer",
    "date": "2026-10-07",
    "time_start": "18:00",
    "time_end": "20:00",
    "color": "#4A90D9",
    "recurrence": "none"
  }
]
```

| Feld | Werte |
|------|-------|
| `recurrence` | `none` / `daily` / `weekly` / `biweekly` / `monthly` / `yearly` |
| `color` | Hex `#RRGGBB` |

---

## POST /custom-events

**Eingeloggt.**

```json
{
  "title": "Klausurwoche",
  "description": "Auswertung am Freitag",
  "date": "2026-11-03",
  "time_start": "09:00",
  "time_end": "10:00",
  "color": "#E74C3C",
  "recurrence": "yearly"
}
```

- `user_id` wird aus dem Token gesetzt (nicht im Body!).
- `color` Default `#4A90D9`, `recurrence` Default `none`.

### Response (201)

`CustomEventResponse` (wie oben).

---

## PUT /custom-events/{id}

**Eingeloggt.** Nur der **Eigentümer**. Partiell:

```json
{
  "title": "Klausurwoche (verschoben)",
  "date": "2026-11-10"
}
```

### Response `200`

---

## DELETE /custom-events/{id}

**Eingeloggt.** Nur der **Eigentümer**. `204`.

---

## Konfliktprüfung

Das Frontend führt vor dem Speichern eine Client-seitige Konfliktprüfung durch (überschneidende Zeiten am selben Tag). Die API selbst liefert keine Konflikt-Fehler – ein eigenes Frontend sollte das selbst machen.

---

## Echtzeit

Custom-Events lösen KEINE SSE-Events aus – nur Timetable- und Chat-Aktionen pushen. Dein Frontend sollte nach Änderungen selbst neu laden.