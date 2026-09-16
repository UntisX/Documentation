# API: Nachrichten (`/messages`)

> Klassisches Order-Modell (Inbox/Sent) für Nachrichten zwischen Benutzern – getrennt vom Chat-System.

---

## GET /messages

**Eingeloggt.** Query: `?folder=inbox` (default) oder `?folder=sent`.

### Response (200)

```json
{
  "messages": [
    {
      "id": 42,
      "sender_id": 1,
      "receiver_id": 7,
      "subject": "Klassenfahrt",
      "body": "Bitte Abtretungen bis Freitag melden.",
      "priority": "normal",
      "read_at": null,
      "created_at": "2026-09-13T09:00:00Z"
    }
  ],
  "unread_count": 1
}
```

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `priority` | String | `low` / `normal` / `high` / `urgent` |
| `read_at` | Timestamp/null | null = ungelesen |
| `unread_count` | Number | Anzahl ungelesener im Ordner |

> **Bekannte Einschränkung:** `sender_first_name`, `sender_last_name`, `receiver_first_name`, `receiver_last_name` sind aktuell immer `null` (kein JOIN). Namen muss dein Frontend über `GET /users` selbst auflösen.

---

## POST /messages

**Eingeloggt.** Sender ist der Aufrufer.

```json
{
  "receiver_id": 7,
  "subject": "Klassenfahrt",
  "body": "Bitte Abtretungen bis Freitag melden.",
  "priority": "high"
}
```

### Response (201)

```json
{
  "id": 43,
  "sender_id": 1,
  "receiver_id": 7,
  "subject": "Klassenfahrt",
  "body": "Bitte Abtretungen bis Freitag melden.",
  "priority": "high",
  "read_at": null,
  "created_at": "2026-09-13T09:15:00Z"
}
```

---

## PUT /messages/{id}/read

**Eingeloggt.** Markiert die Nachricht (inbox) als gelesen.

### Response

`204 No Content`

---

## DELETE /messages/{id}

**Eingeloggt.** Nur **Empfänger (Owner)** oder **Sender** dürfen löschen. Sonst `404`.

### Response

`204 No Content`

---

## Frontend-Verhalten

- `Notifications.tsx` fragt `GET /notifications` im 15s-Intervall (nicht die Nachrichten).
- Der Kontakt/Verfasser wird über die Userliste aufgelöst.
- Für Umfragen/Prioritäten existiert ein `priority`-Feld – das Frontend kann Banner nach Priorität einfärben.