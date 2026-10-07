# Realtime / SSE

> Die Server-Sent-Events-Schnittstelle für Live-Updates in der UntisX-Welt.

---

## Was wird unterstützt?

UntisX nutzt **Server-Sent Events (SSE)** – kein WebSocket. Eine SSE-Verbindung:

- ist eine **unidirektionale** Verbindung (Server → Client)
- kann beliebige HTTP-Header senden, wenn sie per `fetch` (Stream-Reader) aufgebaut wird

**Seit dem Security-Update:** Der offizielle Client baut SSE **fetch-basiert** auf (`api/realtime.ts`) und sendet das Token im **`Authorization`-Header** – nicht mehr im Query. Der Server bevorzugt den Header und akzeptiert `?token=` weiterhin als Fallback (Abwärtskompatibilität für Alt-Clients/Python).

---

## Endpunkt

```
GET /events?enc=1
Authorization: Bearer <token>
```

| Variante | Parameter / Header | Zweck |
|----------|--------------------|-------|
| `Authorization: Bearer <token>` | Header (bevorzugt) | Authentifizierung |
| `?token=<bearer-token>` | Query (Fallback) | für Clients ohne Header-Support (natives `EventSource`, Python) |
| `?enc=1` | Query, optional | die `data:`-Zeilen sind AES-256-GCM-verschlüsselt |

**Sicherheitsvorteil des Headers:** Das Token landet nicht mehr in Browser-Historie, Proxy-Logs oder Server-Access-Logs.

---

## Event-Format

Jede `data:`-Zeile ist ein JSON-Objekt:

```json
{
  "kind": "message",
  "title": "Neue Nachricht",
  "description": "Max Müller hat dir geschrieben",
  "created_at": "2026-09-13T10:30:00Z",
  "conversation_id": 42,
  "message_id": 123,
  "sender_id": 7
}
```

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `kind` | String | Event-Typ (s.u.) |
| `title` | String | Kurztitel |
| `description` | String | Detailtext |
| `created_at` | Timestamp | Zeitstempel |
| `conversation_id` | Number (optional) | Nur bei Chat-Events |
| `message_id` | Number (optional) | Nur bei Nachrichten-Events |
| `sender_id` | Number (optional) | Absender |

---

## Event-Typen (`kind`)

| kind | Bedeutung | Typische Daten |
|------|-----------|----------------|
| `substitution` | Neue Vertretung | timetable_entry_id |
| `cancellation` | Neue/Entfernte Stunde | timetable_entry_id |
| `grade` | Neue Note | grade_id |
| `message` | Neue Chat-Nachricht | conversation_id, message_id, sender_id |
| `absence` | Neue Fehlzeit | absence_id |
| `homework` | Neue Hausaufgabe | homework_id |
| `message_deleted` | Chat-Nachricht gelöscht | conversation_id, message_id |
| `chat` | Chat geändert (Mitglieder, Name) | conversation_id |
| `typing` | Benutzer tippt gerade | conversation_id, sender_id |
| `chat_read` | Nachrichten als gelesen markiert | conversation_id |

---

## Verwendung im Browser (offizieller Client)

### `openEncryptedStream` aus `api/realtime.ts` (empfohlen)

```ts
import { openEncryptedStream } from '../api/realtime';

const token = localStorage.getItem('accessToken');

// Token läuft im Authorization-Header, jede Zeile wird AAD-gebunden entschlüsselt
const stream = openEncryptedStream('/api/events?enc=1', token, (plain) => {
  const event = JSON.parse(plain);          // plain ist bereits entschlüsselt
  handleEvent(event);
}, () => console.log('verbunden'));

// später:
stream.close();
```

`openEncryptedStream` implementiert intern: fetch mit `Accept: text/event-stream` + `Authorization`-Header, `data:`-Zeilen-Parsing, Entschlüsselung je Zeile (`decryptSseData`), Backoff-Reconnect und `AbortController`-Close.

### Variante ohne Verschlüsselung (unkritische Anzeigen)

```js
const token = localStorage.getItem('accessToken');

const res = await fetch('/api/events', {
  headers: { Accept: 'text/event-stream', Authorization: `Bearer ${token}` },
});
const reader = res.body.getReader();
// … `data:`-Zeilen aus dem Stream parsen, JSON.parse aufs Plain-Event
```

### Native `EventSource` (nur mit Query-Fallback)

