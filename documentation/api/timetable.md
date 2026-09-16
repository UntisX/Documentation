# API: Stundenplan & Organisation (`/timetable`)

> Stundenplan, Ausfälle, Vertretungen, Lehrer-Abwesenheiten, Hausaufgaben sowie die kombinierten Ansichten (Vertretungsplan, Monatsdaten, Klassenplan).

---

## Stundenplan `/timetable`

### GET /timetable

**Eingeloggt.**

```json
[
  {
    "id": 5,
    "subject_id": 1,
    "teacher_id": 7,
    "room_id": 1,
    "class_id": 3,
    "first_started_at": "2026-09-01",
    "first_ended_at": "2026-09-01",
    "repeats_every_days": 7,
    "valid_from": "2026-09-01",
    "valid_until": "2027-06-30",
    "day_of_week": 2,
    "period": 1,
    "period_count": 1,
    "week_type": null,
    "subject_name": "Mathematik",
    "subject_short_name": "MATH",
    "color": "#FF5733",
    "teacher_first_name": "Max",
    "teacher_last_name": "Müller",
    "room": "R202",
    "class_name": "7a"
  }
]
```

**Berechnete Felder:**

| Feld | Berechnung |
|------|-----------|
| `day_of_week` | Aus `first_started_at` (0=Sonntag … 6=Samstag) |
| `period` | Start-Zeit → 1.000 Periode (hardcodierte Zeiten) |
| `period_count` | Stundenzahl aus Start/Ende |
| `week_type` | `odd`/`even`/`null` (ungerade/gerade Woche) |

**Die hardcodierten 12 Periodenzeiten:**

| Periode | Start | Ende |
|---------|-------|------|
| 1 | 07:30 | 08:15 |
| 2 | 08:15 | 09:00 |
| 3 | 09:00 | 09:45 |
| 4 | 09:45 | 10:30 |
| 5 | 10:30 | 11:15 |
| 6 | 11:15 | 12:00 |
| 7 | 12:00 | 12:45 |
| 8 | 12:45 | 13:30 |
| 9 | 13:30 | 14:15 |
| 10 | 14:15 | 15:00 |
| 11 | 15:00 | 15:45 |
| 12 | 15:45 | 16:30 |

### POST /timetable — **Admin**

Wiederkehrender Eintrag:

```json
{
  "subject_id": 1,
  "teacher_id": 7,
  "room_id": 1,
  "class_id": 3,
  "first_started_at": "2026-09-01",
  "first_ended_at": "2026-09-01",
  "repeats_every_days": 7,
  "valid_until": "2027-06-30"
}
```

| Feld | Pflicht | Beschreibung |
|------|---------|--------------|
| `subject_id` | ✔ | |
| `teacher_id` | – | (nullable) |
| `room_id` | – | (nullable) |
| `class_id` | – | |
| `first_started_at` | ✔ | Datum der Wiederholung & erster Zeitslot |
| `first_ended_at` | ✔ | Ende der ersten Stunde (für Zeitslot-Berechnung) |
| `repeats_every_days` | – | Standard 7 |
| `valid_until` | – | Standard 2099-12-31 |

### PUT /timetable/{id} — **Admin**

Gleiche Felder, partiell möglich.

### DELETE /timetable/{id} — **Admin**

`204`.

---

## Ausfälle

### GET /timetable/cancellations

**Eingeloggt.** Query: `?date=YYYY-MM-DD` optional.

```json
[
  {
    "id": 9,
    "timetable_entry_id": 5,
    "date": "2026-09-15",
    "reason": "Lehrer krank",
    "cancelled_by": 1,
    "created_at": "2026-09-13T10:00:00Z"
  }
]
```

### POST /timetable/{id}/cancel

**Eingeloggt.** Eine Stunde an einem bestimmten Datum ausfallen lassen:

```json
{ "date": "2026-09-15", "reason": "Fortbildung der Lehrkraft" }
```

Response: `CancellationResponse`, zusätzlich wird ein Echtzeit-Event `kind: cancellation` auf `/events` ausgelöst.

### DELETE /timetable/{id}/cancel (Route)

Nimmt den Ausfall an `?date=YYYY-MM-DD` zurück (Route im Backend vorhanden).

---

## Vertretungen

### GET /timetable/substitutions

**Eingeloggt.** Query: `?date=` optional. Nur aktive (`status=active`).

```json
[
  {
    "id": 3,
    "timetable_entry_id": 5,
    "date": "2026-09-15",
    "original_teacher_id": 7,
    "substitute_teacher_id": 9,
    "original_subject_id": 1,
    "substitute_subject_id": 1,
    "room": "R214",
    "reason": "Vertretung für Müller",
    "type": "teacher",
    "status": "active",
    "created_by": 1,
    "created_at": "2026-09-13T10:00:00Z"
  }
]
```

