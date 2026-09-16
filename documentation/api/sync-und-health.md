# API: Sync & Health

> Health-Check und Ressourcen-Versionierung für Offline-Sync/Abgleich.

---

## GET /health

**Öffentlich.**

### Response (200)

```
Server is alive
```

(Plaintext, kein JSON.) Ideal für Load-Balancer/Status-Seiten.

---

## GET /sync/versions

**Eingeloggt.**

### Response (200)

```json
{
  "resources": {
    "settings": 3,
    "subjects": 12,
    "rooms": 5,
    "classes": 8,
    "bell_schedule": 2,
    "timtable_entries": 40,
    "cancellations": 6,
    "substitutions": 2,
    "homework": 15,
    "grades": 80,
    ...
  }
}
```

| Feld | Beschreibung |
|------|--------------|
| `resources` | BTreeMap `ressourcenname → version` |

### Wofür das gut ist

- Offline-Apps können lokal versionieren und nur bei geänderter Version neu laden.
- Der Map-Key-Name variiert nach Schema (String max. 20 Zeichen).

---

## Details zur Implementierung

- Quelle: Tabelle `resource_versions` (`resource_name`, `version`, `updated_at`).
- Versionen werden bei relevanten Schreibvorgängen erhöht (wenn die Implementierung Instrumente nutzt).