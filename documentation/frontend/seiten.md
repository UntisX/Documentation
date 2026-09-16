# Frontend: Seiten & Routen

> Alle 34 Seiten mit ihrer Funktion und den wichtigsten API-Aufrufen.

---

## Öffentliche Seiten

### Login — `/login`
- Formular `user_name` + `password`
- `POST /auth/login` → Token → localStorage `accessToken`
- danach `GET /auth/me` → Benutzer → Rollen-Landingpage

### Setup — `/setup`
- First-Run (Bootstrap)
- `POST /bootstrap` (mit Server-Bootstrap-Token)
- Feld: user_name, real_name, email, password

### Activate — `/activate`
- Aktivierung per Link oder QR-Code-Scan
- `POST /auth/activate`

### Terminal — `/terminal`
- Super-Admin-Konsole über **separate** Terminal-API (`/api/terminal/...`)
- Login, Statistiken, Schul-Approvals, User-Verwaltung, Logs
- Token: `terminal_token`

### Infoscreen — `/infoscreen`
- Öffentliche Anzeigetafel (kein Login nötig)
- Zeigt aktuelle Vertretungen, Lehrer-Abwesenheiten und Ausfälle
- `GET /timetable/vertretungsplan` (ohne Token)
- Ideal für große Monitore im Schulflur

---

## Dashboard — `/dashboard` (auth)

- Drag&Drop-Widget-Grid (12-Spalten-CSS-Grid)
- Widget-Typen: Statistiken, Fehlzeiten, nächste Stunde, Quick-Actions, ungelesene Nachrichten, Hausaufgaben, Wochenplan, Noten, Fächer, Räume, Klassen, Vertretungsplan, Heute, Klassen-Heatmap, Aktivität, Serverstatus
- Layout + Shortcut-Bar persistiert pro Benutzer via `GET/PUT /preferences` (dashboard_layout)
- lokal zusätzlich: `untisx_dash_layout_<userId>` / `untisx_dash_bar_<userId>`

---

## Stundenplan-Seiten

### Timetable ("Planer") — `/timetable` (auth)
- Wochen-Kachel (12 Perioden, 07:30–16:30) + Monatsansicht
- Live-Markierung der aktuellen Stunde
- Stundenausfälle und Vertretungen inline sichtbar
- Hausaufgaben je Stunde
- Custom-Events (CRUD + Wiederholungen, Konfliktprüfung)
- Export: ICS, CSV, PDF/print
- Filter: Lehrer (Admin), nur Änderungen

### Stundenplan (Editor) — `/stundenplan` (admin)
- Master-Stundenplan bearbeiten (Klassen × Wochentage × Perioden)
- `POST/PUT/DELETE /timetable`
- Wochen-Typ (ungerade/gerade) + Gültigkeitsfenster

### Vertretungsplan — `/vertretungsplan` (admin, teacher)
- Vertretungen/Ausfälle je Klasse pflegen
- `POST /timetable/substitutions`, `POST /timetable/{id}/cancel`
- Overlay „Lehrer-ABWESENHEIT" (teacher-absences)
- List/Grid-Ansichten, Filter „nur Änderungen" / „heute"
- Live via SSE

---

## Video & Buchung

### ClassBook (Digitales Klassenbuch) — `/class-book` (admin, teacher)
- Einträge pro Klasse/Fach/Datum mit Unterrichtsinhalt
- `GET/POST /class-book`, `PUT/DELETE /class-book/{id}`
- Entry-Types: `lesson`/`homework`/`test`/`project`/`other`
- Keine SSE-Events (Client poollt)

### Bookings (Buchungssystem) — `/bookings`
- Buchbare Ressourcen (Computer-Räume, Tablets, Beamer)
- Zeitbuchungen mit Konfliktprüfung
- `GET/POST /resources`, `GET/POST /bookings`
- `GET /bookings/check-conflict` für Echtzeit-Validierung beim Buchen
- Status: confirmed / cancelled

### VideoConference — `/video`
- WebRTC-Konferenzraum mit Meeting-Verwaltung
- `GET/POST /video/meetings`, `PUT/DELETE /video/meetings/{id}`
- Join per Link: `GET /video/join/{room}`
- Signaling via `POST /video/signals/{room}` + SSE-Stream

### VideoJoin — `/video/join/:room`
- Schnellleinstieg über geteilten Link
- `GET /video/join/{room}` → Meeting-Daten + Join-URL

---

## Noten & Fehlzeiten

### Grades — `/grades`
- `GET/POST /students/grades`, `GET /students/grades/average`
- Gewichtung + Typ (mündlich/Test/Klausur/Sonstiges)
- CSV-Export, deutsche Notenfarben (1–6)

### Absences — `/absences`
- `GET/POST /students/absences`, `GET /students/absences/stats`
- Monats-Heatmap-Kalender
- Batch-Operationen, CSV-Export

---

## Kommunikation

### Chats — `/chats`
- WhatsApp-artig: Konversationsliste, Nachrichtenstream, Read-Receipts, Tipp-Indikator, Gruppenverwaltung
- `useChatStream` bindet SSE-Events ein (`message`, `typing`, `chat_read`, `message_deleted`)
- Mitglieder-Picker über `GET /users`

---

## Verwaltung (Admin)

| Seite | Route | Funktion | Wichtigste Aufrufe |
|-------|-------|----------|--------------------|
| Schüler | `/students` | CRUD, CSV-Import, Batch, Aktivierungsbriefe | `GET/POST /users`, `POST /users/{id}/activation-reset` |
| Lehrer | `/teachers` | CRUD | `/users` (Rolle teacher) |
| Admins | `/admins` | CRUD | `/users` (Rolle admin) |
| Klassen | `/classes` | Liste + Detail (`/classes/:id` mit Schülerzuordnung) | `/school/classes` |
| Fächer | `/subjects` | CRUD | `/school/subjects` |
| Räume | `/rooms` | CRUD | `/school/rooms` |
| Admin-Panel | `/admin` | Schul-Settings, Tab-Sichtbarkeit, Moodle, Klingelzeiten, **API-Keys** | `/settings/tabs`, `/school/settings`, `/api-keys` |
| Audit-Log | `/admin/audit` | Read-only Audit, CSV-Export | `/administration/audit` |

---

## Persönliche Seiten

### Settings — `/settings`
- Theme (hell/dunkel) + Akzentfarbe
- Desktop/Sound-Benachrichtigungen
- E-Mail-Update (`PUT /users` / Präferenzen)
- Passwort ändern (`POST /auth/change-password`)

### Help — `/help`
- Statisches FAQ + Glossar

### MoodlePage — `/moodle`
- iframe über `GET /proxy?url=<integration.url>`

### NotFound — `*`
- 404-Seite

---

## Weinige wichtige Patterns (für eigene Frontends)

| Pattern | Umsetzung |
|---------|-----------|
| **Rollen-Landing** | `utils/navigation.ts`: admin→/dashboard, teacher→/dashboard, student→/dashboard |
| **Tabelle = CRUD** | jede Liste = Table + Search + Sort + Pagination + BatchToolbar + Export |
| **Detailseiten** | `/classes/:id` erstellt eigene Unter-API (Zuweisungen) |
| **CSV-Import** | Datei einlesen → je Zeile `POST` (kein Bulk-Endpoint) |
| **Aktivierungsbrief** | `pdfGenerator.ts`: jsPDF + `qrcode` → PDF mit Link/QR |
| **Suche** | Global `Ctrl+K` → `SearchOverlay` |
| **Breadcrumb** | immer: Root → Bereich → Item |