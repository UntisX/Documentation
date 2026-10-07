# Frontend: API-Client

> Wie `api/client.ts`, `api/endpoints.ts` und `api/crypto.ts` zusammenarbeiten – die komplette Kommunikationsschicht.

---

## 1. Grundprinzip

```ts
const data = await apiRequest<User>('/users', { method: 'GET' });
```

`apiRequest` macht daraus automatisch:

```
fetch('/api/users', {
  headers: {
    Authorization: 'Bearer <token>',  // falls Token vorhanden
    'X-Req-Id': '<frische Request-ID>', // AAD + Anti-Replay
    'X-Enc': '1',                     // falls Body verschlüsselt wird / Antwort verschlüsselt sein soll
    'Content-Type': 'application/json'
  },
  body: '{"__enc":"...}'               // verschlüsseltes JSON
  cache: 'no-store'
})
```

Die drei Zusammenarbeit (detailliert in [Verschlüsselung](verschluesselung.md)):
- **Auth:** `Authorization: Bearer <token>`.
- **AAD:** „Methode + kompletter Bearer + `X-Req-Id`" werden byte-identisch zum Backend in `buildAad()` verrechnet und binden Envelope an diesen einen Request.
- **Replay-Schutz:** Jede Request-ID wird serverseitig 300 s lang auf Duplikate geprüft → doppelte ID = `400 Duplicate request identifier`.

---

## 2. API_ROOT und der Pfad-Mapping-Mechanismus

`api/endpoints.ts`:

```ts
export const API_ROOT = '/api';
```

Das Frontend ruft **immer** `/api/...` auf. Der Vite-Proxy entfernt das `/api`-Präfix, sodass beim Server `/users` ankommt.

Zusätzlich gibt es ein **Pfad-Mapping** (`ENDPOINT_MAP`): logische Frontend-Pfade werden auf echte Server-Routen abgebildet:

| Frontend-Aufruf | Server-Route |
|-----------------|--------------|
| `/settings` | `/school/settings` |
| `/schools/self` | `/school/settings` |
| `/users` | `/users` |
| `/admins`, `/students`, `/teachers` | `/users` (Client filtert nach Rolle) |
| `/subjects` | `/school/subjects` |
| `/rooms` | `/school/rooms` |
| `/classes` | `/school/classes` |
| `/grades` | `/students/grades` |
| `/absences` | `/students/absences` |
| `/messages` | `/messages` |
| `/chats` | `/chats` |
| `/events` | `/events` (SSE) |
| `/search` | `/search` |
| `/audit` | `/administration/audit` |
| `/class-book` | `/class-book` |
| `/resources` | `/resources` |
| `/bookings` | `/bookings` |
| `/video/meetings` | `/video/meetings` |

`resolveEndpoint()` sucht den **längsten passenden Map-Prefix** und hängt Rest + Query an.

> **Für eigene Frontends:** Du kannst die echten Server-Routen direkt verwenden – das Mapping ist nur eine Komfortschicht des offiziellen Clients.

---

## 3. Token-Verwaltung

| Funktion | Zweck |
|----------|-------|
| `setToken(token)` | Token im Modul + localStorage (`accessToken`) |
| `setTokens(access, refresh)` | Beide setzen (Refresh wird aktuell nicht genutzt) |
| `clearTokens()` | löschen |
| `getAccessToken()` | aktuell lesen |

**401-Verhalten:**

```
Nicht-OK-Status UND Token war gesetzt UND status===401
  → clearTokens()
  → window.location.href = '/login'   // harte Weiterleitung
```

---

## 4. Fehlerbehandlung

```ts
class ApiError extends Error {
  constructor(message: string, public status: number) { super(message); }
}
```

- Jede Nicht-2xx-Antwort wirft `ApiError`.
- Die `message` kommt aus dem Server-Body (`{message: "..."}`).
- Antworten mit `204 No Content` werden als `null` behandelt.

---

## 5. Verschlüsselung (Kurzfassung)

| Funktion | Verhalten |
|----------|-----------|
| `newRequestId()` | frische Request-ID für `X-Req-Id` |
| `buildAad(method, bearer, requestId)` | AAD-Bytes (Methode + Bearer + Req-Id), byte-identisch zum Backend |
| `encryptJson(payload, aad)` | body → `{__enc: ...}`, Header `X-Enc: 1` |
| `decryptBody(text, aad)` | response → automatisch entschlüsseln falls Envelope; wirft bei Fehlschlag |
| `decryptSseData(token, text)` | SSE-`data:`-Zeile → mit Token-AAD entschlüsseln |

- `X-Enc` wird gesetzt, wenn der Body ein echtes JSON ist (verschlüsselt) **oder** kein Body vorliegt (damit die Antwort verschlüsselt wird). Nicht-JSON-Bodies bleiben plaintext ohne `X-Enc`.
- Ohne WebCrypto: einmalige Warnung + Plaintext-Fallback.
- „Ungültige verschlüsselte Antwort" (ApiError) = Envelope ließ sich nicht authentisieren.

