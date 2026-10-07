# Frontend: Verschlüsselung

> Die AES-256-GCM-Implementierung des offiziellen Clients (`src/api/crypto.ts`) – exakt gleich wie die Server-Middleware in `server-basis/src/crypto.rs`.

---

## Warum überhaupt?

Alle JSON-Payloads (Requests UND Responses) können transparent verschlüsselt transportiert werden. So sind sensible Daten (Noten, Fehlzeiten, Chats) auch bei einem kompromittierten Reverse-Proxy oder Log-Layer geschützt – **App-Level-Verschlüsselung**, unabhängig von TLS.

Seit dem Security-Update ist die Verschlüsselung zusätzlich **an den Request-Kontext gebunden**:

- Jede Nachricht wird gegen **Additional Authenticated Data (AAD)** authentisiert (Methode + Bearer + `X-Req-Id`).
- Damit kann ein (mit dem Key ausgestatteter) Angreifer aufgenommenen Klartext-Envelopes **nicht** auf andere Endpunkte, unter anderen Sessions oder als Replay verschieben.
- Pro Request sendet der Client eine **frische `X-Req-Id`**; der Server lehnt doppelte IDs innerhalb 300 s ab (Anti-Replay).

---

## Das Envelope-Format

Jede verschlüsselte Nachricht sieht so aus:

```json
{
  "__enc": "<base64url(iv · ciphertext · authTag)>"
}
```

- **iv**: 12 Bytes, zufällig pro Nachricht
- **ciphertext**: AES-256-GCM-verschlüsselter Klartext
- **authTag**: Authenticity-Tag (GCM hängt ihn an)

Die drei Teile werden verkettet und **base64url** codiert (kein `=`, `-`/`_` statt `+`/`/`).

---

## Schlüssel-Ableitung

Der 32-Byte-AES-Schlüssel wird aus dem Secret abgeleitet:

```
secret = VITE_ENC_SECRET        (Frontend)
         / ENCRYPTION_SECRET    (Backend)
         / 'UntisX-2026-AppLevel-Encryption-Secret-7f4c9a2b'  (Dev-Fallback)

key = SHA-256(UTF8(secret))
```

> **Kritisch:** Beide Secrets MÜSSEN identisch sein. Bei Abweichung: verweigerte Entschlüsselung → `400 Could not decrypt request body`.

---

## Der AAD-Aufbau (muss byte-identisch auf beiden Seiten sein)

AAD = längenpräfixierte Konkatenation (drei Teile, jeweils **u32 Big-Endian Länge + Bytes**):

| Teil | Wert | Beispiel |
|------|------|----------|
| 1. Methode | `method.toUpperCase()` | `"POST"` |
| 2. Bearer | kompletter `Authorization`-Wert **inkl. `"Bearer "`** | `"Bearer V3uXxklf..."` |
| 3. Request-Id | Wert des `X-Req-Id`-Headers | `"9f2c8a1b..."` |

```ts
// client/src/api/crypto.ts
export function buildAad(method: string, bearer: string, requestId: string): Uint8Array<ArrayBuffer> {
  const enc = new TextEncoder()
  const parts = [method, bearer, requestId].map((p) => enc.encode(p))
  const total = parts.reduce((sum, part) => sum + 4 + part.length, 0)
  const out = new Uint8Array(total)
  const view = new DataView(out.buffer, out.byteOffset, out.byteLength)
  let offset = 0
  for (const part of parts) {
    view.setUint32(offset, part.length)
    offset += 4
    out.set(part, offset)
    offset += part.length
  }
  return out
}
```

> Wichtig: Der Pfad fließt **nicht** in die AAD ein (Proxy/Präfix-Unsicherheit). Methoden + Bearer + Request-Id reichen zum Binden – der AAD wird einmal pro Request berechnet und für Verschlüsselung **und** Entschlüsselung der Antwort verwendet.

### SSE-Sonderfall

Ein Stream hat keine Request-Id. Die `data:`-Zeilen eines SSE-Endpunkts werden mit einem **token-gebundenen** AAD verschlüsselt:

```ts
buildAad('GET', `Bearer ${token}`, '')
```

d. h. `"GET"`, `"Bearer <token>"`, leere Request-Id. Der Server baut denselben AAD in `events.rs`/`video.rs` (`build_aad("GET", &format!("Bearer {token}"), "")`).

---

## Die Funktionen von `api/crypto.ts` (Stand aktuell)

| Funktion | Zweck |
|----------|-------|
| `encryptionAvailable()` | Prüft, ob WebCrypto bereitsteht (sonst Fallback auf Plaintext + einmalige Warnung) |
| `newRequestId()` | Frische zufällige Request-ID (UUID ohne Bindestriche bzw. 32 Hex) |
| `buildAad(method, bearer, requestId)` | AAD-Bytes wie oben |
| `encryptJson(payload, aad)` | JSON → kompletter `{"__enc": ...}`-Request-Body-String |
| `decryptEnvelope(b64Combined, aad)` | base64url-Envelope → Klartext-String |
| `decryptBody(text, aad)` | Antworttext → Envelope? entschlüsseln : Passthrough; **wirft** bei unauthentischem Envelope |
| `decryptSseData(token, text)` | SSE-`data:`-Zeile mit Token-AAD entschlüsseln |

