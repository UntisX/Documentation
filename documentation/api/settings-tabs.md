# API: Einstellungen & Tabs (`/settings`)

> Sichtbarkeit von Frontend-Seiten je Rolle (Tabs), Integrationen und Moodle-URL.

---

## GET /settings

**Eingeloggt.** Kurzform der Tab-Übersicht.

### Response (200)

```json
{
  "tabs": {
    "admin": ["dashboard", "stundenplan", "vertretungsplan", "teachers", "students", "classes", "subjects", "rooms", "grades", "chats", "homework", "admin", "settings", "help"],
    "teacher": ["dashboard", "timetable", "vertretungsplan", "grades", "chats", "homework", "settings", "help"],
    "student": ["dashboard", "timetable", "grades", "absences", "chats", "homework", "settings", "help"]
  },
  "moodle_label": null,
  "integrations": []
}
```

---

## GET /settings/tabs

**Eingeloggt.** Gleiche Struktur wie `GET /settings`. Dient dem Sidebar-Aufbau im Frontend.

---

## PUT /settings/tabs — **Admin**

Speichert die komplette Tab-Konfiguration:

```json
{
  "tabs": {
    "admin": ["dashboard", "stundenplan", "vertretungsplan", "users", "settings"],
    "teacher": ["dashboard", "timetable", "chats"],
    "student": ["dashboard", "timetable", "grades", "chats"]
  },
  "moodle_label": "Moodle",
  "integrations": [
    {
      "id": "moodle",
      "name": "Moodle",
      "url": "https://moodle.schule.de",
      "icon": "https://moodle.schule.de/favicon.ico",
      "enabled": true,
      "roles": ["teacher", "student"]
    }
  ]
}
```

**Integrationen:** werden per Replace-all geschrieben (alle alten gelöscht, neue eingefügt).

### Response `200`

`SettingsTabsResponse`

---

## GET /settings/moodle-url

**Eingeloggt.** Aktuell ein **Stub**:

```json
{
  "enabled": false,
  "url": "",
  "tab_name": "Moodle"
}
```

---

## Frontend-Verwendung

- `Sidebar.tsx` ruft `GET /settings/tabs` und zeigt für die aktuelle Rolle nur die erlaubten Navigationspunkte.
- Dynamische Integrationen (aus `integrations`) werden als zusätzliche Nav-Links gerendert; `MoodlePage.tsx` zeigt die Integration per iframe über `/proxy`.
- Geänderte Konfiguration wird im AdminPanel über `PUT /settings/tabs` gespeichert.