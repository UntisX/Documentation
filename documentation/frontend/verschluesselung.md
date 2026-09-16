# Frontend: Verschlüsselung

> Die AES-256-GCM-Implementierung des offiziellen Clients (`src/api/crypto.ts`) – exakt gleich wie die Server-Middleware.

---

## Warum überhaupt?

Alle JSON-Payloads (Requests UND Responses) können transparent verschlüsselt transportiert werden. So sind sensible Daten (Noten, Fehlzeiten, Chats) auch bei einem kompromittierten Reverse-Proxy geschützt.

---

## Das Envelope-Format

Jede verschlüsselte Nachricht sieht so aus:

```json
{
  "__enc": "<base64url(iv · ciphertext · authTag)>"
}
```

- **iv**: 12 Bytes, zufällig pro Nachricht
- **ciphertext**: AES-256-GCM-Verschlüsselter Klartext
- **authTag**: Authenticity-Tag (in Serverfällen am Ende angehängt)

Die drei Teile werden verkettet und base64url codiert.

---

## Schlüssel-Ableitung

Der 32-Byte-AES-Schlüssel wird aus dem Secret abgeleitet:

```
secret = VITE_ENC_SECRET        (Frontend)
         / ENCRYPTION_SECRET    (Backend)
         / 'UntisX-2026-AppLevel-Encryption-Secret-7f4c9a2b'  (Dev-Fallback)

key = SHA-256(UTF8(secret))
```

> **Kritisch:** Beide Secrets MÜSSEN identisch sein. Bei Abweichung: verweigerte Entschlüsselung → 400/Chiffre-Fehler.

---

## Der Code (Vereinfacht)

```ts
const enc = new TextEncoder();
const dec = new TextDecoder();

async function getKey(): Promise<CryptoKey> {
  const secret = import.meta.env.VITE_ENC_SECRET
    ?? 'UntisX-2026-AppLevel-Encryption-Secret-7f4c9a2b';

  const digest = await crypto.subtle.digest('SHA-256', enc.encode(secret));
  return crypto.subtle.importKey('raw', digest, { name: 'AES-GCM' }, false, ['encrypt', 'decrypt']);
}

// Anfrage-Body verschlüsseln
export async function encryptJson(payload: unknown): Promise<{ __enc: string }> {
  const key = await getKey();
  const iv = crypto.getRandomValues(new Uint8Array(12));        // 12 Bytes IV
  const buffer = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv },
    key,
    enc.encode(JSON.stringify(payload))
  );

  const combined = new Uint8Array(iv.length + buffer.byteLength);
  combined.set(iv);
  combined.set(new Uint8Array(buffer), iv.length);
  return { __enc: btoa(String.fromCharCode(...combined)).replaceAll('+', '-').replaceAll('/', '_').replace(/=+$/, '') };
}

// Antwort entschlüsseln (falls Envelope)
export async function tryDecryptBody<T>(text: string): Promise<T> {
  const parsed = JSON.parse(text);
  if (parsed && typeof parsed.__enc === 'string') {
    const key = await getKey();
    const raw = Uint8Array.from(atob(parsed.__enc), (c) => c.charCodeAt(0));
    const iv = raw.slice(0, 12);
    const data = raw.slice(12);
    const plain = await crypto.subtle.decrypt({ name: 'AES-GCM', iv }, key, data);
    return JSON.parse(dec.decode(plain));
  }
  return parsed;
}
```

---

## Der Header `X-Enc`

| Richtung | Header | Vorgang |
|----------|--------|---------|
| Request (mit Body) | `X-Enc: 1` | Body wird mit `encryptJson` verschlüsselt |
| Request (ohne Body) | – | Kein Header, kein Envelope |
| Response | – | Server erkennt verschlüsselten Request und gibt Envelope zurück (Kontext der Middleware) |
| SSE-Daten | – | `enc=1` im Query-Parameter des `/events`-Aufrufs |

---

## Verwendung in `api/client.ts`

```ts
export async function apiRequest<T>(endpoint: string, options: ApiOptions = {}): Promise<T> {
  const path = resolveEndpoint(endpoint, options);   // Mapping
  const headers: Record<string, string> = {};
  const token = getAccessToken();
  if (token) headers['Authorization'] = `Bearer ${token}`;

  let body: string | undefined;
  if (options.body !== undefined) {
    headers['Content-Type'] = 'application/json';
    headers['X-Enc'] = '1';
    body = JSON.stringify(await encryptJson(options.body));
  }

  const res = await fetch(path, { method: options.method ?? 'GET', headers, body, cache: 'no-store' });
  const text = await res.text();
  let data: unknown = text ? await tryDecryptBody(text) : null;
  if (!res.ok) throw new ApiError(data?.message ?? res.statusText, res.status);
  return data as T;
}
```

---

## SSE-Entschlüsselung

```ts
export async function tryDecryptSseData<T>(raw: string): Promise<T> {
  try {
    return await tryDecryptBody<T>(raw);   // gleiches Envelope-Schema
  } catch {
    return JSON.parse(raw);                // Fallback: unverschlüsselt
  }
}
```

---

## Häufige Fehler

| Symptom | Ursache | Lösung |
|---------|---------|--------|
| `400 Bad Request` bei jedem POST | Server hat anderes `ENCRYPTION_SECRET` | Secrets angleichen |
| `OperationError` im Browser | IV/Envelope fehlerhaft (z.B. falsch codiert) | base64url korrekt bilden |
| Login klappt, Rest nicht | Nur `VITE_ENC_SECRET` geändert, aber Build gecacht | `npm run build` neu |
| Griechische/Sonderzeichen im Secret | Zeichenkodierung | Reines ASCII verwenden |