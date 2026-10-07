# Authentifizierung & Sicherheit

> Der komplette Auth-Flow, das Token-System, API-Keys und die AES-256-GCM-Verschlüsselung.

---

## 1. Übersicht der Schutzmechanismen

| Mechanismus | Typ | Einsatz |
|-------------|-----|---------|
| **Opake Bearer-Tokens** | Auth für Benutzer | Alle geschützten Endpunkte |
| **X-Api-Key** | Auth für Dienste | `/external/*` (lesend) |
| **X-Bootstrap-Token** | Einmalige Auth | `POST /bootstrap` |
| **Argon2** | Passwort-Hashing | Login / Aktivierung |
| **SHA-256** | Token-/Key-Hashing | DB-Speicherung (kein Klartext) |
| **AES-256-GCM + AAD** | Payload-Verschlüsselung | Anfragen + Antworten (Envelope, AAD-gebunden) |
| **`X-Req-Id` + Anti-Replay** | Replay-Schutz | Doppelte Request-ID in 300 s → `400` |

---

## 2. Der Login-Flow im Detail

```
Client                               Server                               DB
  │  POST /auth/login                  │                                   │
  │  {user_name, password}  ─────────►│  validate_string                    │
  │                                    │  argon2::verify_password  ───────►│ hash
  │                                    │  (Erfolg/Fehler)                  │
  │                                    │  token = rand(32 bytes) → b64url  │
  │                                    │  hash = sha256(token)      ─────►│ sessions
  │                                    │                                   │
  │  {token, role, expires_in_seconds}◄┼── (Rückgabe)                       │
```

**Wichtige Details:**
- Der Token ist **32 zufällige Bytes**, base64url-codiert.
- In der DB wird **nicht der Token**, sondern sein **SHA-256-Hash** gespeichert.
- Gültigkeit: **7 Tage** (`expires_in_seconds: 604800`).
- `POST /auth/logout` löscht die Session (Hash) sofort.
- Expired Sessions werden automatisch **stündlich** vom Cleanup-Loop entfernt.

### Request

```http
POST /auth/login
Content-Type: application/json

{
  "user_name": "mueller",
  "password": "geheim123"
}
```

### Response

```json
{
  "token": "V3VXxklf9AfHhJ2pJb7vVqoVyQ94kH2F5T8zR3Yq1Pk",
  "token_type": "Bearer",
  "expires_in_seconds": 604800,
  "role": "teacher"
}
```

### Autorisierung aller weiteren Requests

```
Authorization: Bearer V3uXxklf9AfHhJ2pJb7vVqoVyQ94kH2F5T8zR3Yq1Pk
```

Der Server:
1. hasht den Header-Wert mit SHA-256
2. sucht nach der Session in `sessions`
3. prüft `expires_at > NOW()`
4. lädt den User (um dessen Rolle zu kennen)

---

## 3. Rollen-Autorisierung

### `validate_user` (jeder eingeloggte Benutzer)

Gibt `user_id` + `role` zurück oder `401 Unauthorized`.

### `validate_admin` (nur Admins)

Zusätzlich `403 Forbidden` wenn Rolle ≠ `admin`.

### Betroffene Endpunkte

| Nur Admin | Authentifiziert (Role egal) |
|-----------|------------------------------|
| `POST/PUT/DELETE /users` | Alle übrigen Endpunkte |
| `POST/PUT/DELETE /school/subjects, /rooms, /classes` | Lesen erlaubt für alle |
| `POST/PUT/DELETE /timetable` | |
| `POST/PUT/DELETE /bell-schedule` | |
| `PUT /settings/tabs` | |
| `GET /administration/audit/stats` | |
| `GET/POST/PUT/DELETE /api-keys` | |

### Datenroollen (students/teachers)

| Endpunkt | Zugriff |
|----------|---------|
| `GET /students/grades` | Schüler: nur eigene; Lehrer/Admin: alle |
| `GET /students/absences` | Schüler: nur eigene; Lehrer/Admin: alle |
| `POST /timetable/homework` | Nur Lehrer/Admin |

---

## 4. Bootstrap – erster Admin

Der allererste Admin-Account wird über `POST /bootstrap` erzeugt:

```http
POST /bootstrap
X-Bootstrap-Token: <BOOTSTRAP_TOKEN aus .env>
Content-Type: application/json

{
  "user_name": "admin",
  "real_name": "Admin",
  "email": "admin@schule.de",
  "password": "SehrSicher123!"
}
```

| Ergebnis | Code |
|----------|------|
| Admin angelegt | `201` |
| Admin existiert schon | `409` |

---

## 5. Benutzer-Aktivierung

