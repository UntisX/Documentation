# API nach Bereich (Sortiersystem)

> **Das Sortiersystem:** Alle Endpunkte hierarchisch nach Funktionsbereich sortiert – als Strukturbaum. Ideal zum Erforschen der API.

---

## Inhaltsverzeichnis der Bereiche

1. [Authentifizierung](#1-authentifizierung-auth)
2. [Bootstrap & System](#2-bootstrap--system)
3. [Benutzer](#3-benutzer-users)
4. [Schulressourcen](#4-schulressourcen-school)
5. [Schülerdaten](#5-schülerdaten-students)
6. [Stundenplan & Organisation](#6-stundenplan--organisation-timetable)
7. [Nachrichten](#7-nachrichten-messages)
8. [Chats](#8-chats)
9. [Benachrichtigungen](#9-benachrichtigungen-notifications)
10. [Kalender](#10-kalender-custom-events)
11. [Suche](#11-suche-search)
12. [Einstellungen & Tabs](#12-einstellungen--tabs-settings)
13. [Verwaltung & Audit](#13-verwaltung--audit-administration)
14. [API-Schlüssel](#14-api-schlüssel-api-keys)
15. [Benutzer-Präferenzen](#15-benutzer-präferenzen-preferences)
16. [Echtzeit (SSE)](#16-echtzeit-sse-events)
17. [Externe Schnittstelle](#17-externe-schnittstelle-external)
18. [Proxy & Integrationen](#18-proxy--integrationen)
19. [Sync](#19-sync)

---

## 1. Authentifizierung (`/auth`)

```
/auth
├── POST /activate          Aktivierungscode einlösen (öffentlich)
├── POST /login             Login → Token (öffentlich)
├── POST /logout            Session beenden (eingeloggt)
└── GET  /me                Eigenes Profil (eingeloggt)
```

→ [Ausführlich](api/auth.md)

---

## 2. Bootstrap & System

```
/bootstrap
├── POST /bootstrap         Ersten Admin anlegen (X-Bootstrap-Token)

/health
└── GET  /health            Server-Alive-Check (öffentlich)
```

→ [Bootstrap](api/bootstrap.md) · [Sync & Health](api/sync-und-health.md)

---

## 3. Benutzer (`/users`)

```
/users
├── GET    /users               Liste aller Benutzer
├── POST   /users               Benutzer anlegen (nur Admin)
├── PUT    /users/{id}          Benutzer ändern (nur Admin)
├── DELETE /users/{id}          Benutzer löschen (nur Admin)
└── POST   /users/{id}/activation-reset   Neuer Aktivierungscode
```

→ [Ausführlich](api/users.md)

---

## 4. Schulressourcen (`/school`)

```
/school
├── /subjects
│   ├── GET    /school/subjects
│   ├── POST   /school/subjects          (Admin)
│   ├── GET    /school/subjects/{id}
│   ├── PUT    /school/subjects/{id}     (Admin)
│   └── DELETE /school/subjects/{id}     (Admin, soft)
├── /rooms
│   ├── GET    /school/rooms
│   ├── POST   /school/rooms             (Admin)
│   ├── GET    /school/rooms/{id}
│   ├── PUT    /school/rooms/{id}        (Admin)
│   └── DELETE /school/rooms/{id}        (Admin, soft)
├── /classes
│   ├── GET    /school/classes
│   ├── POST   /school/classes           (Admin)
│   ├── GET    /school/classes/{id}
│   ├── PUT    /school/classes/{id}      (Admin)
│   └── DELETE /school/classes/{id}      (Admin, hard)
├── /bell-schedule
│   ├── GET    /school/bell-schedule
│   ├── POST   /school/bell-schedule     (Admin)
│   ├── GET    /school/bell-schedule/{id}
│   ├── PUT    /school/bell-schedule/{id} (Admin)
│   └── DELETE /school/bell-schedule/{id} (Admin)
└── /settings
    ├── GET    /school/settings
    ├── GET    /schools/self             (Alias)
    └── PATCH  /school/settings          (Admin)
```

→ [Ausführlich](api/school.md)

---

## 5. Schülerdaten (`/students`)

```
/students
├── /absences
│   ├── GET    /students/absences
│   ├── POST   /students/absences
│   ├── GET    /students/absences/stats
│   ├── PUT    /students/absences/{id}
│   └── DELETE /students/absences/{id}
├── /grades
│   ├── GET    /students/grades
│   ├── POST   /students/grades
│   ├── GET    /students/grades/average
│   ├── PUT    /students/grades/{id}
│   └── DELETE /students/grades/{id}
```

→ [Ausführlich](api/students.md)

---

## 6. Stundenplan & Organisation (`/timetable`)

```
/timetable
├── GET    /timetable                    Alle Einträge
├── POST   /timetable                    Eintrag anlegen (Admin)
├── PUT    /timetable/{id}               Eintrag ändern (Admin)
├── DELETE /timetable/{id}               Eintrag löschen (Admin)
├── /cancellations
│   ├── GET    /timetable/cancellations
│   └── POST   /timetable/{id}/cancel    Stunde ausfallen lassen
├── /substitutions
│   ├── GET    /timetable/substitutions
│   ├── POST   /timetable/substitutions
│   └── DELETE /timetable/substitutions/{id}   (storniert, soft)
├── /teacher-absences
│   ├── GET    /timetable/teacher-absences
│   ├── POST   /timetable/teacher-absences
│   └── DELETE /timetable/teacher-absences/{id}
├── /homework
│   ├── GET    /timetable/homework
│   ├── POST   /timetable/homework       (Lehrer/Admin)
│   ├── PUT    /timetable/homework/{id}  (Lehrer/Admin)
│   └── DELETE /timetable/homework/{id}  (Lehrer/Admin)
├── /vertretungsplan
│   └── GET    /timetable/vertretungsplan   Kombi-Ansicht
├── /month-data
│   └── GET    /timetable/month-data     Monats-Ansicht
└── /class-plan
    └── GET    /timetable/class-plan     Klassen-Plan
```

→ [Ausführlich](api/timetable.md)

---

## 7. Nachrichten (`/messages`)

```
/messages
├── GET    /messages?folder=inbox|sent
├── POST   /messages
├── PUT    /messages/{id}/read
└── DELETE /messages/{id}
```

→ [Ausführlich](api/messages.md)

---

## 8. Chats (`/chats`)

```
/chats
├── GET    /chats                        Liste
├── POST   /chats                        Neu (direkt oder Gruppe)
├── GET    /chats/{id}                   Details
├── GET    /chats/{id}/messages          Nachrichten (paginiert)
├── POST   /chats/{id}/messages          Senden
├── DELETE /chats/{id}/messages/{message_id}  Löschen (Absender)
├── POST   /chats/{id}/read              Gelesen markieren
├── POST   /chats/{id}/typing            Tipp-Status
├── POST   /chats/{id}/members           Mitglieder hinzufügen
├── PUT    /chats/{id}/members           Mitglieder (Admin)
├── DELETE /chats/{id}/members/{user_id} Mitglied entfernen
├── POST   /chats/{id}/leave             Chat verlassen
└── POST   /chats/{id}/rename            Umbenennen (Admin)
```

→ [Ausführlich](api/chats.md)

---

## 9. Benachrichtigungen (`/notifications`)

```
/notifications
├── GET  /notifications              Liste (max. 50)
└── PUT  /notifications/{id}/read    Gelesen
```

→ [Ausführlich](api/notifications.md)

---

## 10. Kalender (`/custom-events`)

```
/custom-events
├── GET    /custom-events?year=&month=
├── POST   /custom-events
├── PUT    /custom-events/{id}
└── DELETE /custom-events/{id}       (nur Eigentümer)
```

→ [Ausführlich](api/custom-events.md)

---

## 11. Suche (`/search`)

```
/search
└── GET /search?q=…                   Sucht Benutzer, Fächer, Räume
```

→ [Ausführlich](api/search.md)

---

## 12. Einstellungen & Tabs (`/settings`)

```
/settings
├── GET    /settings                  Tab-Übersicht
├── GET    /settings/tabs             Tab-Liste je Rolle
├── PUT    /settings/tabs             Tab-Konfiguration (Admin)
└── GET    /settings/moodle-url       Moodle-URL (stub)
```

→ [Ausführlich](api/settings-tabs.md)

---

## 13. Verwaltung & Audit (`/administration`)

```
/administration
└── /audit
    ├── GET    /administration/audit          Protokolle (limit/offset)
    └── GET    /administration/audit/stats    Aggregation (Admin)
```

→ [Ausführlich](api/administration.md)

---

## 14. API-Schlüssel (`/api-keys`)

```
/api-keys
├── GET    /api-keys
├── POST   /api-keys                  (gibt Key NUR 1× zurück)
├── PUT    /api-keys/{id}/toggle      Aktiv/deaktiviert
└── DELETE /api-keys/{id}
```

→ [Ausführlich](api/apikeys.md)

---

## 15. Benutzer-Präferenzen (`/preferences`)

```
/preferences
├── GET  /preferences
└── PUT  /preferences
```

→ [Ausführlich](api/preferences.md)

---

## 16. Echtzeit (SSE) (`/events`)

```
/events
└── GET /events?token=…&enc=1   Echtzeit-Ereignisstrom
```

→ [Ausführlich](api/sse-events.md) · [Konzept](08-realtime-sse.md)

---

## 17. Externe Schnittstelle (`/external`)

```
/external          (Auth: X-Api-Key, nur Lesen)
├── GET /external/classes
├── GET /external/subjects
├── GET /external/rooms
└── GET /external/bell-schedule
```

→ [Ausführlich](api/external.md)

---

## 18. Proxy & Integrationen

```
/proxy
└── GET /proxy?url=…   Lädt externe Seite und gibt sie als HTML zurück
```

→ [Ausführlich](api/proxy.md)

---

## 19. Sync

```
/sync
└── GET  /sync/versions    Versionen aller Ressourcen
```

→ [Ausführlich](api/sync-und-health.md)

---

## Legende

| Symbol | Bedeutung |
|--------|-----------|
| **(Admin)** | Nur Rolle `admin` |
| **(soft)** | Kein echtes Löschen, sondern Deaktivieren |
| **(hard)** | Echtes Löschen aus der Datenbank |
| **öffentlich** | Kein Token nötig |
| **eingeloggt** | Gültiger `Authorization: Bearer`-Header nötig |

---

## Gesamtzahlen

| Kategorie | GET | POST | PUT | PATCH | DELETE |
|-----------|-----|------|-----|-------|--------|
| Auth | 1 | 3 | – | – | – |
| Bootstrap/System | 1 | 1 | – | – | – |
| Users | 1 | 2 | 1 | – | 1 |
| School | 8 | 4 | 4 | 1 | 4 |
| Students | 5 | 2 | 2 | – | 2 |
| Timetable | 8 | 5 | 2 | – | 4 |
| Messages | 1 | 1 | 1 | – | 1 |
| Chats | 3 | 7 | 1 | – | 2 |
| Notifications | 1 | – | 1 | – | – |
| Custom-Events | 1 | 1 | 1 | – | 1 |
| Search | 1 | – | – | – | – |
| Settings | 3 | – | 1 | – | – |
| Administration | 2 | – | – | – | – |
| API-Keys | 1 | 1 | 1 | – | 1 |
| Preferences | 1 | – | 1 | – | – |
| SSE | 1 | – | – | – | – |
| External | 4 | – | – | – | – |
| Proxy | 1 | – | – | – | – |
| Sync | 1 | – | – | – | – |
| **Gesamt** | **45** | **27** | **16** | **1** | **16** |