Details: [Verschlüsselung](verschluesselung.md)

---

## 6. SSE-Verbindung

Die SSE-Strecke läuft seit dem Security-Update über **`api/realtime.ts`** (`openEncryptedStream`) statt `new EventSource`:

```ts
openEncryptedStream('/api/events?enc=1', token, onData, onOpen?)
```

- **Token im `Authorization`-Header** (fetch-basiert) – nicht mehr im Query (`?token=` nur noch Server-Fallback für Alt-Clients).
- Jede `data:`-Zeile wird mit token-gebundenem AAD entschlüsselt (`decryptSseData`).
- Backoff-Reconnect: 1s → max 30s, max 10 Versuche; `close()` beendet den Stream.
- Genutzt in: `useRealtime`, `useChatStream` (beide `/api/events?enc=1`) und `VideoCallRoom` (`/api/video/signals/stream/{room}?enc=1`).

---

## 7. Alle Endpunkte, die das Frontend nutzt

Gegliedert nach Seiten/Tasks – als „Trigger“-Liste zum Nachschlagen:

| Zweck | Aufruf |
|-------|--------|
| Login | `POST /auth/login` |
| Logout | `POST /auth/logout` |
| Profil | `GET /auth/me` |
| Passwort ändern | `POST /auth/change-password` |
| Bootstrap | `POST /bootstrap` |
| Aktivieren | `POST /auth/activate` |
| Benutzer | `GET/POST /users`, `PUT/DELETE /users/{id}` |
| Aktivierungslink | `POST /users/{id}/activation-reset` |
| Noten | `GET/POST /students/grades`, `GET /students/grades/average`, `PUT/DELETE /students/grades/{id}` |
| Fehlzeiten | `GET/POST /students/absences`, `GET /students/absences/stats`, `PUT/DELETE /students/absences/{id}` |
| Fächer/Räume/Klassen | `GET/POST /school/{subjects,rooms,classes}`, `…/{id}` |
| Stundenplan | `GET/POST /timetable`, `PUT/DELETE /timetable/{id}` |
| Vertretungen | `GET/POST /timetable/substitutions`, `DELETE …/{id}` |
| Ausfälle | `POST /timetable/{id}/cancel`, `GET /timetable/cancellations` |
| Hausaufgaben | `GET/POST /timetable/homework`, `…/{id}` |
| Vertretungsplan | `GET /timetable/vertretungsplan` |
| Monatsdaten | `GET /timetable/month-data` |
| Klassenplan | `GET /timetable/class-plan` |
| Lehrer-Abwesenheiten | `GET/POST /timetable/teacher-absences` |
| Nachrichten | `GET/POST /messages`, `PUT /messages/{id}/read`, `DELETE /messages/{id}` |
| Chats | alle `/chats*` |
| Benachrichtigungen | `GET /notifications`, `PUT /notifications/{id}/read` |
| Kalender | `GET/POST /custom-events`, `…/{id}` |
| Suche | `GET /search?q=` |
| Settings/Tabs | `GET /settings/tabs`, `PUT /settings/tabs`, `GET /settings/moodle-url` |
| Präferenzen | `GET/PUT /preferences` |
| Audit | `GET /administration/audit`, `GET /administration/audit/stats` |
| API-Keys | `GET/POST /api-keys`, `PUT /api-keys/{id}/toggle`, `DELETE /api-keys/{id}` |
| Sync | `GET /sync/versions` |
| Proxy/Integration | `GET /proxy?url=` |
| Klassenbuch | `GET/POST /class-book`, `PUT/DELETE /class-book/{id}`, `GET /class-book/overview`, `GET /class-book/infoscreen` |
| Ressourcen | `GET/POST /resources`, `PUT/DELETE /resources/{id}` |
| Buchungen | `GET/POST /bookings`, `GET /bookings/overview`, `GET /bookings/check-conflict`, `PUT/DELETE /bookings/{id}` |
| Video-Meetings | `GET/POST /video/meetings`, `PUT/DELETE /video/meetings/{id}`, `GET /video/meetings/{id}/join`, `GET /video/join/{room}` |
| Video-Signaling | `POST /video/signals/{room}`, `GET /video/signals/stream/{room}` (SSE) |

---

## 8. Terminal-Konsole (separat)

`pages/Terminal.tsx` spricht **direkt** (ohne Mapping) mit:

```
/api/terminal/login
/api/terminal/stats
/api/terminal/schools?status=
/api/terminal/users
/api/terminal/logs
/api/terminal/schools/{id}/approve|reject|delete
/api/terminal/users/{id}/delete
```

Token in localStorage: `terminal_token`.

> Das Terminal ist eine **Multi-Tenant-Super-Admin-Schnittstelle** (alle Schulen), während die Haupt-App single-tenant ist.