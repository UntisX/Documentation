# API: Suche (`/search`)

> Globale Suche über Benutzer, Fächer und Räume – der Motor hinter dem Ctrl+K-Suchoverlay.

---

## GET /search

**Eingeloggt.** Query: `?q=<suchtext>`

### Response (200)

```json
{
  "students": [
    { "id": 12, "user_name": "s2001", "real_name": "Lina Meyer", "short_name": "LM", "class_id": 3, "class_name": "7a" }
  ],
  "teachers": [
    { "id": 7, "user_name": "mueller", "real_name": "Max Müller", "short_name": "MM" }
  ],
  "subjects": [
    { "id": 1, "name": "Mathematik", "short_name": "MATH", "color": "#FF5733" }
  ],
  "rooms": [
    { "id": 1, "name": "R202", "building": "A", "capacity": 30 }
  ]
}
```

- Kategorien: `students`, `teachers`, `subjects`, `rooms`.
- **Beschränkung: 20 Treffer pro Kategorie.**
- Matching: SQL `LIKE '%q%'` gegen Namen (`real_name` / `user_name` / `name` / `short_name`).

---

## Frontend-Verhalten (SearchOverlay)

- Tastenkombination: **Ctrl/Cmd + K**
- 250ms Debounce nach jedem Tastendruck → `GET /search?q=…`
- Ergebnisse werden nach Kategorie gruppiert dargestellt (Schüler, Lehrer, Fächer, Räume)
- Treffer werden im Text hervorgehoben (Highlight)
- Tastennavigation: Pfeile hoch/runter, Enter = öffnet Detailseite, Esc = schließen
- Letzte Suchbegriffe werden in localStorage (`untisx_recent_searches`) persistiert

---

## Hinweis für eigene Frontends

Die Suche liest NUR Benutzer, Fächer und Räume. Für Klassen/Hausaufgaben/Noten gibt es KEINE eigene Suche – du müsstest die jeweiligen Listen selbst filtern (clientseitig).