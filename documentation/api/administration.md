# API: Verwaltung & Audit (`/administration`)

> Audit-Protokoll und -Statistiken für die Administration.

---

## GET /administration/audit

**Eingeloggt.**

Query-Parameter:

| Parameter | Typ | Beschreibung |
|-----------|-----|--------------|
| `limit` | Number | Seiten-Größe |
| `offset` | Number | Versatz für Pagination |
| `action` | String | **Angeboten, aber aktuell NICHT gefiltert** |
| `user_id` | Number | **Angeboten, aber aktuell NICHT gefiltert** |
| `from` / `to` | Datum | **Angeboten, aber aktuell NICHT gefiltert** |

### Response (200)

```json
{
  "entries": [
    {
      "id": 91,
      "user_id": 1,
      "action": "CREATE",
      "entity_type": "subject",
      "entity_id": 5,
      "details": "{\"name\":\"Deutsch\"}",
      "ip_address": "192.168.1.10",
      "created_at": "2026-09-13T10:00:00Z"
    }
  ],
  "total": 91
}
```

| Feld | Beschreibung |
|------|--------------|
| `action` | `CREATE` / `UPDATE` / `DELETE` |
| `entity_type` | z.B. `subject`, `user`, `grade`, `timetable_entry` |
| `details` | JSON-String |
| `total` | Gesamtzahl (für Pagination) |

> **Wichtig:** Nur `limit` und `offset` werden aktuell angewendet. `action`, `user_id`, `from`, `to` werden ignoriert (bekannte Einschränkung der Implementierung).

---

## GET /administration/audit/stats

**Nur Admin.**

### Response (200)

```json
[
  { "action": "CREATE", "count": 52 },
  { "action": "UPDATE", "count": 17 },
  { "action": "DELETE", "count": 4 }
]
```

Aggregation per `GROUP BY action, COUNT(*)`.

---

## Frontend-Verwendung

- `AuditLog.tsx` zeigt die Liste mit Pagination (`limit`/`offset`), Such- und Datumsfelder werden angeboten.
- CSV-Export wird client-seitig generiert.
- `AdminPanel`-Seite „Audit-Statistiken“ nutzt `/administration/audit/stats`.