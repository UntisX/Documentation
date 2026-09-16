# API: Echtzeit (SSE) — `/events`

> Der Server-Sent-Events-Strom für Live-Updates.

---

## Endpunkt

```
GET /events?token=<bearer-token>&enc=1
```

- **`token`**: Pflicht, dein Bearer-Token (Authentifizierung). SSE kann keine `Authorization`-Header setzen.
- **`enc=1`**: optional – die `data:`-Zeilen sind AES-256-GCM-verschlüsselt (Envelope-Format).

### Response (200, Streaming)

```
data: {"kind":"message","title":"Neue Nachricht","description":"...",...}

data: {"kind":"substitution","title":"Neue Vertretung",...}
```

---

## Event-Schema

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `kind` | String | Event-Typ |
| `title` | String | Kurztitel |
| `description` | String | Detailtext |
| `created_at` | Timestamp | Zeitstempel |
| `conversation_id` | Number, optional | Nur Chat-Events |
| `message_id` | Number, optional | Nachrichten-ID |
| `sender_id` | Number, optional | Absender-ID |

---

## Alle `kind`-Werte

| kind | Bedeutung |
|------|-----------|
| `substitution` | Neue Vertretung |
| `cancellation` | Stundenausfall (neu/gelöscht) |
| `grade` | Neue Note |
| `message` | Neue Chat-Nachricht |
| `absence` | Neue Fehlzeit |
| `homework` | Neue Hausaufgabe |
| `message_deleted` | Nachricht gelöscht |
| `chat` | Chat geändert (Mitglieder, Name) |
| `typing` | Tipp-Indikator |
| `chat_read` | Gelesen-Markierung |

---

## Verbindungs-Parameter auf einen Blick

| Parameter | Wert | Beschreibung |
|-----------|------|--------------|
| `token` | `string` | Pflicht – Bearer-Token als Query-Parameter |
| `enc` | `1` | Optionale Ende-zu-Ende-Verschlüsselung |

---

## Vertrauensmodell

- Der Server filtert über `target_user_ids` pro Event – du bekommst NUR Events, die dich betreffen.
- Keep-Alive-Pings kommen automatisch (~alle 30s) via axum `KeepAlive::default()`.

---

## Backup für Clients

- Wenn die SSE-Verbindung >512 Events verpasst (Kanal voll) oder unterbrochen ist, hol dir den aktuellen Stand über REST (z.B. `GET /chats`, `GET /timetable/vertretungsplan`).
- Frontend-Beispiel siehe [Realtime/SSE-Konzept](../08-realtime-sse.md).