# Video-Meetings (WebRTC-Signaling)

> Verwaltung von Video-Meetings mit eingebautem WebRTC-Signaling. Kein externer Jitsi-Server nötig – das Backend fungiert als Signal-Relay.

---

## Übersicht

| Eigenschaft | Wert |
|-------------|------|
| Auth | `Bearer <token>` |
| Echtzeit | SSE `GET /video/signals/stream/{room}` |
| Speicherung | Meetings in DB, Signale nur im RAM (Broadcast) |
| Verschlüsselung | optional via `X-Enc: 1` Header |

---

## Endpunkte

### GET /video/meetings

Alle Meetings auflisten.

**Zugriff:** eingeloggt

**Query-Parameter:** keine

**Response (200):**
```json
[
  {
    "id": 1,
    "title": "Elternsprechtag",
    "description": "30 Min. Slots",
    "room_name": "abc123def456",
    "host_id": 5,
    "host_name": "Max Mustermann",
    "starts_at": "2026-09-20T09:00:00",
    "ends_at": "2026-09-20T17:00:00",
    "jitsi_server": "meet.jit.si",
    "is_active": true,
    "participant_count": 0,
    "created_at": "2026-09-15T10:00:00"
  }
]
```

---

### POST /video/meetings

Neues Meeting erstellen.

**Zugriff:** eingeloggt

**Request Body:**
```json
{
  "title": "Mathe-Nachhilfe",
  "description": "Wöchentlicher Termin",
  "starts_at": "2026-09-22T14:00:00",
  "ends_at": "2026-09-22T15:00:00",
  "jitsi_server": "meet.jit.si"
}
```

| Feld | Typ | Pflicht | Hinweis |
|------|-----|---------|---------|
| `title` | string | ja | |
| `description` | string | nein | |
| `starts_at` | datetime | ja | |
| `ends_at` | datetime | ja | muss > starts_at |
| `jitsi_server` | string | nein | Default: `meet.jit.si` |

**Response (201):** Meeting-Objekt mit generiertem `room_name`.

---

### PUT /video/meetings/{id}

Meeting aktualisieren.

**Zugriff:** eingeloggt (nur Host)

**Request Body:** wie POST (alle Felder optional)

**Response (200):** Aktualisiertes Meeting-Objekt.

---

### DELETE /video/meetings/{id}

Meeting löschen.

**Zugriff:** eingeloggt (nur Host)

**Response:** `204 No Content`

---

### GET /video/meetings/{id}/join

Dem Meeting beitreten. Gibt die Join-URL und den Room-Token zurück.

**Zugriff:** eingeloggt

**Response (200):**
```json
{
  "room_name": "abc123def456",
  "room_token": "a1b2c3d4e5f6...",
  "jitsi_server": "meet.jit.si",
  "join_url": "https://meet.jit.si/abc123def456#jwt=..."
}
```

> **Hinweis:** Der `room_token` wird aus `SHA-256(room_name:user_id)` abgeleitet und berechtigt zum Signal-Stream.

---

### GET /video/join/{room}

Meeting über geteilten Link beitreten.

**Zugriff:** eingeloggt

**Response:** wie `/video/meetings/{id}/join`

---

### POST /video/signals/{room}

WebRTC-Signalnachricht an alle Teilnehmer im Raum senden (Relay).

**Zugriff:** eingeloggt

**Request Body:**
```json
{
  "to": 42,
  "type": "offer",
  "sdp": "...",
  "candidate": null,
  "content": null
}
```

| Feld | Typ | Beschreibung |
|------|-----|-------------|
| `type` | string | `hello`, `offer`, `answer`, `ice`, `leave`, `chat` |
| `to` | int | Ziel-User-ID (optional – null = Broadcast an alle) |
| `sdp` | string | SDP-Deskriptor (bei offer/answer) |
| `candidate` | string | ICE-Kandidat (bei ice) |
| `content` | string | Textinhalt (bei chat) |

**Response (200):** `{"ok": true}`

> Das Backend leitet die Nachricht über den Broadcast-Channel weiter – sie wird nicht gespeichert.

---

### GET /video/signals/stream/{room}

SSE-Stream für WebRTC-Signale in einem bestimmten Raum.

**Zugriff:** eingeloggt (Query-Parameter: `token`, optional `enc=1`)

**Event-Format:**
```
data: {"room":"abc123","from":5,"from_name":"Max Mustermann","message":{"type":"offer","sdp":"..."}}
```

| Feld | Beschreibung |
|------|-------------|
| `room` | Raumname |
| `from` | Sender-User-ID |
| `from_name` | Sender-Name |
| `message` | `VideoSignalingMessage` (s.o.) |

> **Sicherheit:** Nur Nachrichten für den eingeloggten User werden geliefert (gefiltert nach `to`-Feld).

---

## Signal-Protokoll

```
Teilnehmer A                    Backend                     Teilnehmer B
     │                            │                              │
     │── POST /signals/{room} ───►│                              │
     │   type: "hello"            │                              │
     │                            │── SSE stream ───────────────►│
     │                            │   from: A, type: "hello"     │
     │                            │                              │
     │◄── SSE stream ────────────│◄── POST /signals/{room} ─────│
     │   from: B, type: "offer"  │    type: "offer", to: A      │
     │                            │                              │
     │── POST /signals/{room} ───►│                              │
     │   type: "answer", to: B   │── SSE stream ───────────────►│
     │                            │                              │
     │◄── POST /signals/{room} ──│◄── POST /signals/{room} ─────│
     │   type: "ice", to: A      │    type: "ice", to: A        │
     │                            │                              │
     │── POST /signals/{room} ───►│                              │
     │   type: "chat", content:  │── SSE stream ───────────────►│
     │   "Hallo!"                │                              │
```

---

## Signal-Typen

| Typ | Zweck | Felder |
|-----|-------|--------|
| `hello` | Neuer Teilnehmer meldet sich | – |
| `offer` | WebRTC-Angebot | `sdp`, optional `to` |
| `answer` | WebRTC-Antwort | `sdp`, `to` |
| `ice` | ICE-Kandidat | `candidate`, `to` |
| `leave` | Raum verlassen | – |
| `chat` | Textnachricht im Raum | `content` |

---

## Fehler

| Status | Grund |
|--------|-------|
| 400 | Ungültiger Request-Body |
| 401 | Kein Token / abgelaufen |
| 403 | Nur Host darf Meeting löschen/ändern |
| 404 | Meeting nicht gefunden |

---

## Hinweise

- **Kein externer Jitsi-Server nötig:** Das Backend leitet WebRTC-Signale weiter. Die Nutzer können sich aber auch über einen externen Jitsi-Server verbinden (`jitsi_server` Feld).
- **Keine Medien-Speicherung:** Audio/Video läuft Peer-to-Peer – das Backend sieht/ hört nichts mit.
- **Raum-Token:** Wird aus `SHA-256(room_name:user_id)` abgeleitet und ist pro User unterschiedlich.
- **SSE-Streaming:** Der Signal-Stream nutzt denselben Mechanismus wie `/events` (Broadcast-Channel, Kapazität 256).
- **Kapazität:** Broadcast-Channel-Kapazität = 256. Bei mehr gleichzeitigen Teilnehmern können Pakete verloren gehen.
