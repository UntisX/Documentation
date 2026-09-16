# API: Schülerdaten (`/students`)

> Fehlzeiten und Noten. **Rollenbedingt gefiltert:** Schüler sehen nur ihre eigenen Datensätze, Lehrer/Admins alle.

---

## Fehlzeiten `/students/absences`

### GET /students/absences

**Eingeloggt.**

```json
[
  {
    "id": 4,
    "student_id": 12,
    "start": "2026-09-08T08:00:00Z",
    "end": "2026-09-08T10:00:00Z",
    "reason": "Krank",
    "excused": true,
    "recorded_by": 7
  }
]
```

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `id` | Number | |
| `student_id` | Number | Ergänzter Schüler |
| `start` / `end` | Timestamp | Zeitraum |
| `reason` | String/null | Grund |
| `excused` | Boolean | Entschuldigt? |
| `recorded_by` | Number | Wer hat erfasst |

> Schüler sehen NUR eigene Einträge.

### POST /students/absences

**Eingeloggt.**

```json
{
  "student_id": 12,
  "start": "2026-09-08T08:00:00Z",
  "end": "2026-09-08T10:00:00Z",
  "reason": "Arzttermin",
  "excused": true
}
```

- `recorded_by` wird vom Server auf den Aufrufer gesetzt.
- Response: `AbsenceResponse`.

### PUT /students/absences/{id}

**Eingeloggt.** `{ start?, end?, reason? }` – partiell.

### DELETE /students/absences/{id}

**Eingeloggt.** `204`.

### GET /students/absences/stats

**Eingeloggt.**

```json
{
  "total": 12,
  "excused": 9,
  "unexcused": 3,
  "late": 0
}
```

> `late` ist aktuell hart auf `0` codiert (DB kennt kein „zu spät").

---

## Noten `/students/grades`

### GET /students/grades

**Eingeloggt.**

```json
[
  {
    "id": 18,
    "student_id": 12,
    "subject_id": 1,
    "teacher_id": 7,
    "grade": 2.5,
    "weight": 2,
    "kind": "oral",
    "comment": "Gute mündliche Beteiligung",
    "created_at": "2026-09-10T10:00:00Z",
    "subject_name": "Mathematik",
    "subject_short_name": "MATH",
    "student_first_name": "Lina",
    "student_last_name": "Meyer",
    "teacher_first_name": "Max",
    "teacher_last_name": "Müller"
  }
]
```

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `grade` | Decimal | 1.0–6.0 |
| `weight` | Number | Gewichtung (Standard 1) |
| `kind` | String | `oral` / `test` / `exam` / `other` |
| `subject_name` etc. | String | JOIN-Daten aus subjects/users |

### POST /students/grades

**Eingeloggt.**

```json
{
  "student_id": 12,
  "subject_id": 1,
  "teacher_id": 7,
  "grade": 1.75,
  "weight": 1,
  "kind": "test",
  "comment": "Klassenarbeit 1"
}
```

### GET /students/grades/average

**Eingeloggt.** Query: `?student_id=12` optional – Schüler-Anfragen werden automatisch auf eigene ID begrenzt.

```json
{
  "averages": [
    { "id": 1, "name": "Mathematik", "short_name": "MATH", "average": 2.25, "count": 4 }
  ],
  "overall": {
    "overall_average": 2.5,
    "total_grades": 10
  }
}
```

| Feld | Beschreibung |
|------|--------------|
| `averages[]` | Durchschnitt je Fach |
| `average` | Gewichteter Durchschnitt |
| `count` | Anzahl Noten |
| `overall_average` | Gewichteter Gesamt-Durchschnitt |
| `total_grades` | Gesamtzahl Noten |

### PUT /students/grades/{id}

**Eingeloggt.** `{ grade?, weight?, kind?, comment? }` – partiell. Response: `GradeResponse`.

### DELETE /students/grades/{id}

**Eingeloggt.** `204`.

---

## Frontend-Verwendung

- `Grades.tsx` lädt `/students/grades`, gruppiert nach Fach und zeigt deutsche Notenfarbe:
  - 1 = grün … 6 = rot
- `Absences.tsx` zeigt Monats-Heatmap (Gruppierung nach Monat clientseitig) und CSV-Export
- Batch-Operationen (mehrere Fehlzeiten löschen) verwendet das BatchToolbar-Muster

---

## Validierung im Backend

| Feld | Regel |
|------|-------|
| `grade` | Bereich 1.0–6.0 |
| `weight` | >= 1 |
| `kind` | Whitelist `oral`/`test`/`exam`/`other` |
| `student_id` | Muss existieren |
| `start` < `end` | Muss erfüllt sein |