Neue Benutzer werden ohne Passwort angelegt. Sie erhalten über `POST /users` (Admin) oder `POST /users/{id}/activation-reset` einen **Aktivierungsschlüssel**:

```json
{
  "id": 12,
  "user_name": "s2001",
  "activation_key": "AbCdEf123456...",
  "activation_expires_in_hours": 72
}
```

### Aktivierung durch den Benutzer

```http
POST /auth/activate
Content-Type: application/json

{
  "user_name": "s2001",
  "activation_key": "AbCdEf123456...",
  "email": "s2001@schule.de",
  "password": "NeuesPasswort"
}
```

→ `204 No Content`

Der Server vergleicht `activation_key_hash`, setzt das Passwort und markiert `activated_at`.

---

## 6. API-Keys (für Dienste)

### Erstellung (nur Admin)

```http
POST /api-keys
Authorization: Bearer <admin-token>

{
  "name": "Stundenplan-Portal",
  "scopes": ["/external/classes", "/external/subjects", "/timetable/class-plan"],
  "api_type": "read"
}
```

Das Format des Keys: **`untisx_` + base64url(Zufallsbytes)**.

> **Achtung:** Der Key wird nur EINMAL zurückgegeben und niemals in der DB gespeichert (nur sein SHA-256-Hex-Hash). Verliere ihn → lösche ihn und erstelle einen neuen.

### Verwendung

```
GET /external/subjects
X-Api-Key: untisx_VGhpcyAgaXMgYSBzZWNyZXQ...
```

### Zugriffskontrolle pro Pfad

`ApiKeyContext::allows(path, method)` prüft:
1. Existiert der Key? → sonst `401`
2. Ist `active == true`? → sonst `401`
3. Ist der Pfad in `scopes`? → sonst `403`
4. Passt `api_type` zur HTTP-Methode? → sonst `403`

---

## 7. Payload-Verschlüsselung (AES-256-GCM, AAD-gebunden)

### Prinzip

Alle JSON-Bodies (Requests UND Responses) werden transparent als Envelope übertragen. Jeder Envelope ist gegen den **Request-Kontext** gebunden:

```
AAD = build_aad(Methode, kompletter Authorization-Wert, X-Req-Id)
    = Längenpräfixierte (u32BE) Konkatenation der drei Teile

Klartext JSON
   │
   ▼
crypto.subtle.encrypt(key = SHA-256(ENCRYPTION_SECRET), iv = 12 Zufallsbytes, additionalData = AAD)
   │
   ▼
Envelope: { "__enc": base64url(iv ‖ ciphertext ‖ authTag) }
   │
   ▼  Header "X-Enc: 1" + "X-Req-Id: <frische ID>"
   ▼
Server -> encryption_middleware: entschlüsselt (wenn X-Enc) / verschlüsselt Antwort (wenn X-Enc)
```

### Was das bringt

- **Kontextbindung:** Ein aufgezeichneter Envelope kann weder auf eine andere Methode, eine andere Session/Token noch eine andere Request-ID verschoben werden – er lässt sich dort nicht mehr entschlüsseln.
- **Anti-Replay:** Der Server merkt sich jede `X-Req-Id` für **300 s** (Cache-Cap 50 000, TTL-Prune). Eine doppelte ID → `400 Duplicate request identifier`. Damit kann ein Replay-Angriff (gleicher verschlüsselter Request erneut abspielen) den Server nicht zweimal ausführen.
- **Kein Token-Leak in URLs:** Der offizielle Client überträgt SSE-Tokens im `Authorization`-Header statt im Query.

### Middleware-Verhalten (server-basis/src/crypto.rs)

| Bedingung | Verhalten |
|-----------|-----------|
| Request mit `X-Enc: 1` + JSON-Body | Body-Envelope wird mit Request-AAD entschlüsselt und als JSON weitergereicht |
| Request mit `X-Enc: 1` + leerem Body | Skip (nur Antwortpfad aktiv) |
| Request mit `X-Enc: 1` + doppelter `X-Req-Id` | `400 Duplicate request identifier` |
| Request ohne `X-Enc` | Klartext wie normal verarbeitet (Abwärtskompatibilität) |
| Response auf `X-Enc`-Request | JSON-Antwort wird mit dem **Request-AAD** verschlüsselt; leere Bodies → `{"__enc": <Envelope von {}>}` |
| Nicht entschlüsselbarer Body | `400 Could not decrypt request body` |
| Nicht verschlüsselbare Antwort | `500 Encryption failed` |
| SSE (`/events`, `/video/signals/stream/…`) | jede `data:`-Zeile einzeln verschlüsselt sobald `enc=1` – AAD = `build_aad("GET", "Bearer <token>", "")` |

