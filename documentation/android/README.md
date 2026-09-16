# Android-App (UntisX) – Dokumentation

> Native Android-App für UntisX mit Jetpack Compose, MVVM, Repository-Pattern und Echtzeit-Chat (SSE).
> Diese Doku ist – wie die API-Doku – in einzelne Kapitel pro Bereich aufgeteilt.

---

## Schnellüberblick

| Eigenschaft | Wert |
|-------------|------|
| Sprache | Kotlin 2.0 |
| UI | Jetpack Compose (Material 3) |
| Architektur | MVVM + Repository Pattern |
| DI | Hilt (Dagger) |
| Netzwerk | Retrofit 2 + OkHttp 4 |
| JSON | Moshi (Kotlin-Reflection) |
| Navigation | Navigation Compose |
| Persistenz | DataStore Preferences (keine DB) |
| Echtzeit | OkHttp SSE + 5s Polling-Fallback |
| Build | Gradle 8.7, AGP 8.5.2 |
| Min SDK | 29 (Android 10) |
| Target SDK | 35 |
| Berechtigungen | nur `INTERNET` + `ACCESS_NETWORK_STATE` |

---

## Inhaltsverzeichnis

### Grundlagen

| # | Seite | Inhalt |
|---|-------|--------|
| 1 | [Architektur](01-architektur.md) | Schichten, Datenfluss, Projektstruktur, Core-Patterns |
| 2 | [Auth & Login](02-auth-login.md) | Token-Management, DataStore-Keys, Multi-Account, Bootstrap |
| 3 | [API-Client & Netzwerk](03-api-client.md) | Retrofit/OkHttp-Setup, Interceptor, `/api`-Pfad, SSE, Endpunkt-Mapping |
| 4 | [Navigation & Theme](04-navigation-theme.md) | Routen, Bottom-Navigation, Start-Flow, Farben/Typografie |
| 5 | [Sicherheit](05-security.md) | TLS, Token-Speicherung, Network Security Config, Risiken |
| 6 | [Build & Setup](06-build-setup.md) | Gradle, Version Catalog, Befehle, Emulator-Hinweise |
| 7 | [Datenmodelle](07-datenmodelle.md) | Alle DTOs & Request/Response-Typen |

### Repositories

| # | Seite | Inhalt |
|---|-------|--------|
| 8 | [Repositories – Übersicht](repositories/README.md) | Aufruf-Pattern, `ApiResult`, Fehlerbehandlung |
| 9 | [AuthRepository](repositories/auth-repository.md) | Login, Logout, Activate, Bootstrap |
| 10 | [TimetableRepository](repositories/timetable-repository.md) | Stundenplan, Custom Events |
| 11 | [HomeworkRepository](repositories/homework-repository.md) | Hausaufgaben CRUD |
| 12 | [GradeRepository](repositories/grade-repository.md) | Noten + Durchschnitt |
| 13 | [MessageRepository](repositories/message-repository.md) | Nachrichten (Ordner) |
| 14 | [ChatRepository](repositories/chat-repository.md) | Chats, Nachrichten, Mitglieder |
| 15 | [AbsenceRepository](repositories/absence-repository.md) | Fehlzeiten + Statistik |
| 16 | [AdminRepository](repositories/admin-repository.md) | Benutzer, Fächer, Räume, Schule, Vertretungen, Audit |

### Screens / Features

| # | Seite | Inhalt |
|---|-------|--------|
| 17 | [Screens – Übersicht](screens/README.md) | Routen-Tabelle, Zugriffs-Steuerung |
| 18 | [Server-URL](screens/server-url.md) | Server-Setup beim ersten Start |
| 19 | [Login](screens/login.md) | Anmeldung, Fehler, Aktivierung |
| 20 | [Planner (Stundenplan)](screens/planner.md) | Wochenansicht, 10 Perioden |
| 21 | [Fehlstunden](screens/absences.md) | Fehlzeiten + Statistik-Karten |
| 22 | [Hausaufgaben](screens/homework.md) | Liste, Status-Badges, Erstellung |
| 23 | [Nachrichten / Chat](screens/messages.md) | Chat-Liste, Chat-Thread, SSE, Lese-Status |
| 24 | [Noten](screens/grades.md) | Noten-Liste, Fach-Durchschnitte |
| 25 | [Profil (Mehr)](screens/more.md) | Konto-Switcher, Einstellungen, Admin-Links |
| 26 | [Vertretungsplan](screens/vertretungsplan.md) | Tagesansicht, Vertretungen/Ausfälle |
| 27 | [Admin-Panel](screens/admin-panel.md) | Schuleinstellungen, Layout, Integration |
| 28 | [Audit-Protokoll](screens/audit-log.md) | Timeline-/Tabellen-Ansicht |
| 29 | [Schüler](screens/students.md) | Benutzer-Verwaltung + CSV-Import |
| 30 | [Lehrer](screens/teachers.md) | Benutzer-Verwaltung + CSV-Import |
| 31 | [Fächer](screens/subjects.md) | Fach-Verwaltung, Wochenstunden |
| 32 | [Räume](screens/rooms.md) | Raum-Verwaltung |
| 33 | [Klingelzeiten](screens/bell-schedule.md) | Zeitplan-Verwaltung (Placeholder) |

---

## Technologie-Stack (Detail)

| Komponente | Technologie | Version |
|------------|-------------|---------|
| Compose BOM | Jetpack Compose | 2024.06.00 |
| Material 3 | Design-System | (via BOM) |
| Hilt | Dependency Injection | 2.51.1 |
| Retrofit | HTTP-Client | 2.11.0 |
| OkHttp | Netzwerk + SSE | 4.12.0 |
| Moshi | JSON-Parsing | 1.15.1 |
| Navigation Compose | Screen-Navigation | 2.7.7 |
| DataStore | Lokale Persistenz | 1.1.1 |
| Coroutines | Async | 1.8.1 |

---

## Bekannte Einschränkungen (WIP)

- `CreateAbsenceDialog`: Bestätigen-Button ruft noch nicht die API auf
- `CreateHomeworkDialog`: Erstellung noch `// TODO`
- `BellScheduleScreen`: Nur Placeholder (Lade-Logik leer)
- `AdminPanelScreen`: Layout-/Integration-Tabs sind UI-only
- `GradeCard` Delete: Löscht nicht wirklich (nur Dialog schließen)
- **Kein Offline-Caching:** Daten werden bei jedem Screen-Laden neu geholt
- **Keine Push-Notifications:** Echtzeit nur via SSE/Polling im aktiven Screen
- **Kein Background-Service:** Keine Hintergrund-Verarbeitung