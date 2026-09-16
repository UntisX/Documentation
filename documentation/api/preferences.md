# API: Benutzer-Präferenzen (`/preferences`)

> Persönliche Einstellungen des eingeloggten Benutzers (Theme, Akzentfarbe, Dashboard-Layout, Tutorial-Status).

---

## GET /preferences

**Eingeloggt.**

### Response (200)

```json
{
  "user_id": 7,
  "theme": "dark",
  "accent_color": "#3498db",
  "dashboard_layout": {
    "grid": ["stats", "timetable-week", "homework"],
    "bar": ["grades", "absences"]
  },
  "tutorial_seen": true
}
```

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `theme` | String | `light` / `dark` |
| `accent_color` | String | Hex `#RRGGBB` (Default `#3498db`) |
| `dashboard_layout` | Object | Widget-Layout (JSONB) – Form frei |
| `tutorial_seen` | Boolean | First-Run-Tour abgeschlossen |

---

## PUT /preferences

**Eingeloggt.** UPSERT – legt bei fehlendem Eintrag automatisch einen an.

```json
{
  "theme": "dark",
  "accent_color": "#e74c3c",
  "tutorial_seen": true
}
```

### Response (200)

`UserPreferencesResponse` (mit gespeicherten Werten).

---

## Frontend-Verwendung (ThemeContext)

```
1. Login
2. ThemeContext lädt GET /preferences
3. tutorial_seen → FeatureTour starten ja/nein
4. theme/accent_color → <html data-theme="dark"> + CSS-Variable --accent
5. Änderungen → PUT /preferences (atomar, zusammen mit Dashboard-Layout)
```

> Der Context speichert zusätzlich local (dirty-tracking), damit die lokalen Werte Vorrang vor Serverwerten haben.