### Schlüssel-Ableitung (muss auf beiden Seiten identisch sein!)

```
K = SHA-256(UTF8(ENCRYPTION_SECRET))   // Backend: ENCRYPTION_SECRET
VITE_ENC_SECRET == ENCRYPTION_SECRET   // Frontend: VITE_ENC_SECRET
```

Dev-Fallback (wenn nichts gesetzt): `UntisX-2026-AppLevel-Encryption-Secret-7f4c9a2b`

### build_aad (muss byte-identisch auf beiden Seiten implementiert sein)

```
build_aad(method, bearer, request_id):
  out = []
  für jeden Teil p in [method, bearer, request_id]:
    out += u32_BE(len(p))            # 4 Bytes Big-Endian
    out += utf8(p)
  return out
```

- **method:** exakt wie übertragen, z.B. `"POST"` (Großschreibung).
- **bearer:** der **komplette** `Authorization`-Wert inkl. `"Bearer "` – leer wenn kein Token.
- **request_id:** Wert des `X-Req-Id`-Headers; bei SSE-Streams die leere Zeichenkette `""`.
- Der Pfad ist **nicht** Teil der AAD (Proxy-Präfix-Unsicherheit), Methoden+Bearer+ID binden ausreichend.

### Beispiel (manuell, Node.js)

```js
const secret = 'MeinGeheimerSchluessel';
const key = await crypto.subtle.importKey('raw', await crypto.subtle.digest('SHA-256', new TextEncoder().encode(secret)), {name:'AES-GCM'}, false, ['encrypt','decrypt']);

function buildAad(method, bearer, id) {
  const enc = new TextEncoder();
  const parts = [method, bearer, id].map(p => enc.encode(p));
  const out = new Uint8Array(parts.reduce((s,p)=>s+4+p.length,0));
  const v = new DataView(out.buffer);
  let o = 0;
  for (const p of parts) { v.setUint32(o, p.length); o+=4; out.set(p,o); o+=p.length; }
  return out;
}

const aad = buildAad('POST', 'Bearer meinToken', 'req-123');

// Verschlüsseln (Payload als Buffer)
const iv = crypto.getRandomValues(new Uint8Array(12));
const ct = await crypto.subtle.encrypt({name:'AES-GCM', iv, additionalData: aad}, key, Buffer.from(JSON.stringify({hello:'welt'})));
const envelope = {__enc: Buffer.concat([iv, ct]).toString('base64url')};

// Entschlüsseln
const raw = Buffer.from(envelope.__enc, 'base64url');
const plain = await crypto.subtle.decrypt({name:'AES-GCM', iv: raw.subarray(0,12), additionalData: aad}, key, raw.subarray(12));
console.log(JSON.parse(new TextDecoder().decode(plain)));
```

> **Hinweis:** Das alte Format (ohne `additionalData`, ohne `X-Req-Id`) ist **nicht** mehr kompatibel – es handelt sich um einen Wire-Break. Client und Server gemeinsam ausrollen!

### CORS

Der Server erlaubt (statt `Any`) nur explizite Methoden und Header – für den Browser-CORS-Dialog relevant, für curl/fetch egal:

```
allow_methods: GET, POST, PUT, PATCH, DELETE, OPTIONS
allow_headers: content-type, authorization, x-enc, x-req-id
```

---

## 8. Sicherheits-Konventionen der Rollen-Namen

| Exakter Wert | Bedeutung |
|--------------|-----------|
| `admin` | Schul-Admin |
| `teacher` | Lehrkraft |
| `student` | Schüler/in |

> Die Werte sind **case-sensitive** und dürfen weder geändert noch übersetzt werden.

---

## 9. Empfohlene Sicherheits-Einstellungen

| Maßnahme | Empfehlung |
|----------|-----------|
| `ENCRYPTION_SECRET` | Mind. 32 Zeichen, kryptografisch zufällig; identisch zu `VITE_ENC_SECRET` |
| `X-Req-Id` pro Request | Frische, einmalige ID senden – serverseitiges Anti-Replay (300 s) greift sonst nicht |
| `BOOTSTRAP_TOKEN` | `openssl rand -hex 32` |
| `CORS_ORIGIN` | Nur die echten Frontend-Origins (Methoden/Header sind serverseitig auf Whitelist beschränkt) |
| PostgreSQL-Port | Im Produktivbetrieb nach außen schließen |
| Passwörter | Mind. 8 Zeichen (Server validiert), Argon2-verifiziert |
| `auth/logout` | Aktiv nutzen – Session wird sofort gelöscht |
| API-Keys | `read`-Typ für öffentliche Integrationen, `full` nur intern |