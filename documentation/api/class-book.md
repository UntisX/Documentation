# Digitales Klassenbuch

> Verwaltung von Klassenbucheinträgen (Unterrichtsinhalte, Hausaufgaben, Tests) mit öffentlichem Infoscreen für die Anzeigetafel.

---

## Übersicht

| Eigenschaft | Wert |
|-------------|------|
| Auth | `Bearer <token>` (außer `/infoscreen`) |
| Rollen | Lehrer erstellen/ändern, Schüler lesen |
| Speicherung | `class_book_entries` Tabelle |

---

## Endpunkte

### GET /class-book

Klassenbucheinträge abfragen. Hinter dem `user_id`-Feld verbirgt sich automatisch die Zuordnung zum eingeloggten Lehrer (nicht in Request Body angebbar).

**Zugriff:** eingeloggt

**Query-Parameter:**

| Parameter | Typ | Pflicht | Beschreibung |
|-----------|-----|---------|-------------|
| `class_id` | int | nein | Filter nach Klasse |
| `subject_id` | int | nein | Filter nach Fach |
| `date_from` | string | nein | Startdatum (`YYYY-MM-DD`) |
| `date_to` | string | nein | Enddatum (`YYYY-MM-DD`) |
| `limit` | int | nein | Max. Ergebnisse (Default: 50) |

**Response (200):**
```json
[
  {
    "id": 1,
    "class_id": 10,
    "class_name": "5a",
    "teacher_id": 5,
    "teacher_name": "Max Mustermann",
    "subject_id": 3,
    "subject_name": "Mathematik",
    "subject_short_name": "MATH",
    "date": "2026-09-16",
    "period": 3,
    "content": "Quadratische Gleichungen: x² + bx + c = 0",
    "homework": "Übung 7a S. 45 Nr. 1-5",
    "entry_type": "lesson",
    "remarks": "Nächstes Mal: Klausurvorbereitung",
    "created_at": "2026-09-16T10:00:00",
    "updated_at": "2026-09-16T10:00:00"
  }
]
```

---

### GET /class-book/overview

Übersicht der letzten Einträge pro Klasse (für das Dashboard).

**Zugriff:** eingeloggt

**Response (200):**
```json
[
  {
    "class_id": 10,
    "class_name": "5a",
    "last_entry_date": "2026-09-16",
    "last_subject": "Mathematik",
    "entry_count": 45
  }
]
```

---

### GET /class-book/infoscreen

Öffentliche Anzeige für den Infoscreen / die Anschlagtafel. **Keine Authentifizierung nötig.**

**Zugriff:** öffentlich

**Query-Parameter:**

| Parameter | Typ | Pflicht | Beschreibung |
|-----------|-----|---------|-------------|
| `class_id` | int | nein | Nur Einträge dieser Klasse |

**Response (200):**
```json
[
  {
    "class_name": "5a",
    "subject_name": "Mathematik",
    "teacher_name": "Max Mustermann",
    "date": "2026-09-16",
    "period": 3,
    "content": "Quadratische Gleichungen",
    "homework": "Übung 7a S. 45 Nr. 1-5",
    "entry_type": "lesson"
  }
]
```

> **Hinweis:** Dieser Endpunkt ist bewusst ohne Auth konzipiert – ideal für einen Raspberry Pi mit Browser im Schulflur.

---

### POST /class-book

Neuen Klassenbucheintrag erstellen.

**Zugriff:** eingeloggt (Lehrer)

**Request Body:**
```json
{
  "class_id": 10,
  "subject_id": 3,
  "date": "2026-09-16",
  "period": 3,
  "content": "Quadratische Gleichungen",
  "homework": "Übung 7a S. 45 Nr. 1-5",
  "entry_type": "lesson",
  "remarks": ""
}
```

| Feld | Typ | Pflicht | Hinweis |
|------|-----|---------|---------|
| `class_id` | int | ja | |
| `subject_id` | int | ja | |
| `date` | string | ja | `YYYY-MM-DD` |
| `period` | int | ja | 1–12 |
| `content` | string | ja | Unterrichtsinhalt |
| `homework` | string | nein | Hausaufgabe |
| `entry_type` | string | nein | `lesson`/`homework`/`test`/`project`/`other` (Default: `lesson`) |
| `remarks` | string | nein | Anmerkungen |

**Response (201):** Erstellter Eintrag mit `id`.

---

### GET /class-book/{id}

Einzelnen Eintrag abrufen.

**Zugriff:** eingeloggt

**Response (200):** Eintrag-Objekt.

---

### PUT /class-book/{id}

Eintrag aktualisieren.

**Zugriff:** eingeloggt (nur eigene Einträge)

**Request Body:** wie POST (alle Felder optional)

**Response (200):** Aktualisierter Eintrag.

---

### DELETE /class-book/{id}

Eintrag löschen.

**Zugriff:** eingeloggt (nur eigene Einträge)

**Response:** `204 No Content`

---

## Entry-Types

| Typ | Beschreibung |
|-----|-------------|
| `lesson` | Normaler Unterricht |
| `homework` | Hausaufgabe |
| `test` | Test / Klassenarbeit |
| `project` | Projektarbeit |
| `other` | Sonstiges |

---

## Zugriffslogik

| Aktion | Rolle | Bedingung |
|--------|-------|-----------|
| Einträge lesen | eingeloggt | Lehrer: eigene Einträge; Schüler: Einträge ihrer Klasse |
| Eintrag erstellen | eingeloggt | Lehrer |
| Eintrag ändern | eingeloggt | Nur Eigentümer |
| Eintrag löschen | eingeloggt | Nur Eigentümer |
| Infoscreen lesen | öffentlich | Kein Token nötig |

---

## Fehler

| Status | Grund |
|--------|-------|
| 400 | Ungültige Felder (z.B. `period` außerhalb 1–12) |
| 401 | Kein Token / abgelaufen |
| 403 | Kein Zugriff (falscher Lehrer / Schüler nicht in der Klasse) |
| 404 | Eintrag nicht gefunden |

---

## Hinweise

- **Lehrer-Zuordnung:** Die `teacher_id` wird automatisch aus dem Token des eingeloggten Users übernommen – nicht im Request Body angebbar.
- **Keine SSE-Events:** Klassenbucheinträge lösen keine Echtzeit-Events aus. Clients müssen pollen oder die Seite neu laden.
- **Infoscreen:** Der öffentliche Endpunkt ist ideal für eine Wandanzeige im Schulflur. Er gibt immer die neuesten Einträge zurück (sortiert nach Datum/Stdunde).
