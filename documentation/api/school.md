# API: Schulressourcen (`/school`)

> Fächer, Räume, Klassen, Klingelzeiten und Schuleinstellungen.

---

## Fächer <a id="subjects"></a>`/school/subjects`

### GET /school/subjects

**Eingeloggt.** Kein Query-Parameter. Nur aktive Fächer.

```json
[
  {
    "id": 1,
    "name": "Mathematik",
    "short_name": "MATH",
    "color": "#FF5733",
    "active": true
  }
]
```

### POST /school/subjects — **Admin**

```json
{ "name": "Deutsch", "short_name": "DEU", "color": "#33A2FF" }
```

Response: `SubjectResponse` (wie oben).

### PUT /school/subjects/{id} — **Admin**

```json
{ "name": "Deutsch", "short_name": "DEU", "color": "#3380FF", "active": true }
```

### DELETE /school/subjects/{id} — **Admin**

`204` – **Soft-Deaktivierung** (setzt `active=false`, Zeile bleibt).

---

## Räume <a id="rooms"></a>`/school/rooms`

### GET /school/rooms

**Eingeloggt.**

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

### POST /school/rooms — **Admin**

```json
{
  "name": "R214",
  "building": "B",
  "capacity": 28,
  "room_type": "Fachraum",
  "floor": "3. OG",
  "accessible": false,
  "notes": "",
  "active": true
}
```

### PUT /school/rooms/{id} — **Admin**

Alle Felder wie oben änderbar.

### DELETE /school/rooms/{id} — **Admin**

`204` – **Soft-Deaktivierung** (`active=false`).

---

## Klassen <a id="classes"></a>`/school/classes`

### GET /school/classes

**Eingeloggt.**

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

> `student_count` wird per JOIN berechnet. `room`/`class_teacher` sind Strings, mit den angelegten Klassen/Schüler-Namen befüllt.

### POST /school/classes — **Admin**

```json
{
  "name": "8b",
  "grade_level": "8",
  "room": "R101",
  "class_teacher": "max.mueller",
  "class_deputy_teacher": "eva.braun"
}
```

### PUT /school/classes/{id} — **Admin**

Alle Felder änderbar.

### DELETE /school/classes/{id} — **Admin**

`204` – **Hard-Delete** (echtes `DELETE FROM classes` – Referenzen vorher prüfen!).

---

## Klingelzeiten <a id="bell-schedule"></a>`/school/bell-schedule`

### GET /school/bell-schedule

**Eingeloggt.** Sortiert nach `start_time`.

```json
[
  { "id": 1, "kind": "lesson", "label": "1. Stunde", "start_time": "07:30", "end_time": "08:15" },
  { "id": 2, "kind": "break",  "label": "Pause",     "start_time": "08:15", "end_time": "08:20" }
]
```

| Feld | Beschreibung |
|------|--------------|
| `kind` | z.B. `lesson` / `break` |
| `label` | Anzeige-Name |
| `start_time` / `end_time` | `HH:MM` 24h |

### POST /school/bell-schedule — **Admin**

```json
{ "kind": "lesson", "label": "2. Stunde", "start_time": "08:20", "end_time": "09:05" }
```

### PUT /school/bell-schedule/{id} — **Admin**

Gleiche Felder.

### DELETE /school/bell-schedule/{id} — **Admin**

`204` – **Hard-Delete**.

> **Hinweis:** Die `bell_schedule`-Tabelle wird aktuell NICHT für die Zeitberechnung von Stunden verwendet – die 12 Periodenzeiten sind hart codiert (07:30–15:45). Die Tabelle dient der Verwaltung/Anzeige.

---

## Schuleinstellungen <a id="settings"></a>

### GET /school/settings

**Eingeloggt.**

```json
{
  "school_name": "Gymnasium Musterstadt",
  "school_logo": "data:image/png;base64,iVBORw0KGgo...",
  "timezone": "Europe/Berlin",
  "address": "Musterstraße 1, 12345 Musterstadt",
  "phone": "0123 456789",
  "email": "info@schule.de"
}
```

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `school_name` | string | Name der Schule (wird für Branding/Titel verwendet) |
| `school_logo` | string | Schul-Logo als **Data-URL** (z. B. `data:image/png;base64,...`), leer = kein Logo (Migration 050) |
| `timezone` | string | Zeitzone |
| `address` / `phone` / `email` | string | Kontaktdaten |

### GET /schools/self

**Alias** für `GET /school/settings` – gleiche Response.

### PATCH /school/settings — **Admin**

Partielles Update – nur gesetzte Felder werden geändert:

```json
{
  "school_name": "Gymnasium Musterstadt",
  "school_logo": "data:image/png;base64,iVBORw0KGgo...",
  "timezone": "Europe/Berlin"
}
```

> `school_logo` löschen = Wert `""` (leerer String) senden. Das offizielle Frontend nutzt den Wert u. a. für `BrandLogo` + Favicon (Client-seitig gecacht in localStorage unter `untisx_school_logo`).

### Response

Gleiche Struktur wie GET (alle Felder zurückgegeben).

---

## Hinweise für eigene Implementierungen

| Aspekt | Wert |
|--------|------|
| Soft-Deletes | `active=false`, Antworten filtern `active=true` |
| Klassen-Löschung | Hard-Delete – Achtung mit Fremdschlüsseln |
| Identität | `school_settings` ist eine einzelne Zeile (`id=1`) |
| Farbformat | Hex `#RRGGBB` (Validierung über `validate_color`) |
| Fehler | `409` bei doppeltem Klassennamen, `422` bei ungültiger Farbe |