# API: Benachrichtigungen (`/notifications`)

> Systembenachrichtigungen (z.B. neue Noten, Vertretungen) mit ungelesen-Zähler.

---

## GET /notifications

**Eingeloggt.** Liste der letzten Benachrichtigungen (Limit: 50).

### Response (200)

```json
{
  "notifications": [
    {
      "id": 33,
      "type": "grade",
      "title": "Neue Note in Mathematik",
      "message": "Du hast in Mathematik die Note 2 erhalten.",
      "read": false,
      "entity_type": "grade",
      "entity_id": 18,
      "created_at": "2026-09-13T10:00:00Z"
    }
  ],
  "unread_count": 1
}
```

| Feld | Beschreibung |
|------|--------------|
| `type` | z.B. `grade`, `message`, `substitution`, `cancellation`, `absence`, `homework` |
| `entity_type` / `entity_id` | Verknüpfung zur Ressource (für Deep-Link) |
| `read` | Boolean |
| `unread_count` | Für Badge/Glöckchen |

---

## PUT /notifications/{id}/read

**Eingeloggt.** Einzelne Benachrichtigung als gelesen markieren.

### Response

`204 No Content`

---

## Weitere relevante Endpunkte (Frontend)

| Funktion | Endpunkt |
|----------|----------|
| Alle als gelesen markieren | wird im Client als Schleife über `PUT /notifications/{id}/read` umgesetzt |
| Benachrichtigungstypen filtern | Client-seitig (localStorage `untisx_notif_types`) |
| Live-Benachrichtigungen | über SSE `/events` (kinds `grade`, `message`, ...) |

---

## Frontend-Verhalten (Notifications.tsx)

1. Pollt `GET /notifications` **alle 15 Sekunden**
2. Zeigt Ungelesen-Badge am Glöckchen
3. Öffnet eine Flyout-Liste, einzelne Einträge → `PUT /notifications/{id}/read`
4. Neue ungelesene Einträge erzeugen Toast-Benachrichtigungen
5. Benachrichtigungs-Typen (notification types) werden in localStorage gefiltert (z.B. nur Noten + Vertretungen)