| Feld | Beschreibung |
|------|--------------|
| `type` | `teacher` / `room` / `subject` / `merge` |
| `status` | `active` / `cancelled` |
| `original_*` / `substitute_*` | Wer/Was ersetzt wird |

### POST /timetable/substitutions

**Eingeloggt.** Body:

```json
{
  "timetable_entry_id": 5,
  "date": "2026-09-15",
  "original_teacher_id": 7,
  "substitute_teacher_id": 9,
  "room": "R214",
  "reason": "Dienstbesprechung",
  "type": "teacher"
}
```

`original_subject_id`/`substitute_subject_id` optional. Erzeugt Echtzeit-Event `kind: substitution`.

### DELETE /timetable/substitutions/{id}

**Eingeloggt.** `204` – **storniert die Vertretung** (setzt `status='cancelled'`), löscht die Zeile NICHT.

---

## Lehrer-Abwesenheiten

### GET /timetable/teacher-absences

**Eingeloggt.** Query: `?date=` optional.

```json
[
  { "id": 2, "teacher_id": 7, "date": "2026-09-16", "reason": "Kongress", "created_at": "..." }
]
```

### POST /timetable/teacher-absences

**Eingeloggt.**

```json
{ "teacher_id": 7, "date": "2026-09-16", "reason": "Kongress" }
```

### DELETE /timetable/teacher-absences/{id}

**Eingeloggt.** `204`.

---

## Hausaufgaben

### GET /timetable/homework

**Eingeloggt.**

```json
[
  {
    "id": 3,
    "subject_id": 1,
    "teacher_id": 7,
    "title": "Übungsblatt 5",
    "content": "Aufgaben 1–8",
    "date": "2026-09-20",
    "date_until": null,
    "subject_name": "Mathematik",
    "subject_short_name": "MATH",
    "teacher_first_name": "Max",
    "teacher_last_name": "Müller"
  }
]
```

### POST /timetable/homework — **Lehrer/Admin**

```json
{
  "subject_id": 1,
  "title": "Vokabeltest vorbereiten",
  "content": "Unit 4, p. 55",
  "date": "2026-09-22",
  "date_until": null
}
```

- `teacher_id` wird vom Server aus dem Token gesetzt.
- Erzeugt Echtzeit-Event `kind: homework`.

### PUT /timetable/homework/{id} — **Lehrer/Admin**

`{ title?, content?, date_until? }` – partiell.

### DELETE /timetable/homework/{id} — **Lehrer/Admin**

`204`.

---

## Kombinierte Ansichten

### GET /timetable/vertretungsplan

**Eingeloggt.** Query: `?date=YYYY-MM-DD` (sonst Heute).

Liefert einen kompletten Tagesplan als eine Response:

```json
{
  "day_of_week": 2,
  "week_type": null,
  "cancellations": [ ... ],
  "substitutions": [ ... ],
  "teacher_absences": [ ... ]
}
```

> Perfekt für Vertretungsplan-Seiten – kein stückweises Zusammenbauen nötig.

### GET /timetable/month-data

**Eingeloggt.** Query: `?year=2026&month=9` (optional).

```json
{
  "entries": [ ... ],
  "cancellations": [ ... ],
  "homework": [ ... ],
  "week_cache": {}
}
```

> Liefert alle Daten eines Monats für die Monatsansicht im Frontend. `week_cache` ist aktuell ein leeres Objekt.

### GET /timetable/class-plan

**Eingeloggt.** Query: `?class_id=3` (optional).

```json
{
  "class_id": 3,
  "class_name": "7a",
  "entries": [
    {
      "id": 5,
      "subject_id": 1,
      "period": 1,
      "period_count": 1,
      "day_of_week": 2,
      "subject_name": "Mathematik",
      "subject_short_name": "MATH",
      "color": "#FF5733",
      "teacher_first_name": "Max",
      "teacher_last_name": "Müller",
      "room": "R202"
    }
  ]
}
```

> Ideal für die Stundenplan-Anzeige einer Klasse.

---

## Realtime-Integration

Alle schreibenden Timetable-Endpunkte stoßen SSE-Events aus:

| Endpunkt | SSE-kind |
|----------|----------|
| `POST /timetable/substitutions` | `substitution` |
| `POST /timetable/{id}/cancel` | `cancellation` |
| `POST /timetable/homework` | `homework` |
| `POST /chats/.../messages` (Beispiel) | `message` |