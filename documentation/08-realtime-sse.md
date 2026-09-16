# Realtime / SSE

> Die Server-Sent-Events-Schnittstelle für Live-Updates in der UntisX-Welt.

---

## Was wird unterstützt?

UntisX nutzt **Server-Sent Events (SSE)** – kein WebSocket. Eine SSE-Verbindung:

- ist eine **unidirektionale** Verbindung (Server → Client)
- liefert **automatische Reconnects** über die Browser-HTTP-Stack
- kann **kein Authorization-Header** senden → Token muss als **Query-Parameter** übergeben werden

---

## Endpunkt

```
GET /events?token=<bearer-token>&enc=1
```

| Parameter | Wert | Zweck |
|-----------|------|-------|
| `token` | Dein Bearer-Token | Authentifizierung (Pflicht) |
| `enc` | `1` (optional) | Die `data`-Zeilen sind AES-verschlüsselt |

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

## Verwendung im Browser

### Native `EventSource`

```js
const token = localStorage.getItem('accessToken');

const es = new EventSource(`/api/events?token=${token}&enc=1`);

es.onmessage = (evt) => {
  const data = JSON.parse(evt.data);          // ohne Verschlüsselung
  handleEvent(data);
};

es.onerror = () => {
  // EventSource reconnectet automatisch.
  // Nach mehreren Fehlern: Session prüfen, ggf. neu einloggen.
};
```

### Mit Verschlüsselung (wie im offiziellen Client)

```js
const es = new EventSource(`/api/events?token=${token}&enc=1`);

es.onmessage = async (evt) => {
  try {
    const parsed = await tryDecryptBody(evt.data);   // Entschlüsselung per AES-256-GCM
    handleEvent(parsed);
  } catch (e) { /* Verschlüsselungsfehler ignorieren */ }
};
```

---

## Verwendung im Python-Backend/Dienst

```python
import json, requests

token = login()["token"]
resp = requests.get(f"http://localhost:3000/events?token={token}", stream=True, timeout=600)

for line in resp.iter_lines(decode_unicode=True):
    if line and line.startswith("data:"):
        event = json.loads(line[5:])
        if event["kind"] == "message":
            print(f"Neue Nachricht von {event.get('sender_id')}")
```

---

## Keep-Alive & Reconnect

- Axum sendet automatisch **Keep-Alive-Pings** (Standard ~30s).
- Browser `EventSource` reconnectet bei Verbindungsverlust automatisch.
- Der offizielle Client (`hooks/useRealtime.ts`) macht **exponentielles Backoff**:
  - Start: 1 Sekunde
  - Maximum: 30 Sekunden
  - Maximum 10 Versuche, dann Abbruch

---

## Backend-Implementierung (wie funktioniert's intern?)

```
server-default::main
   │
   ├── event_tx: broadcast::Sender<ChannelEvent>   (Kapazität 512)
   │
   ├── routs/events.rs
   │   │  GET /events?token=… 
   │   │  → validate_user
   │   │  → event_tx.subscribe()
   │   │  → axum Sse stream
   │   │  → Filter: nur Events, deren target_user_ids den User enthält
   │   │  → wenn enc=1: jedes Event mit AES-GCM verschlüsseln
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
| `enc=1` nur mit korrektem Secret verwenden | Sonst unlesbarer Datensalat |