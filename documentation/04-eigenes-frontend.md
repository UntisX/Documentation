# Eigenes Frontend verbinden

> Schritt-für-Schritt-Anleitung: baue dein eigenes Dashboard/UI und verbinde es mit dem UntisX-System.

---

## Das API-Vertrags-Modell in Kürze

UntisX ist eine klassische REST-API mit folgenden Regeln:

1. **JSON** in/out – immer `snake_case`-Felder.
2. **Auth** via `Authorization: Bearer <token>` Header (Token aus Login).
3. **Verschlüsselung (optional, aber empfohlen):** Anfrage-Body und Antwort werden mit AES-256-GCM in ein `{"__enc": "..."}`-Envelope verpackt, wenn der Header `X-Enc: 1` gesetzt ist. Wenn das Backend `ENCRYPTION_SECRET` gesetzt hat, muss dein Client den Schlüssel ebenfalls kennen.
4. **Fehler** kommen als `{"message": "..."}` mit passenden HTTP-Statuscodes.

---

## Variante A: Einfaches Frontend (ohne Verschlüsselung)

Wenn der Server im Dev/Test-Modus läuft und die Verschlüsselung in der Middleware nicht erzwungen wird, reicht ein simpler fetch:

```js
async function api(method, path, body, token) {
  const res = await fetch(`http://localhost:3000${path}`, {
    method,
    headers: {
      'Content-Type': 'application/json',
      ...(token && { Authorization: `Bearer ${token}` }),
    },
    body: body ? JSON.stringify(body) : undefined,
  });

  if (!res.ok) {
    const err = await res.json().catch(() => ({ message: res.statusText }));
    throw new Error(err.message || `HTTP ${res.status}`);
  }
  return res.status === 204 ? null : res.json();
}

// ─── Beispiele ───
async function login(userName, password) {
  const data = await api('POST', '/auth/login', { user_name: userName, password });
  // { token, token_type: "Bearer", expires_in_seconds: 604800, role }
  return data;
}

async function loadTimetable(token) {
  return api('GET', '/timetable', null, token);
}

async function createHomework(token, subjectId) {
  return api('POST', '/timetable/homework', {
    subject_id: subjectId,
    title: 'AB Seite 12',
    date: '2026-09-15'
  }, token);
}
```

---

## Variante B: Voll verschlüsselt (produktionssicher)

Für den Produktionsbetrieb (ENCRYPTION_SECRET gesetzt) brauchst du AES-256-GCM:

```js
// crypto.js – AES-256-GCM Envelope wie im offiziellen Client
import { Buffer } from 'buffer';

const enc = new TextEncoder();
const dec = new TextDecoder();

async function getKey() {
  const secret = import.meta.env.VITE_ENC_SECRET || 'UntisX-2026-AppLevel-Encryption-Secret-7f4c9a2b';
  const digest = await crypto.subtle.digest('SHA-256', enc.encode(secret));
  return crypto.subtle.importKey('raw', digest, { name: 'AES-GCM' }, false, ['encrypt', 'decrypt']);
}

export async function encryptJson(payload) {
  const key = await getKey();
  const iv = crypto.getRandomValues(new Uint8Array(12));
  const buffer = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv },
    key,
    enc.encode(JSON.stringify(payload))
  );
  const combined = new Uint8Array(iv.length + buffer.byteLength);
  combined.set(iv);
  combined.set(new Uint8Array(buffer), iv.length);
  return { __enc: Buffer.from(combined).toString('base64url') };
}

