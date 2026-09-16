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

## Schnellsuche

Suche nach einem spezifischen Endpunkt oder Begriff:

### Nach HTTP-Methode

| Methode | Anzahl | Übersicht |
|---------|--------|-----------|
| **GET** | 32 | [Alle GET-Endpunkte →](09-api-suchindex.md#get-endpunkte) |
| **POST** | 18 | [Alle POST-Endpunkte →](09-api-suchindex.md#post-endpunkte) |
| **PUT** | 14 | [Alle PUT-Endpunkte →](09-api-suchindex.md#put-endpunkte) |
| **DELETE** | 12 | [Alle DELETE-Endpunkte →](09-api-suchindex.md#delete-endpunkte) |
| **PATCH** | 1 | [Alle PATCH-Endpunkte →](09-api-suchindex.md#patch-endpunkte) |

### Nach Zugriffsberechtigung

| Level | Beschreibung | Endpunkte |
|-------|-------------|-----------|
| **Öffentlich** | Kein Token nötig | `/health`, `/bootstrap`, `/auth/login`, `/auth/activate` |
| **Authentifiziert** | `Bearer <token>` | Alle anderen Endpunkte (Rolle egal) |
| **Nur Admin** | `Bearer` + Rolle `admin` | User-CUD, School-CUD, API-Keys, Audit-Stats |
| **API-Key** | `X-Api-Key` Header | `/external/*` (Read-only) |
| **Terminal** | Separates Token | `/terminal/*` (Super-Admin) |

---

## Zusammenfassung der Technologien

| Komponente | Technologie |
|------------|-------------|
| Frontend | React 18, TypeScript, Vite 5, React Router 6 |
| Backend | Rust, Axum 0.8, Tokio |
| Datenbank | PostgreSQL 17 via SQLx |
| Auth | Opaque Bearer Tokens (SHA-256 gehasht in DB) |
| Verschlüsselung | AES-256-GCM (WebCrypto Frontend / aes-gcm Rust) |
| Realtime | Server-Sent Events (SSE) |
| Passwörter | Argon2 Hashing |
| Deployment | Docker Compose (Multi-Stage) |
