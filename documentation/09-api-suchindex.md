# API-Suchindex (alphabetisch)

> **Das Suchsystem:** Alle Endpunkte alphabetisch sortiert. Suche einen Pfad oder eine Methode – hier findest du ihn samt Kurzbeschreibung und Link zur Detailseite.

---

## Navigation nach Methode

- [Alle GET-Endpunkte](#get-endpunkte)
- [Alle POST-Endpunkte](#post-endpunkte)
- [Alle PUT-Endpunkte](#put-endpunkte)
- [Alle PATCH-Endpunkte](#patch-endpunkte)
- [Alle DELETE-Endpunkte](#delete-endpunkte)

---

## GET-Endpunkte

| # | Endpunkt | Zweck | Rolle | Detail |
|---|----------|-------|-------|--------|
| 1 | `GET /school/bell-schedule` | Klingelzeiten auflisten | eingeloggt | [school](api/school.md#bell-schedule) |
| 2 | `GET /school/bell-schedule/{id}` | Einzelne Klingelzeit | eingeloggt | [school](api/school.md#bell-schedule) |
| 3 | `GET /administration/audit` | Audit-Protokoll lesen | eingeloggt | [administration](api/administration.md) |
| 4 | `GET /administration/audit/stats` | Audit-Statistiken | **Admin** | [administration](api/administration.md) |
| 5 | `GET /api-keys` | API-Schlüssel auflisten | **Admin** | [apikeys](api/apikeys.md) |
| 6 | `GET /auth/me` | Eigenes Benutzerprofil | eingeloggt | [auth](api/auth.md) |
| 7 | `GET /chats` | Chat-Liste | eingeloggt | [chats](api/chats.md) |
| 8 | `GET /chats/{id}` | Chat-Details | eingeloggt | [chats](api/chats.md) |
| 9 | `GET /chats/{id}/messages` | Nachrichten eines Chats | eingeloggt | [chats](api/chats.md) |
| 10 | `GET /custom-events` | Eigene Kalendereinträge | eingeloggt | [custom-events](api/custom-events.md) |
| 11 | `GET /events` | SSE-Echtzeitstream (Auth: Authorization-Header bzw. `?token=` Fallback; `?enc=1` = verschlüsselte Zeilen) | eingeloggt | [sse-events](api/sse-events.md) |
| 12 | `GET /external/bell-schedule` | Klingelzeiten (extern) | API-Key | [external](api/external.md) |
| 13 | `GET /external/classes` | Klassen (extern) | API-Key | [external](api/external.md) |
| 14 | `GET /external/rooms` | Räume (extern) | API-Key | [external](api/external.md) |
| 15 | `GET /external/subjects` | Fächer (extern) | API-Key | [external](api/external.md) |
| 16 | `GET /health` | Health-Check | öffentlich | [sync-und-health](api/sync-und-health.md) |
| 17 | `GET /messages` | Nachrichten-Ordner laden | eingeloggt | [messages](api/messages.md) |
| 18 | `GET /notifications` | Benachrichtigungen | eingeloggt | [notifications](api/notifications.md) |
| 19 | `GET /preferences` | Benutzereinstellungen | eingeloggt | [preferences](api/preferences.md) |
| 20 | `GET /proxy` | Drittanbieter-Proxy | eingeloggt | [proxy](api/proxy.md) |
| 21 | `GET /school/classes` | Klassenliste | eingeloggt | [school](api/school.md#classes) |
| 22 | `GET /school/classes/{id}` | Einzelne Klasse | eingeloggt | [school](api/school.md#classes) |
| 23 | `GET /school/rooms` | Raumliste | eingeloggt | [school](api/school.md#rooms) |
| 24 | `GET /school/rooms/{id}` | Einzelner Raum | eingeloggt | [school](api/school.md#rooms) |
| 25 | `GET /school/settings` | Schuleinstellungen | eingeloggt | [school](api/school.md#settings) |
| 26 | `GET /school/subjects` | Fachliste | eingeloggt | [school](api/school.md#subjects) |
| 27 | `GET /school/subjects/{id}` | Einzelfach | eingeloggt | [school](api/school.md#subjects) |
| 28 | `GET /schools/self` | Eigenes Schulprofil (Alias) | eingeloggt | [school](api/school.md#settings) |
| 29 | `GET /search` | Globale Suche | eingeloggt | [search](api/search.md) |
| 30 | `GET /settings` | Tab-Übersicht | eingeloggt | [settings-tabs](api/settings-tabs.md) |
| 31 | `GET /settings/moodle-url` | Moodle-URL (stub) | eingeloggt | [settings-tabs](api/settings-tabs.md) |
| 32 | `GET /settings/tabs` | Tab-Konfiguration | eingeloggt | [settings-tabs](api/settings-tabs.md) |
| 33 | `GET /students/absences` | Fehlzeiten | eingeloggt | [students](api/students.md) |
| 34 | `GET /students/absences/stats` | Fehlzeiten-Statistik | eingeloggt | [students](api/students.md) |
| 35 | `GET /students/grades` | Notenliste | eingeloggt | [students](api/students.md) |
| 36 | `GET /students/grades/average` | Notendurchschnitt | eingeloggt | [students](api/students.md) |
| 37 | `GET /sync/versions` | Sync-Versionen | eingeloggt | [sync-und-health](api/sync-und-health.md) |
| 38 | `GET /timetable` | Stundenplanliste | eingeloggt | [timetable](api/timetable.md) |
| 39 | `GET /timetable/cancellations` | Ausfälle | eingeloggt | [timetable](api/timetable.md) |
| 40 | `GET /timetable/class-plan` | Klassen-Plan | eingeloggt | [timetable](api/timetable.md) |
| 41 | `GET /timetable/homework` | Hausaufgaben | eingeloggt | [timetable](api/timetable.md) |
| 42 | `GET /timetable/month-data` | Monatsdaten | eingeloggt | [timetable](api/timetable.md) |
| 43 | `GET /timetable/substitutions` | Vertretungen | eingeloggt | [timetable](api/timetable.md) |
| 44 | `GET /timetable/teacher-absences` | Lehrer-Abwesenheiten | eingeloggt | [timetable](api/timetable.md) |
| 45 | `GET /timetable/vertretungsplan` | Vertretungsplan (Kombi) | eingeloggt | [timetable](api/timetable.md) |
| 46 | `GET /users` | Benutzerliste | eingeloggt | [users](api/users.md) |
| 47 | `GET /class-book` | Klassenbucheinträge | eingeloggt | [class-book](api/class-book.md) |
| 48 | `GET /class-book/overview` | Klassenbuch-Übersicht | eingeloggt | [class-book](api/class-book.md) |
| 49 | `GET /class-book/infoscreen` | Öffentlicher Infoscreen | öffentlich | [class-book](api/class-book.md) |
| 50 | `GET /class-book/{id}` | Einzelner Eintrag | eingeloggt | [class-book](api/class-book.md) |
| 51 | `GET /resources` | Ressourcenliste | eingeloggt | [bookings](api/bookings.md) |
| 52 | `GET /bookings` | Buchungen | eingeloggt | [bookings](api/bookings.md) |
| 53 | `GET /bookings/overview` | Buchungs-Übersicht | eingeloggt | [bookings](api/bookings.md) |
| 54 | `GET /bookings/check-conflict` | Konfliktprüfung | eingeloggt | [bookings](api/bookings.md) |
| 55 | `GET /video/meetings` | Video-Meetings | eingeloggt | [video](api/video.md) |
| 56 | `GET /video/meetings/{id}/join` | Meeting beitreten | eingeloggt | [video](api/video.md) |
| 57 | `GET /video/join/{room}` | Meeting via Link beitreten | eingeloggt | [video](api/video.md) |
| 58 | `GET /video/signals/stream/{room}` | SSE-Signal-Stream | eingeloggt | [video](api/video.md) |

---

## POST-Endpunkte

| # | Endpunkt | Zweck | Rolle | Detail |
|---|----------|-------|-------|--------|
| 1 | `POST /api-keys` | API-Key anlegen | **Admin** | [apikeys](api/apikeys.md) |
| 2 | `POST /auth/activate` | Konto aktivieren | öffentlich | [auth](api/auth.md) |
| 3 | `POST /auth/login` | Einloggen | öffentlich | [auth](api/auth.md) |
| 4 | `POST /auth/logout` | Ausloggen | eingeloggt | [auth](api/auth.md) |
| 5 | `POST /bootstrap` | Ersten Admin anlegen | Bootstrap-Token | [bootstrap](api/bootstrap.md) |
| 6 | `POST /chats` | Chat erstellen | eingeloggt | [chats](api/chats.md) |
| 7 | `POST /chats/{id}/leave` | Chat verlassen | eingeloggt | [chats](api/chats.md) |
| 8 | `POST /chats/{id}/members` | Mitglieder hinzufügen | eingeloggt | [chats](api/chats.md) |
| 9 | `POST /chats/{id}/messages` | Nachricht senden | eingeloggt | [chats](api/chats.md) |
| 10 | `POST /chats/{id}/read` | Gelesen markieren | eingeloggt | [chats](api/chats.md) |
| 11 | `POST /chats/{id}/rename` | Chat umbenennen | eingeloggt | [chats](api/chats.md) |
| 12 | `POST /chats/{id}/typing` | Tipp-Status senden | eingeloggt | [chats](api/chats.md) |
| 13 | `POST /custom-events` | Kalendereintrag anlegen | eingeloggt | [custom-events](api/custom-events.md) |
| 14 | `POST /messages` | Nachricht senden | eingeloggt | [messages](api/messages.md) |
| 15 | `POST /school/bell-schedule` | Klingelzeit anlegen | **Admin** | [school](api/school.md#bell-schedule) |
| 16 | `POST /school/classes` | Klasse anlegen | **Admin** | [school](api/school.md#classes) |
| 17 | `POST /school/rooms` | Raum anlegen | **Admin** | [school](api/school.md#rooms) |
| 18 | `POST /school/subjects` | Fach anlegen | **Admin** | [school](api/school.md#subjects) |
| 19 | `POST /students/absences` | Fehlzeit anlegen | eingeloggt | [students](api/students.md) |
| 20 | `POST /students/grades` | Note vergeben | eingeloggt | [students](api/students.md) |
| 21 | `POST /timetable` | Stundenplaneintrag | **Admin** | [timetable](api/timetable.md) |
| 22 | `POST /timetable/homework` | Hausaufgabe anlegen | Lehrer/Admin | [timetable](api/timetable.md) |
| 23 | `POST /timetable/substitutions` | Vertretung anlegen | eingeloggt | [timetable](api/timetable.md) |
| 24 | `POST /timetable/teacher-absences` | Lehrer-Abwesenheit | eingeloggt | [timetable](api/timetable.md) |
| 25 | `POST /timetable/{id}/cancel` | Stunde ausfallen lassen | eingeloggt | [timetable](api/timetable.md) |
| 26 | `POST /users` | Benutzer anlegen | **Admin** | [users](api/users.md) |
| 27 | `POST /users/{id}/activation-reset` | Aktivierungslink neu | **Admin** | [users](api/users.md) |
| 28 | `POST /class-book` | Klassenbucheintrag erstellen | eingeloggt | [class-book](api/class-book.md) |
| 29 | `POST /resources` | Ressource anlegen | eingeloggt | [bookings](api/bookings.md) |
| 30 | `POST /bookings` | Buchung erstellen | eingeloggt | [bookings](api/bookings.md) |
| 31 | `POST /video/meetings` | Meeting erstellen | eingeloggt | [video](api/video.md) |
| 32 | `POST /video/signals/{room}` | Signal senden | eingeloggt | [video](api/video.md) |

---

## PUT-Endpunkte

| # | Endpunkt | Zweck | Rolle | Detail |
|---|----------|-------|-------|--------|
| 1 | `PUT /api-keys/{id}/toggle` | Key aktiv/deaktiviert | **Admin** | [apikeys](api/apikeys.md) |
| 2 | `PUT /chats/{id}/members` | Mitglieder ersetzen | eingeloggt | [chats](api/chats.md) |
| 3 | `PUT /custom-events/{id}` | Kalendereintrag ändern | eingeloggt | [custom-events](api/custom-events.md) |
| 4 | `PUT /messages/{id}/read` | Nachricht als gelesen | eingeloggt | [messages](api/messages.md) |
| 5 | `PUT /notifications/{id}/read` | Benachrichtigung gelesen | eingeloggt | [notifications](api/notifications.md) |
| 6 | `PUT /preferences` | Einstellungen speichern | eingeloggt | [preferences](api/preferences.md) |
| 7 | `PUT /school/bell-schedule/{id}` | Klingelzeit ändern | **Admin** | [school](api/school.md#bell-schedule) |
| 8 | `PUT /school/classes/{id}` | Klasse ändern | **Admin** | [school](api/school.md#classes) |
| 9 | `PUT /school/rooms/{id}` | Raum ändern | **Admin** | [school](api/school.md#rooms) |
| 10 | `PUT /school/subjects/{id}` | Fach ändern | **Admin** | [school](api/school.md#subjects) |
| 11 | `PUT /settings/tabs` | Tab-Konfiguration speichern | **Admin** | [settings-tabs](api/settings-tabs.md) |
| 12 | `PUT /students/absences/{id}` | Fehlzeit ändern | eingeloggt | [students](api/students.md) |
| 13 | `PUT /students/grades/{id}` | Note ändern | eingeloggt | [students](api/students.md) |
| 14 | `PUT /timetable/homework/{id}` | Hausaufgabe ändern | Lehrer/Admin | [timetable](api/timetable.md) |
| 15 | `PUT /timetable/{id}` | Stundenplaneintrag ändern | **Admin** | [timetable](api/timetable.md) |
| 16 | `PUT /users/{id}` | Benutzer ändern | **Admin** | [users](api/users.md) |
| 17 | `PUT /class-book/{id}` | Eintrag ändern | eingeloggt | [class-book](api/class-book.md) |
| 18 | `PUT /resources/{id}` | Ressource ändern | eingeloggt | [bookings](api/bookings.md) |
| 19 | `PUT /bookings/{id}` | Buchung ändern | eingeloggt | [bookings](api/bookings.md) |
| 20 | `PUT /video/meetings/{id}` | Meeting ändern | eingeloggt | [video](api/video.md) |

---

## PATCH-Endpunkte

| # | Endpunkt | Zweck | Rolle | Detail |
|---|----------|-------|-------|--------|
| 1 | `PATCH /school/settings` | Schuleinstellungen ändern | **Admin** | [school](api/school.md#settings) |

---

## DELETE-Endpunkte

| # | Endpunkt | Zweck | Rolle | Detail |
|---|----------|-------|-------|--------|
| 1 | `DELETE /api-keys/{id}` | API-Key löschen | **Admin** | [apikeys](api/apikeys.md) |
| 2 | `DELETE /chats/{id}/messages/{message_id}` | Nachricht löschen | Absender | [chats](api/chats.md) |
| 3 | `DELETE /chats/{id}/members/{user_id}` | Mitglied entfernen | Admin/Selbst | [chats](api/chats.md) |
| 4 | `DELETE /custom-events/{id}` | Kalendereintrag löschen | Eigentümer | [custom-events](api/custom-events.md) |
| 5 | `DELETE /messages/{id}` | Nachricht löschen | Owner/Sender | [messages](api/messages.md) |
| 6 | `DELETE /school/classes/{id}` | Klasse löschen | **Admin** | [school](api/school.md#classes) |
| 7 | `DELETE /school/rooms/{id}` | Raum deaktivieren | **Admin** | [school](api/school.md#rooms) |
| 8 | `DELETE /school/subjects/{id}` | Fach deaktivieren | **Admin** | [school](api/school.md#subjects) |
| 9 | `DELETE /students/absences/{id}` | Fehlzeit löschen | eingeloggt | [students](api/students.md) |
| 10 | `DELETE /students/grades/{id}` | Note löschen | eingeloggt | [students](api/students.md) |
| 11 | `DELETE /timetable/homework/{id}` | Hausaufgabe löschen | Lehrer/Admin | [timetable](api/timetable.md) |
| 12 | `DELETE /timetable/substitutions/{id}` | Vertretung stornieren | eingeloggt | [timetable](api/timetable.md) |
| 13 | `DELETE /timetable/teacher-absences/{id}` | Lehrer-Abwesenheit löschen | eingeloggt | [timetable](api/timetable.md) |
| 14 | `DELETE /timetable/{id}` | Stundenplaneintrag löschen | **Admin** | [timetable](api/timetable.md) |
| 15 | `DELETE /users/{id}` | Benutzer löschen | **Admin** | [users](api/users.md) |
| 16 | `DELETE /class-book/{id}` | Eintrag löschen | eingeloggt | [class-book](api/class-book.md) |
| 17 | `DELETE /resources/{id}` | Ressource deaktivieren | eingeloggt | [bookings](api/bookings.md) |
| 18 | `DELETE /bookings/{id}` | Buchung stornieren | eingeloggt | [bookings](api/bookings.md) |
| 19 | `DELETE /video/meetings/{id}` | Meeting löschen | eingeloggt | [video](api/video.md) |

> **Hinweis:** `DELETE /timetable/{id}/cancel` existiert als Route (Stundenausfall zurücknehmen) und erwartet optional `?date=YYYY-MM-DD`. <br>
> `PUT /settings/tabs` erfasst auch `/settings` (GET) im offiziellen Client – im Backend sind `/school/settings` und `/settings` zwei getrennte Konzepte.

---

## Index nach Schlüsselbegriff

| Suchbegriff | Endpunkte |
|-------------|-----------|
| Aktivierung | `POST /auth/activate`, `POST /users/{id}/activation-reset` |
| API-Key | `GET/POST /api-keys`, `PUT /api-keys/{id}/toggle`, `DELETE /api-keys/{id}` |
| Audit | `GET /administration/audit`, `GET /administration/audit/stats` |
| Ausfall | `GET/POST /timetable/cancellations`, `POST /timetable/{id}/cancel` |
| Benachrichtigung | `GET /notifications`, `PUT /notifications/{id}/read` |
| Chat | `GET/POST /chats`, `GET/POST /chats/{id}/…` |
| Einstellungen | `GET /settings`, `GET/PUT /settings/tabs`, `GET /settings/moodle-url`, `GET/PATCH /school/settings` |
| Extern | `GET /external/classes|subjects|rooms|bell-schedule` |
| Fehlzeit | `GET/POST /students/absences`, `PUT/DELETE /students/absences/{id}`, `GET /students/absences/stats` |
| Hausaufgabe | `GET/POST /timetable/homework`, `PUT/DELETE /timetable/homework/{id}` |
| Kalender | `GET/POST /custom-events`, `PUT/DELETE /custom-events/{id}` |
| Klasse | `GET/POST /school/classes`, `PUT/DELETE /school/classes/{id}`, `GET /timetable/class-plan` |
| Klingel | `GET/POST /school/bell-schedule`, `PUT/DELETE /school/bell-schedule/{id}` |
| Lehrkraft-Abwesenheit | `GET/POST /timetable/teacher-absences`, `DELETE /timetable/teacher-absences/{id}` |
| Login/Logout | `POST /auth/login`, `POST /auth/logout`, `GET /auth/me` |
| Nachricht | `GET/POST /messages`, `PUT /messages/{id}/read`, `DELETE /messages/{id}` |
| Note | `GET/POST /students/grades`, `PUT/DELETE /students/grades/{id}`, `GET /students/grades/average` |
| Präferenzen | `GET/PUT /preferences` |
| Proxy | `GET /proxy?url=…` |
| Raum | `GET/POST /school/rooms`, `PUT/DELETE /school/rooms/{id}` |
| Stundenplan | `GET/POST /timetable`, `PUT/DELETE /timetable/{id}`, `GET /timetable/month-data` |
| Suche | `GET /search?q=…` |
| Sync | `GET /sync/versions` |
| System | `GET /health`, `POST /bootstrap` |
| Vertretung | `GET/POST /timetable/substitutions`, `DELETE /timetable/substitutions/{id}` |
| Vertretungsplan | `GET /timetable/vertretungsplan` |
| Benutzer | `GET/POST /users`, `PUT/DELETE /users/{id}` |
| Buchung | `GET/POST /bookings`, `PUT/DELETE /bookings/{id}`, `GET /bookings/overview`, `GET /bookings/check-conflict`, `GET/POST /resources`, `PUT/DELETE /resources/{id}` |
| Klassenbuch | `GET/POST /class-book`, `GET /class-book/overview`, `GET /class-book/infoscreen`, `PUT/DELETE /class-book/{id}` |
| Meeting | `GET/POST /video/meetings`, `PUT/DELETE /video/meetings/{id}`, `GET /video/meetings/{id}/join`, `GET /video/join/{room}` |
| Video | `GET/POST /video/signals/{room}`, `GET /video/signals/stream/{room}` |

---

## So suchst du selbst

1. **Nach Pfad:** Endpunkte sind vollständig aufgelistet → Suche mit `Strg+F` nach `/timetable/homework` z.B.
2. **Nach Bereich:** [API nach Bereich (Sortiersystem)](10-api-nach-bereich.md) mit Strukturbaum
3. **Nach Rolle:** die Spalte „Rolle" zeigt dir, wer zugreifen darf
4. **Auf der Website:** Links führen direkt zu den Detailseiten mit Request-/Response-Schemas