# API: Chats (`/chats`)

> WhatsApp-artiges Direkt- und Gruppen-Chat-System inkl. Read-Receipts, Tipp-Indikator, Mitgliederverwaltung und SSE-Push.

---

## GET /chats

**Eingeloggt.** Liste der Chats des Benutzers.

### Response (200)

```json
{
  "chats": [
    {
      "id": 3,
      "type": "group",
      "name": "Klasse 7a",
      "created_by": 1,
      "member_ids": [1, 7, 12],
      "last_message": "Okay, ich bin dabei!",
      "unread_count": 2,
      "created_at": "2026-09-01T08:00:00Z"
    },
    {
      "id": 4,
      "type": "direct",
      "name": null,
      "created_by": 7,
      "member_ids": [7, 12],
      "last_message": "Wann findet die Klausur statt?",
      "unread_count": 0,
      "created_at": "2026-09-02T09:00:00Z"
    }
  ],
  "total_unread": 2
}
```

| Feld | Beschreibung |
|------|--------------|
| `type` | `direct` / `group` |
| `name` | Nur bei Gruppen |
| `last_message` | Neueste Nachricht (Anzeige) |
| `unread_count` | Ungelesen für den Aufrufer |
| `total_unread` | Summe über alle Chats |

---

## POST /chats

**Eingeloggt.**

**Direkt-Chat:**

```json
{ "user_id": 7 }
```

**Gruppen-Chat:**

```json
{ "name": "Projektgruppe Bio", "member_ids": [7, 12, 15] }
```

### Response (201)

```json
{
  "id": 5,
  "type": "group",
  "name": "Projektgruppe Bio",
  "created_by": 1,
  "member_ids": [1, 7, 12, 15],
  "last_message": null,
  "unread_count": 0,
  "created_at": "2026-09-13T10:00:00Z"
}
```

---

## GET /chats/{id}

**Eingeloggt.** Details.

### Response (200)

```json
{
  "id": 5,
  "type": "group",
  "name": "Projektgruppe Bio",
  "created_by": 1,
  "members": [
    { "user_id": 1, "role": "admin",   "real_name": "Administrator" },
    { "user_id": 7, "role": "member",  "real_name": "Max Müller" }
  ],
  "created_at": "2026-09-13T10:00:00Z"
}
```

---

## Nachrichten

### GET /chats/{id}/messages

**Eingeloggt.** Query: `?before_id=&limit=` (1–100, Default 50).

### Response (200)

```json
{
  "messages": [
    {
      "id": 123,
      "conversation_id": 5,
      "sender_id": 7,
      "sender_real_name": "Max Müller",
      "body": "Schönen Nachmittag!",
      "created_at": "2026-09-13T10:05:00Z",
      "read_by_all": false
    }
  ],
  "has_more": true
}
```

| Feld | Beschreibung |
|------|--------------|
| `before_id` | Pagination: Nachrichten VOR dieser ID laden |
| `has_more` | Es existieren weitere ältere Nachrichten |
| `read_by_all` | Alle Mitglieder haben gelesen |

### POST /chats/{id}/messages

**Eingeloggt.**

```json
{ "body": "Schönen Nachmittag!" }
```

- `body`: 1–5000 Zeichen.
- Response (201) `ChatMessageResponse`.
- Löst SSE-Event `kind: message` an alle Mitglieder aus.

### DELETE /chats/{id}/messages/{message_id}

**Eingeloggt.** Nur **Absender**. Löst `kind: message_deleted` aus.

### Response `204`

---

## Read-Receipts

### POST /chats/{id}/read

**Eingeloggt.**

```json
{ "message_id": 123 }
```

Markiert alle Nachrichten bis einschließlich `message_id` als gelesen. Löst `kind: chat_read` aus.

> **Achtung:** Es gibt NUR diesen POST-Endpunkt (`/read`) – kein `/read` als PUT. Der offizielle Client nutzt ihn.

---

## Typing-Status

### POST /chats/{id}/typing

**Eingeloggt.** Kein Body. Löst `kind: typing` an alle Mitglieder aus. Body-Limit reicht; der Client zeigt „Person tippt …“ für kurze Zeit.

---

## Mitgliederverwaltung

### POST /chats/{id}/members

**Eingeloggt.** Hinzufügen:

```json
{ "user_ids": [16, 17] }
```

### PUT /chats/{id}/members

**Eingeloggt.** Ersetzt die komplette Mitgliederliste (Gruppen-Admin nötig):

```json
{ "user_ids": [1, 7, 12, 16] }
```

### DELETE /chats/{id}/members/{user_id}

**Eingeloggt.** Gruppen-Admin oder das Mitglied selbst.

### POST /chats/{id}/leave

**Eingeloggt.** Gruppen-Chat verlassen (funktioniert für Mitglieder).

### POST /chats/{id}/rename

**Eingeloggt.** Gruppen-Admin:

```json
{ "name": "Neuer Projektname" }
```

---

## Event-Typen (SSE `/events`)

| `kind` | Ausgelöst durch |
|--------|-----------------|
| `message` | Neue Chat-Nachricht |
| `message_deleted` | Nachricht gelöscht |
| `typing` | POST /typing |
| `chat_read` | POST /read |
| `chat` | Mitglieder/Name geändert |

---

## Frontend-Integration

Die `hooks/useChatStream.ts` öffnet eine zweite SSE-Verbindung (`EventSource('/api/events?token=…&enc=1')`) und entpackt Events wie:

```ts
{
  kind: 'message',
  conversation_id: 5,
  message_id: 124,
  sender_id: 7,
  title: 'Neue Nachricht',
  description: 'Max Müller: ...'
}
```

Praktisches Muster: `conversation_id` + `message_id` ⇒ Nachricht an der richtigen Stelle in die Liste einfügen.

---

## Limits & Regeln

| Regel | Wert |
|-------|------|
| `body` Länge | 1–5000 Zeichen |
| Pagination | `limit` 1–100 |
| Löschen | nur Absender |
| Mitglieder ändern | nur Gruppen-Admins (außer self-leave/self-remove) |
| Direkt-Chat | genau 2 Teilnehmer, `name=null` |