> **Entfernt:** `tryDecryptSseData` (alt). Verschlüsselungsfehler werden nicht mehr still geschluckt; `decryptBody` wirft, damit der Aufrufer nicht versehentlich Müll verarbeitet.

---

## Die Header im Überblick

| Header | Richtung | Vorgang |
|--------|----------|---------|
| `X-Enc: 1` | Request | Body ist ein Envelope (bzw. Antwort soll verschlüsselt werden) |
| `X-Req-Id` | Request | Frische, einmalige Request-ID (AAD + Anti-Replay) |
| `Authorization: Bearer …` | Request | Token (bleibt wie gehabt) |

| Fall | `X-Enc` gesetzt? | Begründung |
|------|------------------|-----------|
| JSON-Body (verschlüsselt) | **ja** | damit weiß der Server, dass er entschlüsseln muss |
| String-Body ohne JSON | nein | Blob/Text bleibt Plaintext – keine verschlüsselte Lüge (Bugfix) |
| Kein Body (GET/POST ohne Body) | ja | damit auch die **Antwort** verschlüsselt kommt |

> Der alte Stand verschickte `X-Enc: 1` auch dann, wenn der Body gar kein Envelope war – der Server hätte dann versucht, Plaintext als Envelope zu entschlüsseln.

---

## Verwendung in `api/client.ts`

```ts
const method = (options.method || 'GET').toUpperCase();
const requestId = newRequestId();
const bearer = accessToken ? `Bearer ${accessToken}` : '';
const aad = buildAad(method, bearer, requestId);   // EINMAL berechnen
headers['X-Req-Id'] = requestId;

if (canEncrypt) {
  if (body && typeof body === 'string') {
    const parsed = JSON.parse(body);               // nur bei echtem JSON
    if (parsed !== null) {
      body = await encryptJson(parsed, aad);        // -> {"__enc": "..."}
      headers['X-Enc'] = '1';
    }
  } else if (body === undefined || body === null) {
    headers['X-Enc'] = '1';                         // Antwort verschlüsselt
  }
}

const res = await fetch(`${API_ROOT}${resolvedPath}`, { ...options, headers, body, cache: 'no-store' });
const decrypted = await decryptBody(await res.text(), aad);

// decrypted ist entweder der Klartext oder (bei Envelope) das entschlüsselte JSON
```

Fehlerbehandlung:
- Antwort ist ein Envelope, lässt sich aber nicht authentisieren → `ApiError('Ungültige verschlüsselte Antwort', …)`.
- WebCrypto nicht verfügbar (kein `crypto.subtle`, z. B. unsicherer Kontext ohne HTTPS) → **eine** `console.warn`, danach Plaintext-Fallback.
- Verschlüsselung des Request-Bodys schlägt fehl → `ApiError('Anfrage konnte nicht verschlüsselt werden', 0)` (nie still unverschlüsselt senden).

---

## SSE-Entschlüsselung

Die eigentliche SSE-Verbindung läuft über `api/realtime.ts` (`openEncryptedStream`), nicht mehr über natives `EventSource`:

```ts
import { openEncryptedStream } from '../api/realtime';

const stream = openEncryptedStream('/api/events?enc=1', token, (plain) => {
  // plain ist die entschlüsselte JSON-Zeile
  const event = JSON.parse(plain);
}, () => announce());          // optional: onOpen-Callback

stream.close();                // beenden
```

- Token wandert im `Authorization`-Header (kein `?token=` im Query mehr → kein Leak in Logs/Historie). Der Server akzeptiert den Query-Token weiterhin als Fallback.
- Jede `data:`-Zeile wird intern über `decryptSseData(token, payload)` entschlüsselt (AAD = `"GET" | "Bearer <token>" | ""`).
- Bei Verbindungsverlust Reconnect mit Backoff (1 s → max 30 s, max 10 Versuche).
- Für eigene Streams: `decryptSseData(token, text)` direkt nutzen.

---

## Häufige Fehler

| Symptom | Ursache | Lösung |
|---------|---------|--------|
| `400 Could not decrypt request body` bei jedem Request | Server hat anderes `ENCRYPTION_SECRET` | Secrets angleichen |
| `400 Duplicate request identifier` | Gleiche `X-Req-Id` doppelt gesendet (Client-Fehler oder Replay) | Frische ID pro Request; Replay-Angriff abwehren |
| `OperationError` im Browser | IV/Envelope fehlerhaft oder AAD weicht ab | AAD exakt wie `buildAad` bilden; base64url korrekt |
| Client wirft „Ungültige verschlüsselte Antwort" | Antwort ist Envelope, Score aber: AAD (Bearer/Methode/Req-Id) der Antwort ≠ Request | AAD für Entschlüsselung = AAD der Anfrage |
| Login klappt, Rest nicht | Nur `VITE_ENC_SECRET` geändert, aber Build gecacht | `npm run build` neu |
| CRLF-/Sonderzeichen im Envelope | base64url statt base64 verwendet | `-`/`_`, keine Padding `=` |

---

## Deploy-Hinweis (Wire-Break!)

Das neue Schema (AAD-Bindung + `X-Req-Id`) ist ein **Breaking Change**: Ein altes Client-Build ohne `X-Req-Id` bekommt vom neuen Server bei verschlüsselten Responses `400`/Chiffre-Fehler, und ein alter Server versteht die neuen Requests nicht. **Client und Server gehören zusammen deployed.**