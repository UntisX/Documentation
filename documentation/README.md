# UntisX Dokumentation

> Vollständige Referenz für die UntisX-Plattform – erstellt für Entwickler, die ein eigenes Frontend oder Backend mit dem UntisX-System verbinden möchten.

---

## Inhaltsverzeichnis

| Kapitel | Beschreibung |
|---------|-------------|
| [Schnellstart](00-schnellstart.md) | Erste Schritte in 5 Minuten |
| [Architektur](01-architektur.md) | Gesamtarchitektur: Frontend, Backend, Datenbank |
| [Backend-Setup (Docker)](02-backend-setup.md) | Server installieren und konfigurieren |
| [Frontend-Setup (Vite)](03-frontend-setup.md) | Client installieren und starten |
| [Eigenes Frontend verbinden](04-eigenes-frontend.md) | Schritt für Schritt: Eigenes UI mit der API verbinden |
| [Eigenes Backend verbinden](05-eigenes-backend.md) | Schritt für Schritt: Eigenen Server mit UntisX verbinden |
| [Authentifizierung & Sicherheit](06-auth-sicherheit.md) | Token-System, API-Keys, AES-Verschlüsselung |
| [Datenbank-Schema](07-datenbank.md) | PostgreSQL-Tabellen und Beziehungen |
| [Realtime / SSE](08-realtime-sse.md) | Server-Sent Events für Live-Updates |
| [API-Suchindex](09-api-suchindex.md) | Alphabetischer Index aller Endpunkte (Suchsystem) |
| [API nach Bereich](10-api-nach-bereich.md) | Endpunkte gruppiert nach Funktionsbereich (Sortiersystem) |

---

## API-Referenz nach Bereich

| Bereich | Dokumentation |
|---------|---------------|
| Authentifizierung (`/auth/*`) | [api/auth.md](api/auth.md) |
| Bootstrap (`/bootstrap`) | [api/bootstrap.md](api/bootstrap.md) |
| Benutzer (`/users`) | [api/users.md](api/users.md) |
| Schule (`/school/*`) | [api/school.md](api/school.md) |
| Schüler & Noten (`/students/*`) | [api/students.md](api/students.md) |
| Stundenplan (`/timetable/*`) | [api/timetable.md](api/timetable.md) |
| Nachrichten (`/messages`) | [api/messages.md](api/messages.md) |
| Chats (`/chats`) | [api/chats.md](api/chats.md) |
| Benachrichtigungen (`/notifications`) | [api/notifications.md](api/notifications.md) |
| Kalender (`/custom-events`) | [api/custom-events.md](api/custom-events.md) |
| Suche (`/search`) | [api/search.md](api/search.md) |
| Einstellungen (`/settings/*`) | [api/settings-tabs.md](api/settings-tabs.md) |
| Verwaltung (`/administration/*`) | [api/administration.md](api/administration.md) |
| API-Schlüssel (`/api-keys`) | [api/apikeys.md](api/apikeys.md) |
| Präferenzen (`/preferences`) | [api/preferences.md](api/preferences.md) |
| SSE-Events (`/events`) | [api/sse-events.md](api/sse-events.md) |
| Extern (`/external/*`) | [api/external.md](api/external.md) |
| Proxy (`/proxy`) | [api/proxy.md](api/proxy.md) |
| Sync & Health | [api/sync-und-health.md](api/sync-und-health.md) |
| Fehlerbehandlung | [api/fehlerbehandlung.md](api/fehlerbehandlung.md) |
| Video-Meetings (`/video/*`) | [api/video.md](api/video.md) |
| Buchungssystem (`/resources` & `/bookings`) | [api/bookings.md](api/bookings.md) |
| Klassenbuch (`/class-book/*`) | [api/class-book.md](api/class-book.md) |

---

## Frontend-Dokumentation

| Kapitel | Beschreibung |
|---------|-------------|
| [Frontend-Architektur](frontend/README.md) | React + TypeScript + Vite Überblick |
| [Verzeichnisstruktur](frontend/struktur.md) | Alle Dateien und Ordner |
| [API-Client](frontend/api-client.md) | Wie das Frontend mit dem Backend kommuniziert |
| [Verschlüsselung](frontend/verschluesselung.md) | AES-256-GCM Implementierung |
| [Komponenten](frontend/komponenten.md) | Alle React-Komponenten |
| [Seiten](frontend/seiten.md) | Alle Routen und Seiten |
| [State Management](frontend/state-management.md) | Contexts, Hooks, localStorage |

