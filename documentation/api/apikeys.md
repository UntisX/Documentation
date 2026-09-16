# API: API-Schlüssel (`/api-keys`)

> Verwalte Schlüssel für externe Dienste (z.B. Homepage-Integration). Nur Admin.

---

## Format & Sicherheit

- Key-Format: **`untisx_<base64url zufällige Bytes>`**
- In der DB wird NUR der **SHA-256-Hex-Hash** gespeichert (`key_hash`), niemals der Klartext.
- Der Klartext wird dir **einmalig bei Erstellung** zurückgegeben – danach nie wieder.
- `key_hint` zeigt die ersten Zeichen zur Identifikation.

---

## GET /api-keys

**Nur Admin.**

### Response (200)

```json
[
  {
    "id": 1,
    "name": "Homepage-Anzeige",
    "key_hint": "untisx_VGhpcyBpc1RoZVNlY3JldA...",
    "scopes": ["/external/classes", "/external/subjects"],
    "api_type": "read",
    "active": true,
    "created_at": "2026-09-01T08:00:00Z"
  }
]
```

> `key` (Klartext) fehlt hier absichtlich.

---

## POST /api-keys

**Nur Admin.**

```json
{
  "name": "Stundenplan-Portal",
  "scopes": ["/external/classes", "/external/subjects", "/external/bell-schedule"],
  "api_type": "read"
}
```

### Response (201) – ACHTUNG: Key nur hier!

```json
{
  "id": 2,
  "name": "Stundenplan-Portal",
  "key": "untisx_WvfA41gS9bLm...",   // ⚠ JETZT SPEICHERN / KOPIEREN
  "key_hint": "untisx_WvfA41gS9bLm...",
  "scopes": ["/external/classes", "/external/subjects", "/external/bell-schedule"],
  "api_type": "read",
  "active": true,
  "created_at": "2026-09-13T10:00:00Z"
}
```

---

## PUT /api-keys/{id}/toggle

**Nur Admin.** Aktiviert/deaktiviert einen Key (kein Body).

### Response (200)

`ApiKeyResponse` (mit `active` umgeschaltet).

---

## DELETE /api-keys/{id}

**Nur Admin.** `204 No Content`.

---

## Nutzung externer Endpunkte

```
GET /external/subjects
Header: X-Api-Key: untisx_WvfA41gS9bLm...
```

### Prüflogik (server-basis)

```
X-Api-Key vorhanden?
  └─ Nein → 401
  └─ Ja  → key existiert? + active == true?
              └─ Nein → 401
              └─ Ja  → Pfad in scopes?
                          └─ Nein → 403
                          └─ Ja  → api_type erlaubt die Methode?
                                      └─ Nein → 403
                                      └─ Ja  → Request erlaubt ✅
```

### api_type → erlaubte Methoden

| api_type | GET | POST | PUT | DELETE |
|----------|-----|------|-----|--------|
| `read` | ✔ | – | – | – |
| `write` | ✔ | ✔ | ✔ | ✔ |
| `full` | ✔ | ✔ | ✔ | ✔ |

---

## Scope-Referenz (Basis für eigenes cURL/Auto-Tooling)

Alle Server-Routen sind potenzielle Scopes. Häufig verwendete:

```
/external/classes
/external/subjects
/external/rooms
/external/bell-schedule
/timetable/class-plan
/timetable/vertretungsplan
/school/settings
/users
/search
```

> Das AdminPanel im Frontend zeigt alle Routen als Scope-Auswahl (siehe `API_ROUTE_GROUPS` in `AdminPanel.tsx`).