export async function tryDecryptBody(text) {
  try {
    const parsed = JSON.parse(text);
    if (parsed && typeof parsed.__enc === 'string') {
      const key = await getKey();
      const raw = Buffer.from(parsed.__enc, 'base64url');
      const iv = raw.slice(0, 12);
      const data = raw.slice(12);
      const plain = await crypto.subtle.decrypt({ name: 'AES-GCM', iv }, key, data);
      return JSON.parse(dec.decode(plain));
    }
    return parsed;
  } catch {
    return JSON.parse(text);
  }
}
```

### `apiRequest` – verschlüsselter Wrapper

```js
async function apiRequest(method, path, body, token) {
  const headers = { ...(token && { Authorization: `Bearer ${token}` }) };
  let payload;
  if (body !== undefined) {
    payload = await encryptJson(body);     // → { __enc: "..." }
    headers['Content-Type'] = 'application/json';
    headers['X-Enc'] = '1';                // Server weiß: verschlüsselt!
  }
  const res = await fetch(path, {
    method,
    headers,
    body: payload ? JSON.stringify(payload) : undefined,
    cache: 'no-store',
  });
  const rawText = await res.text();
  let data = rawText ? await tryDecryptBody(rawText) : null;
  if (!res.ok) throw new Error(data?.message || `HTTP ${res.status}`);
  return data;
}
```

> **Hinweis:** `X-Enc: 1` muss nur bei verschlüsselten Bodies gesetzt werden. Antworten werden automatisch erkannt (Envelope-Form).

---

## Der Pfad-Mapping-Mechanismus

Das offizielle Frontend nutzt teilweise andere Frontend-Pfade als die Server-Routen und mappt sie über `api/endpoints.ts`. Für EIGENE Frontends kannst du direkt die echten Server-Pfade verwenden:

| Offizielles Frontend ruft | Backend-Route |
|---------------------------|---------------|
| `/api/settings` | `/school/settings` |
| `/api/users` | `/users` |
| `/api/schools/self` | `/school/settings` |
| `/api/subjects` | `/school/subjects` |
| `/api/classes` | `/school/classes` |
| `/api/rooms` | `/school/rooms` |
| `/api/grades` | `/students/grades` |
| `/api/absences` | `/students/absences` |
| `/api/timetable` | `/timetable` |
| `/api/events` | `/events` |

**Empfehlung:** In deinem eigenen Frontend direkt die Server-Routen verwenden – das erspart Verwirrung.

---

## Vollständiges Login-Beispiel (über Vite-Proxy)

```js
// Bei Nutzung des Vite-Proxys ist die API-Basis einfach '/api'
const API = '/api';

async function login() {
  const res = await fetch(`${API}/auth/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ user_name: 'mueller', password: 'passwort' }),
  });
  const { token, role } = await res.json();
  localStorage.setItem('accessToken', token);
  return role; // 'admin' | 'teacher' | 'student'
}
```

---

## Wichtige Fallstricke

| Fallstrick | Erklärung |
|------------|-----------|
| `snake_case` | Feldnamen sind `first_name`, `user_name`, NICHT `firstName`. |
| Datumsformate | Datum `YYYY-MM-DD`, Zeit `HH:MM`, Timestamp ISO-8601 mit UTC. |
| `role` exakt | `admin` / `teacher` / `student` – sonst Zugriffsfehler. |
| Token-Probleme | Bei 401 mit ungültigem Token direkt auf Login-Seite leiten. |
| SSE-Token | Bei `/events` Token als Query-Parameter `?token=…&enc=1` übergeben (SSE kann keine Header). |
| Erst Admin bootstrapen | Vor allem: Der erste Login funktioniert erst nach `POST /bootstrap`. |
| `users/{id}/activation-reset` | Neue Key-Route nutzen, um Schüler-Aktivierungslinks zu erzeugen. |

---

## Mindest-Set für ein eigenes Frontend

Damit ein eigenes Frontend sinnvoll nutzbar ist, reichen diese 6 Aufrufe:

1. `POST /auth/login` → Token
2. `GET /auth/me` → User-Profil
3. `GET /timetable?…` → Stundenplan
4. `GET /students/grades` → Noten
5. `GET /timetable/homework` → Hausaufgaben
6. `GET /search?q=…` → Globale Suche

> Die komplette Endpunkt-Liste mit Request/Response-Schemas findest du im [API-Suchindex](09-api-suchindex.md) und in der [API-Referenz nach Bereich](10-api-nach-bereich.md).