```js
const es = new EventSource(`/api/events?token=${token}&enc=1`);

es.onmessage = async (evt) => {
  const data = JSON.parse(await decryptSseData(token, evt.data));  // AES-256-GCM, Token-AAD
  handleEvent(data);
};

es.onerror = () => { /* EventSource reconnectet automatisch */ };
es.close();   // beim Verlassen
```

---

## Verwendung in Python / eigenen Diensten

```python
import json, requests

token = login()["token"]

# Bevorzugt: Authorization-Header
resp = requests.get(
    "http://localhost:3000/events?enc=1",
    headers={"Authorization": f"Bearer {token}"},
    stream=True, timeout=600,
)

for line in resp.iter_lines(decode_unicode=True):
    if line and line.startswith("data:"):
        data = line[5:]
        if '"__enc"' in data:                     # verschlüsselt?
            event = json.loads(decrypt_sse(data, token))   # eigene Decrypt + AAD-Implementierung
        else:
            event = json.loads(data)
        if event["kind"] == "message":
            print(f"Neue Nachricht von {event.get('sender_id')}")
```

> Ohne `enc=1` kommen die Zeilen als Klartext-JSON – für Lese-Integrationen einfachste Variante und weiterhin erlaubt (Verschlüsselung ist opt-in pro Stream).

---

## Verschlüsselung der SSE-Zeilen (AAD)

Jede verschlüsselte `data:`-Zeile ist ein einzelner Envelope `{"__enc": "…"}` – das gleiche Schema wie bei REST. Der AAD ist **token-gebunden** (kein Request-Id bei Streams):

```
AAD = build_aad("GET", "Bearer <token>", "")
```

⇒ Ein aufgezeichnetes Event kann nicht unter einer anderen Session entschlüsselt werden. Implementierung:
- Client: `decryptSseData(token, text)` in `api/crypto.ts`
- Server: `build_aad("GET", &format!("Bearer {token}"), "")` in `routs/events.rs` bzw. `routs/video.rs`

---

## Keep-Alive & Reconnect

- Axum sendet automatisch **Keep-Alive-Pings** (Standard ~30s).
- `openEncryptedStream` macht bei Verbindungsverlust **exponentielles Backoff**:
  - Start: 1 Sekunde, Maximum: 30 Sekunden, Maximum 10 Versuche, dann Abbruch.
- Verwendet in: `useRealtime`, `useChatStream` (Layout/Chat), `VideoCallRoom` (Signal-Stream).

---

## Backend-Implementierung (wie funktioniert's intern?)

```
server-default::main
   │
   ├── event_tx: broadcast::Sender<ChannelEvent>   (Kapazität 512)
   │
   ├── routs/events.rs
   │   │  GET /events?enc=1
   │   │  → Bearer aus Authorization-Header (Fallback: ?token=)
   │   │  → validate_user
   │   │  → event_tx.subscribe()
   │   │  → axum Sse stream
   │   │  → Filter: nur Events, deren target_user_ids den User enthält
   │   │  → wenn enc=1: jedes Event mit AAD build_aad("GET","Bearer <token>","") verschlüsseln
   │   └─ KeepAlive::default()
   │
   └── routs/chats.rs (Beispiel-Auslöser)
       │   POST /chats/{id}/messages
       │   → DB-Insert
       │   → publish_chat_event(&event_tx, ChannelEvent{...})
       │   → Broadcast an ALLE Subscriber
       │   → events.rs filtert für jeden Client
```

> Der Broadcast-Kanal hat Kapazität 512. Wenn ein Client zu langsam ist, werden alte Events übersprungen – deshalb: rechtzeitig verbinden oder Daten zusätzlich über REST holen.

---

## Best Practices für eigene Frontends

| Tipp | Begründung |
|------|-----------|
| Reconnect mit Backoff implementieren | Server/Datenbank haben Downtimes |
| Beim Verbindungsverlust nicht crashen | Ereignisse sind keine kritischen Daten – REST bleibt fallback |
| Nach Login sofort verbinden | Nichts verpassen |
| Events deduplizieren | Der Server kann Broadcasts nicht garantieren (z.B. bei >512 Events) |
| Token lieber im `Authorization`-Header | Kein Leak in Historie/Logs; `?token=` nur Fallback |
| `enc=1` nur mit korrektem Secret verwenden | Sonst unlesbarer Datensalat |