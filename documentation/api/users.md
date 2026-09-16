# API: Benutzer (`/users`)

> Benutzerverwaltung inkl. Aktivierungsschlüssel. CUD nur für **admin**; `GET` für jeden eingeloggten Benutzer.

---

## GET /users

Liste aller Benutzer.

**Eingeloggt.**

### Response (200)

```json
[
  {
    "id": 1,
    "user_name": "admin",
    "real_name": "Administrator",
    "short_name": null,
    "role": "admin",
    "email": "admin@schule.de",
    "class_id": null,
    "class_name": null,
    "created_at": "2026-09-01T08:00:00Z",
    "activated_at": "2026-09-01T08:05:00Z",
    "activation_expires_at": null
  },
  {
    "id": 7,
    "user_name": "s2001",
    "real_name": "Anna Schmidt",
    "short_name": null,
    "role": "student",
    "email": null,
    "class_id": 3,
    "class_name": "7a",
    "created_at": "2026-09-01T09:00:00Z",
    "activated_at": null,
    "activation_expires_at": "2026-09-04T09:00:00Z"
  }
]
```

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `id` | Number | Benutzer-ID |
| `user_name` | String | Login-Name (eindeutig) |
| `real_name` | String | Voller Name |
| `short_name` | String/null | Abkürzung |
| `role` | String | `admin`/`teacher`/`student` |
| `email` | String/null | |
| `class_id` | Number/null | Klassen-ID |
| `class_name` | String/null | Klassenname (JOIN) |
| `created_at` | Timestamp | |
| `activated_at` | Timestamp/null | |
| `activation_expires_at` | Timestamp/null | |

---

## POST /users

Legt einen neuen Benutzer an und erzeugt einen Aktivierungsschlüssel.

**Nur Admin.**

### Request-Body

```json
{
  "user_name": "s2099",
  "real_name": "Lina Meyer",
  "short_name": "LM",
  "role": "student",
  "class_id": 3
}
```

| Feld | Pflicht | Beschreibung |
|------|---------|--------------|
| `user_name` | ✔ | Eindeutiger Login-Name |
| `real_name` | ✔ | Voller Name |
| `short_name` | – | Abkürzung |
| `role` | ✔ | `admin`/`teacher`/`student` |
| `class_id` | – | Nur für `student` sinnvoll |

### Response (201)

```json
{
  "id": 12,
  "user_name": "s2099",
  "activation_key": "AbCdEf1234567890",
  "activation_expires_in_hours": 72
}
```

| Feld | Beschreibung |
|------|--------------|
| `id` | Neue Benutzer-ID |
| `user_name` | |
| `activation_key` | Klartext-Schlüssel – an den Benutzer weitergeben (PDF-Brief im Frontend!) |
| `activation_expires_in_hours` | Standard: 72 |

### Fehler

| Status | Grund |
|--------|-------|
| 403 | Kein Admin |
| 409 | `user_name` bereits vergeben |
| 422 | Validierungsfehler |

---

## PUT /users/{id}

Ändert einen Benutzer.

**Nur Admin.**

### Request-Body (alle Felder optional)

```json
{
  "user_name": "s2099",
  "real_name": "Lina Meyer-Schmidt",
  "short_name": "LMS",
  "role": "student",
  "email": "lina@schule.de",
  "class_id": 4
}
```

### Response (200)

`CompleteUserResponse` – gleiche Struktur wie `GET /users` einzelnes Element.

---

## DELETE /users/{id}

Löscht einen Benutzer aus der Datenbank.

**Nur Admin.**

### Response

`204 No Content`

> **Achtung:** Löscht auch zugehörige Session-Rows und verschlüsselte Referenzen nicht automatisch – prüfe Referenzen (Absences, Grades, Chats), bevor du löschst.

---

## POST /users/{id}/activation-reset

Erzeugt einen **neuen** Aktivierungsschlüssel (z.B. wenn der alte verfallen ist).

**Nur Admin.**

### Response (201)

```json
{
  "id": 12,
  "user_name": "s2099",
  "activation_key": "NeuGegebenErSchluessel5678",
  "activation_expires_in_hours": 72
}
```

---

## Ablauf Aktivierung (Ende-zu-Ende)

```
1. Admin     POST /users → { id, activation_key }
2. Admin     (Frontend) generiert PDF-Brief mit QR-Code (jsPDF + qrcode)
3. Benutzer  öffnet /activate oder scannt QR
4. Benutzer  POST /auth/activate { user_name, activation_key, email, password }
5. Frontend  → 204, weiter zu /login
```

---

## Frontend-Mapping (mappers.ts)

Das offizielle Frontend übersetzt:

```ts
mapUser(u) => ({
  username: u.user_name,
  first_name: u.real_name.split(' ')[0],
  last_name: u.real_name.split(' ').slice(1).join(' '),
  role: u.role,
  email: u.email,
  created_at: u.created_at,
  activated_at: u.activated_at
})
```