---

## Android-Dokumentation

| Kapitel | Beschreibung |
|---------|-------------|
| [Android-App – Übersicht](android/README.md) | Inhaltsverzeichnis + Kennzahlen |
| [Architektur](android/01-architektur.md) | Schichten, Datenfluss, Projektstruktur |
| [Auth & Login](android/02-auth-login.md) | Token-Management, DataStore, Multi-Account, Bootstrap |
| [API-Client & Netzwerk](android/03-api-client.md) | Retrofit/OkHttp, Interceptor, alle Endpunkte, SSE |
| [Navigation & Theme](android/04-navigation-theme.md) | Routen, Bottom-Navigation, Start-Flow, Farben |
| [Sicherheit](android/05-security.md) | TLS, Token-Speicherung, Network Security, Risiken |
| [Build & Setup](android/06-build-setup.md) | Gradle, Version Catalog, Befehle |
| [Datenmodelle](android/07-datenmodelle.md) | Alle DTOs & Request/Response-Typen |
| [Repositories – Übersicht](android/repositories/README.md) | `ApiResult`-Pattern, Fehlerbehandlung |
| [Repositories im Detail](android/repositories/README.md#repository-liste) | Auth, Timetable, Homework, Grade, Message, Chat, Absence, Admin |
| [Screens – Übersicht](android/screens/README.md) | Routen-Tabelle, Rollen-Zugriff |
| [Screens im Detail](android/screens/README.md#routen-tabelle) | Server-URL, Login, Planner, Fehlstunden, Hausaufgaben, Chat, Noten, Profil, Vertretungsplan, Admin-Panel, Audit, Schüler, Lehrer, Fächer, Räume, Klingelzeiten |

---

## Schnellsuche

Suche nach einem spezifischen Endpunkt oder Begriff:

### Nach HTTP-Methode

| Methode | Anzahl | Übersicht |
|---------|--------|-----------|
| **GET** | 58 | [Alle GET-Endpunkte →](09-api-suchindex.md#get-endpunkte) |
| **POST** | 32 | [Alle POST-Endpunkte →](09-api-suchindex.md#post-endpunkte) |
| **PUT** | 20 | [Alle PUT-Endpunkte →](09-api-suchindex.md#put-endpunkte) |
| **DELETE** | 21 | [Alle DELETE-Endpunkte →](09-api-suchindex.md#delete-endpunkte) |
| **PATCH** | 1 | [Alle PATCH-Endpunkte →](09-api-suchindex.md#patch-endpunkte) |

### Nach Zugriffsberechtigung

| Level | Beschreibung | Endpunkte |
|-------|-------------|-----------|
| **Öffentlich** | Kein Token nötig | `/health`, `/bootstrap`, `/auth/login`, `/auth/activate`, `/class-book/infoscreen` |
| **Authentifiziert** | `Bearer <token>` | Alle anderen Endpunkte (Rolle egal) |
| **Nur Admin** | `Bearer` + Rolle `admin` | User-CUD, School-CUD, API-Keys, Audit-Stats |
| **API-Key** | `X-Api-Key` Header | `/external/*` (Read-only) |
| **Terminal** | Separates Token | `/terminal/*` (Super-Admin) |

---

## Zusammenfassung der Technologien

| Komponente | Technologie |
|------------|-------------|
| Frontend (Web) | React 18, TypeScript, Vite 5, React Router 6 |
| Android-App | Kotlin, Jetpack Compose, Retrofit, Hilt |
| Backend | Rust, Axum 0.8, Tokio |
| Datenbank | PostgreSQL 17 via SQLx |
| Auth | Opaque Bearer Tokens (SHA-256 gehasht in DB) |
| Verschlüsselung | AES-256-GCM (WebCrypto Frontend / aes-gcm Rust), AAD-gebunden an Methode + Bearer + `X-Req-Id` |
| Replay-Schutz | frische `X-Req-Id` pro Request, Server-Cache 300 s (Cap 50 000), doppelte ID → `400` |
| Realtime | Server-Sent Events (SSE), Token im `Authorization`-Header (`?token=` Fallback) |
| Passwörter | Argon2 Hashing |
| Deployment | Docker Compose (Multi-Stage) |
