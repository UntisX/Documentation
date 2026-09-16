# API: Externe Schnittstelle (`/external`)

> Der **offizielle Schreib-arme (read-only)** API-Zugang für Drittanbieter und eigene Programme – authentifiziert per `X-Api-Key` statt Bearer-Token.

---

## Authentifizierung

```
GET /external/<ressource>
Header: X-Api-Key: untisx_<base64url>
```

Einen Key bekommst du als Admin über `POST /api-keys` (siehe [apikeys](api/apikeys.md)). Scopes müssen die gewünschten `/external/*`-Pfade enthalten.

---

## GET /external/classes

### Response (200)

```json
[
  {
    "id": 3,
    "name": "7a",
    "grade_level": "7",
    "room": "R202",
    "class_teacher": "Max Müller",
    "class_deputy_teacher": "Eva Braun",
    "student_count": 28
  }
]
```

---

## GET /external/subjects

### Response (200)

```json
[
  { "id": 1, "name": "Mathematik", "short_name": "MATH", "color": "#FF5733", "active": true }
]
```

---

## GET /external/rooms

### Response (200)

```json
[
  {
    "id": 1,
    "name": "R202",
    "building": "A",
    "capacity": 30,
    "room_type": "Klassenzimmer",
    "floor": "2. OG",
    "accessible": true,
    "notes": "Beamer vorhanden",
    "active": true
  }
]
```

---

## GET /external/bell-schedule

### Response (200)

```json
[
  { "id": 1, "kind": "lesson", "label": "1. Stunde", "start_time": "07:30", "end_time": "08:15" }
]
```

---

## Einsatzbeispiele

| Zweck | Endpunkt |
|-------|----------|
| Schild/Anzeige in der Schule | `GET /external/bell-schedule` |
| Homepage mit vereinfachter Stundenplan-Galerie | `GET /external/classes` + `/external/subjects` |
| Raumbelegungs-Anzeige | `GET /external/rooms` |
| Verifikation/Abgleich externer Systeme | alle `/external/*` |

---

## Sicherheits-Hinweise

- `/external/*` sind **ausschließlich GET** (Lesen). Zum Schreiben musst du dich als Benutzer einloggen (Bearer).
- Die zulässigen Pfade pro Key-Scope werden serverseitig geprüft → 403 bei Überschreitung.
- Vor Umstellung eines Keys auf `write`/`full` Scope-Listen minimal halten!