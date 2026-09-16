# Buchungssystem (Resources & Bookings)

> Verwaltung von buchbaren Ressourcen (Computer-Räume, Beamer, Tablets etc.) und Zeitbuchungen mit Konfliktprüfung.

---

## Übersicht

| Eigenschaft | Wert |
|-------------|------|
| Auth | `Bearer <token>` |
| Rollen | Alle eingeloggten User können buchen |
| Konfliktprüfung | Expliziter Endpoint + implizit beim Erstellen |

---

## Ressourcen

### GET /resources

Alle verfügbaren Ressourcen auflisten.

**Zugriff:** eingeloggt

**Response (200):**
```json
[
  {
    "id": 1,
    "name": "IT-Raum 101",
    "resource_type": "computer_room",
    "location": "Gebäude A, 1. OG",
    "description": "30 Arbeitsplätze mit Windows-PCs",
    "capacity": 30,
    "active": true,
    "created_at": "2026-08-01T10:00:00"
  }
]
```

---

### POST /resources

Neue Ressource anlegen.

**Zugriff:** eingeloggt

**Request Body:**
```json
{
  "name": "Tablet-Klasse",
  "resource_type": "tablet",
  "location": "Gebäude B, Raum 205",
  "description": "20 iPads mit Stiften",
  "capacity": 20
}
```

| Feld | Typ | Pflicht | Hinweis |
|------|-----|---------|---------|
| `name` | string | ja | Eindeutig |
| `resource_type` | string | ja | `computer_room`/`tablet`/`beamer`/`subject_room`/`laptop`/`other` |
| `location` | string | nein | |
| `description` | string | nein | |
| `capacity` | int | nein | Max. Kapazität |

**Response (201):** Erstellte Ressource.

---

### PUT /resources/{id}

Ressource aktualisieren.

**Zugriff:** eingeloggt

**Response (200):** Aktualisierte Ressource.

---

### DELETE /resources/{id}

Ressource deaktivieren (Soft-Delete).

**Zugriff:** eingeloggt

**Response:** `204 No Content`

---

## Buchungen

### GET /bookings

Buchungen abfragen.

**Zugriff:** eingeloggt

**Query-Parameter:**

| Parameter | Typ | Pflicht | Beschreibung |
|-----------|-----|---------|-------------|
| `resource_id` | int | nein | Filter nach Ressource |
| `booking_date` | string | nein | Filter nach Datum (`YYYY-MM-DD`) |
| `user_id` | int | nein | Filter nach User |

**Response (200):**
```json
[
  {
    "id": 1,
    "resource_id": 1,
    "resource_name": "IT-Raum 101",
    "user_id": 5,
    "user_name": "Max Mustermann",
    "booking_date": "2026-09-20",
    "time_start": "08:00",
    "time_end": "10:00",
    "purpose": "Informatik-Unterricht",
    "status": "confirmed",
    "created_at": "2026-09-15T14:00:00"
  }
]
```

---

### POST /bookings

Neue Buchung erstellen. Die Konfliktprüfung erfolgt automatisch.

**Zugriff:** eingeloggt

**Request Body:**
```json
{
  "resource_id": 1,
  "booking_date": "2026-09-20",
  "time_start": "08:00",
  "time_end": "10:00",
  "purpose": "Informatik-Unterricht"
}
```

| Feld | Typ | Pflicht | Hinweis |
|------|-----|---------|---------|
| `resource_id` | int | ja | |
| `booking_date` | string | ja | `YYYY-MM-DD` |
| `time_start` | string | ja | `HH:MM` |
| `time_end` | string | ja | muss > `time_start` |
| `purpose` | string | nein | Zweck der Buchung |

**Response (201):** Erstellte Buchung.

**Fehler bei Konflikt (409):**
```json
{
  "message": "Resource is already booked for this time slot"
}
```

---

### GET /bookings/overview

Buchungsübersicht (ähnlich Klassenbuch-Overview).

**Zugriff:** eingeloggt

**Response (200):**
```json
[
  {
    "resource_id": 1,
    "resource_name": "IT-Raum 101",
    "today_bookings": 3,
    "week_bookings": 12
  }
]
```

---

### GET /bookings/check-conflict

Explizite Konfliktprüfung ohne Buchung zu erstellen.

**Zugriff:** eingeloggt

**Query-Parameter:**

| Parameter | Typ | Pflicht |
|-----------|-----|---------|
| `resource_id` | int | ja |
| `booking_date` | string | ja |
| `time_start` | string | ja |
| `time_end` | string | ja |

**Response (200):**
```json
{
  "conflict": false
}
```

Bei Konflikt:
```json
{
  "conflict": true,
  "conflicting_booking": {
    "id": 5,
    "user_name": "Erika Musterfrau",
    "time_start": "09:00",
    "time_end": "11:00"
  }
}
```

---

### PUT /bookings/{id}

Buchung aktualisieren.

**Zugriff:** eingeloggt (nur eigene Buchungen)

**Response (200):** Aktualisierte Buchung.

---

### DELETE /bookings/{id}

Buchung stornieren.

**Zugriff:** eingeloggt (nur eigene Buchungen)

**Response:** `204 No Content`

---

## Resource-Types

| Typ | Beschreibung |
|-----|-------------|
| `computer_room` | Informatik-Raum |
| `tablet` | Tablet-Klasse |
| `beamer` | Beamer / Projektor |
| `subject_room` | Fachraum |
| `laptop` | Mobilgeräte |
| `other` | Sonstiges |

---

## Zugriffslogik

| Aktion | Rolle | Bedingung |
|--------|-------|-----------|
| Ressourcen lesen | eingeloggt | Alle |
| Buchungen lesen | eingeloggt | Eigene + alle sichtbar |
| Buchung erstellen | eingeloggt | Jeder |
| Buchung ändern | eingeloggt | Nur Eigentümer |
| Buchung stornieren | eingeloggt | Nur Eigentümer |

---

## Konfliktprüfung

Das System prüft beim Erstellen/Ändern einer Buchung, ob für dieselbe Ressource am selben Datum ein Zeitslot-Overlap existiert:

```
Neue Buchung:       08:00 ────── 10:00
Bestehende:              09:00 ────── 11:00
                         ↑ ÜBERLAPPUNG → 409 Conflict
```

Die Prüfung erfolgt auf Datenbank-Ebene mit einem UNIQUE-Constraint oder einer expliziten Abfrage.

---

## Fehler

| Status | Grund |
|--------|-------|
| 400 | Ungültige Felder (z.B. `time_end` <= `time_start`) |
| 401 | Kein Token / abgelaufen |
| 403 | Kein Zugriff (falscher User) |
| 404 | Ressource/Buchung nicht gefunden |
| 409 | Zeitkonflikt mit bestehender Buchung |

---

## Hinweise

- **Keine SSE-Events:** Buchungen lösen keine Echtzeit-Events aus.
- **Soft-Delete:** Ressourcen werden deaktiviert, nicht gelöscht (bestehende Buchungen bleiben erhalten).
- **Status:** Buchungen haben den Status `confirmed` oder `cancelled`.
- **Kapazität:** Die `capacity`-Angabe bei Ressourcen ist rein informativ – es gibt keine automatische Kapazitätsprüfung.
