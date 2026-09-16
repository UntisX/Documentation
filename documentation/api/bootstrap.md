# API: Bootstrap

> Erstmalige Einrichtung eines Single-Tenant-Servers – erzeugt den ersten Admin.

---

## POST /bootstrap

**Auth:** `X-Bootstrap-Token: <BOOTSTRAP_TOKEN>` (aus der Server-`.env`).

Dieser Endpunkt kann **genau einmal** erfolgreich ausgeführt werden.

### Request-Body

```json
{
  "user_name": "admin",
  "real_name": "Administrator",
  "email": "admin@schule.de",
  "password": "SehrSicheresPasswort"
}
```

### Response

`201 Created` (kein Body)

### Fehler

| Status | Grund |
|--------|-------|
| 400 | Token fehlt/unkorrekt |
| 409 | Admin/Benutzername existiert bereits (ON CONFLICT DO NOTHING) |
| 422 | Feld-Validierungsfehler |

### Beispiel (curl)

```bash
curl -X POST http://localhost:3000/bootstrap \
  -H "Content-Type: application/json" \
  -H "X-Bootstrap-Token: $(cat .env | grep BOOTSTRAP_TOKEN | cut -d= -f2)" \
  -d '{
    "user_name": "admin",
    "real_name": "Schulleitung",
    "email": "admin@schule.de",
    "password": "EinGanzSicheresPasswort1!"
  }'
```

### Intern

- `INSERT INTO users ... ON CONFLICT DO NOTHING`
- Argon2-Hashing des Passworts
- Gibt DB-Fehlercode `23505` (unique) als `409` zurück

### Sicherheitshinweis

- Nach erfolgreichem Bootstrap `BOOTSTRAP_TOKEN` aus dem Betrieb nehmen (Secret-Drehung per Neustart).
- Alle weiteren Admins kannst du über die normale Benutzerverwaltung anlegen.