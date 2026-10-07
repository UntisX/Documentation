# API: Authentifizierung (`/auth`)

> Login, Logout, Profil und Kontenaktivierung.

---

## POST /auth/login

Loggt einen Benutzer ein und liefert ein Bearer-Token.

**Öffentlich** – kein Token nötig.

### Request-Body

```json
{
  "user_name": "mueller",
  "password": "geheim123"
}
```

### Response (200)

```json
{
  "token": "V3uXxklf9AfHhJ2pJb7vVqoVyQ94kH2F5T8zR3Yq1Pk",
  "token_type": "Bearer",
  "expires_in_seconds": 604800,
  "role": "teacher"
}
```

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `token` | String | Opakes Bearer-Token (32 Zufallsbytes, base64url) |
| `token_type` | String | Immer `"Bearer"` |
| `expires_in_seconds` | Number | 604800 = 7 Tage |
| `role` | String | `admin` / `teacher` / `student` |

### Fehler

| Status | Grund |
|--------|-------|
| 401 | Falsche Zugangsdaten |
| 422 | Feld-Validierungsfehler (leer, zu lang) |

### Beispiel (curl)

```bash
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"user_name":"mueller","password":"geheim123"}'
```

> **Hinweis:** In der Theorie prüft das Backend per `argon2::verify_password`. Es gibt kein Rate-Limiting – ein eigenes Frontend sollte Login-Versuche selbst drosseln.

---

## GET /auth/me

Liefert das Profil des aktuell eingeloggten Benutzers.

**Eingeloggt** – `Authorization: Bearer <token>`.

### Response (200)

```json
{
  "id": 7,
  "user_name": "mueller",
  "real_name": "Max Müller",
  "short_name": "MM",
  "role": "teacher",
  "email": "max.mueller@schule.de",
  "school_name": "Gymnasium Musterstadt",
  "school_logo": "data:image/png;base64,iVBORw0KGgo..."
}
```

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `id` | Number | User-ID |
| `user_name` / `real_name` / `short_name` | String | Name(n) |
| `role` | String | `admin` / `teacher` / `student` |
| `email` | String, optional | E-Mail |
| `school_name` | String, optional | Schulname (Branding; aus `school_settings`) |
| `school_logo` | String, optional | Schul-Logo als **Data-URL**, leer/`null` = keins (Migration 050) |

> Mit diesen beiden neuen Feldern können Login-, Setup- und Aktivierungsseiten die Schule schon **vor** dem ersten Voll-Login gebrandet zeigen.

### Fehler

| Status | Grund |
|--------|-------|
| 401 | Kein/ungültiger Token |

### Frontend-Verwendung

Der offizielle Client ruft `/auth/me` beim App-Start und nach jedem Login auf und mappt das Ergebnis via `mappers.ts` auf `{username, first_name, last_name, ...}`.

---

## POST /auth/logout

Beendet die aktuelle Session und löscht das Token serverseitig.

**Eingeloggt** – `Authorization: Bearer <token>`.

### Response

`204 No Content`

### Beispiel (curl)

```bash
curl -X POST http://localhost:3000/auth/logout \
  -H "Authorization: Bearer $TOKEN"
```

---

## POST /auth/activate

Schnellt die Aktivierung eines Benutzerkontos ab (setzt das Passwort).

**Öffentlich.**

### Request-Body

```json
{
  "user_name": "s2001",
  "activation_key": "AbCdEf123456...",
  "email": "s2001@schule.de",
  "password": "NeuesSicheresPasswort"
}
```

| Feld | Beschreibung |
|------|--------------|
| `user_name` | Benutzername des zu aktivierenden Kontos |
| `activation_key` | Schlüssel aus dem Aktivierungslink (siehe `/users`, `/users/{id}/activation-reset`) |
| `email` | E-Mail-Adresse für das Konto |
| `password` | Neues Passwort (wird Argon2-gehasht) |

### Response

`204 No Content`

### Fehler

| Status | Grund |
|--------|-------|
| 400 | Ungültiger Schlüssel / Zustand |
| 404 | Benutzer nicht gefunden |

---

## Ablauf im Frontend

```
Login.tsx ─┐
           ├─ POST /auth/login → Token → localStorage('accessToken')
           ├─ GET /auth/me → user
           └─ POST /auth/logout → clear localStorage

Activate.tsx ─┐
              └─ POST /auth/activate → Erfolg → weiter zu /login

AuthContext ──┐
              ├─ Beim Start: token lesen, 401-Monitor (bis 5× Retry)
              └─ onSessionExpiryWarning(cb): Callback 5 Min. vor Ablauf
```