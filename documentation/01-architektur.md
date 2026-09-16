# Architektur

> Ein vollständiger Überblick über die UntisX-Plattform: wie Frontend, Backend und Datenbank zusammenarbeiten.

---

## Systemüberblick

```
┌─────────────────────────────────────────────────────────────────┐
│                         Browser / Client                       │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  WebVersion/client (React 18 + TypeScript + Vite 5)        │ │
│  │  - Seiten & Komponenten                                     │ │
│  │  - api/client.ts (fetch-Wrapper)                           │ │
│  │  - api/crypto.ts (AES-256-GCM Verschlüsselung)             │ │
│  │  - useRealtime / useChatStream (SSE)                       │ │
│  └───────────────────┬─────────────────────────────────────────┘ │
│                      │  HTTP (gleiche Origin: /api/*)           │
└──────────────────────┼──────────────────────────────────────────┘
                       │
                       ▼  Vite-Proxy: /api/* → localhost:3000/*
┌─────────────────────────────────────────────────────────────────┐
│                  UntisX-Server (Rust + Axum 0.8)               │
│  ┌──────────────────────┐  ┌──────────────────────────────────┐ │
│  │ server-basis (Framework)│  │  server-default (Implementierung)│ │
│  │ - Routen + Middleware  │  │  - SQLx Queries                 │ │
│  │ - Auth-Validierung     │  │  - Argon2 Passwort-Hashing      │ │
│  │ - AES-Entschlüsselung  │  │  - Event-Broadcast (tokio)      │ │
│  │ - CORS, Timeout, etc.  │  │  - Video-Signal-Broadcast       │ │
│  └──────────┬─────────────┘  │  - Session-Cleanup              │ │
│             │               └──────────────────────────────────┘ │
│             └───────────► Trait-Interface ◄────────────────────┘ │
│                         (Server Supertrait)                     │
│  AppState<S> ◄──────── Toml-Service                             │
│  ├── GET /events (SSE broadcast channel)                        │
│  ├── GET /video/signals/stream/{room} (SSE broadcast)           │
│  └── DB-Pool (PostgreSQL)                                       │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
              ┌────────────────────────────┐
              │  PostgreSQL 17 (Docker)     │
              │  - 40 Migrationen           │
              │  - 32 Tabellen              │
              └────────────────────────────┘
```

---

## Die drei Crate des Backends

Das Backend ist absichtlich in drei Teile aufgeteilt (Git-Submodule):

| Crate | Zweck | Inhalt |
|-------|-------|--------|
| **`api`** | Gemeinsame DTO-Typen | `LoginRequest`, `UserResponse`, `GradeResponse`, ... |
| **`server-basis`** | Framework (wiederverwendbar) | Router-Aufbau, Middleware, Validierung, Traits |
| **`server-default`** | Konkrete PostgreSQL-Implementierung | SQL-Queries, Business-Logik, Binary (`main.rs`) |

### Warum diese Trennung?

- Das **Framework** (`server-basis`) definiert, WELCHE Endpunkte es gibt und wie sie validiert werden.
- Die **Implementierung** (`server-default`) definiert, WAS passiert (SQL, Hashing, Events).
- Ein anderes Backend (z.B. mit einer anderen Datenbank) könnte das Framework erben und nur die Trait-Methoden neu implementieren.
- Das `Server`-Trait (in `server-basis/src/server.rs`) fordert **35 Service-Traits** – wer alle implementiert, hat ein vollständiges UntisX-Backend.

---

## Der Anfrage-Ablauf im Detail

```
1. Client → Server:  POST /api/auth/login  (Header: X-Enc: 1)
2. Vite-Proxy:       /api String entfernen → POST /auth/login
3. axum-Router:      läuft durch Middleware-Stack
   a. CORS-Layer
   b. RequestBodyLimitLayer (16 MB)
   c. TimeoutLayer (30 Sek.)
   d. encryption_middleware → AES-256-GCM entschlüsseln (wenn X-Enc: 1)
4. Handler `login<S>()`:
   - Validierung (validate_string für user_name/password)
   - argon2::verify_password gegen DB-Hash
   - 32 Zufallsbytes generieren → base64url Token
   - SHA-256 Hash in `sessions`-Tabelle speichern
   - Token im JSON-Body zurückgeben (wird automatisch verschlüsselt)
5. Client:           entschlüsselt Antwort, speichert Token
```

---

## Frontend-Architektur

```
src/
├── main.tsx              → React-Root, Provider-Kette
├── App.tsx               → Routen-Definition
├── api/                  → Backend-Kommunikation
│   ├── client.ts         → apiRequest()-Wrapper (Token, Verschlüsselung, Fehler)
│   ├── endpoints.ts      → Pfad-Mapping Frontend→Backend + API_ROOT
│   ├── crypto.ts         → AES-256-GCM verschlüsseln/entschlüsseln
│   └── mappers.ts        → API-Antwort → UI-Datenmodell
├── components/           → Wiederverwendbare UI-Bausteine (22 Dateien)
├── contexts/             → Auth, Theme, Toast (React Context)
├── hooks/                → useRealtime, useChatStream (SSE)
├── pages/                → 1 Datei pro Route (30 Dateien)
├── styles/global.css     → Komplette CSS (kein Framework)
└── utils/                → format, navigation, pdfGenerator
```

### Wichtigste Konzepte

| Konzept | Implementierung |
|---------|-----------------|
| **Kein** Redux / Zustand | React Context + lokaler State pro Seite |
| **Kein** axios | Natives `fetch` in `api/client.ts` |
| API-Basis | Immer same-origin `/api` (Proxy entfernt Präfix) |
| Verschlüsselung | AES-256-GCM, Schlüssel = SHA-256(`VITE_ENC_SECRET`) |
| Realtime | `EventSource('/api/events?token=...&enc=1')` |
| Persistenz | localStorage + Server via `/preferences` |

---

## Rollen und Berechtigungsmodell

| Rolle | Kann | Seiten |
|-------|------|--------|
| **admin** | Alles: Benutzer anlegen, Stundenplan editieren, API-Keys, Audits | Alle Seiten |
| **teacher** | Vertretungsplan pflegen, Noten vergeben, Hausaufgaben | Dashboard, Timetable, Vertretungsplan, Grades, Chats |
| **student** | Eigene Daten sehen (eigene Noten/Fehlzeiten), Chats | Dashboard, Grades, Absences, Chats |

Berechtigungspyramide im Backend:
```
validate_user(token)      → Erfolg/Optional<User> (Jeder eingeloggte Benutzer)
validate_admin(token)     → 401 wenn nicht eingeloggt, 403 wenn nicht admin
validate_api_key(...)     → Über X-Api-Key Header + Scopes
```

---

## Realtime-Architektur

```
                  tokio::sync::broadcast::Sender<ChannelEvent> (Kapazität 512)
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
   chats.rs              events.rs              notifications…
   (publish_events)      (SSE-Endpoint /events)
        │                       │
   ┌────▼────┐            ┌─────▼─────┐
   │ DB-Insert│            │ Filter nach│
   │ ChatMsg │            │ target_user │
   └─────────┘            └─────┬─────┘
                                │
                        SSE-Verbindung zum Client
                        stream: data: {"kind":"message", ...}
```

- Jeder Client hält eine offene SSE-Verbindung zu `/events`.
- Chat-Aktionen senden Events auf den Broadcast-Channel.
- Der SSE-Handler filtert nach `target_user_ids` – nur relevante Empfänger erhalten das Update.
- Bei `enc=1` wird jede `data:`-Zeile mit AES-256-GCM verschlüsselt.

### Video-Signaling (zweiter Broadcast-Channel)

```
        tokio::sync::broadcast::Sender<VideoSignalBroadcast> (Kapazität 256)
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
  video.rs               video/signals/stream/{room}
  (POST /video/signals)  (SSE-Endpoint, raumbezogen)
        │                       │
   ┌────▼────┐            ┌─────▼─────┐
   │ Relay   │            │ Filter nach│
   │ to Room │            │ room + user│
   └─────────┘            └─────┬─────┘
                                │
                      SSE-Verbindung zum Client
                      data: {"room": "...", "from": 5, "message": {...}}
```

- WebRTC-Signale werden **nicht** in der DB gespeichert – nur im RAM weitergeleitet.
- Jeder Teilnehmer hält eine separate SSE-Verbindung zu `/video/signals/stream/{room}`.
- Nachrichten mit `to:`-Feld werden nur an den Ziel-User zugestellt, Broadcasts an alle aktiven Verbindungen im Raum.
- Room-Token wird aus `SHA-256(room_name:user_id)` abgeleitet – jeder User hat seinen eigenen Token.

---

## Komponenten-System nach Außen

Für die Webseite/Entwickler wichtig:

| Aspekt | Wert |
|--------|------|
| Sprache | Deutsch (`de`) |
| Zeitzonen-Verarbeitung | UTC im Backend, `chrono` crate |
| IDs | `BIGSERIAL` (Ganzzahlen ab 1) |
| JSON-Feldbenennung | `snake_case` (z.B. `short_name` statt `shortName`) |
| Datumsformat | `YYYY-MM-DD` (ISO 8601) |
| Zeitformat | `HH:MM` (24h) |
| Error-Format | `{"message": "..."}` |
| Farben | Hex `#RRGGBB` |
| Rollennamen (exakt) | `admin`, `teacher`, `student` |
| Token-Lebensdauer | 7 Tage (604800 Sek.) |

> **Wichtig für eigene Frontends:** Überall `snake_case` verwenden! Das Frontend mappt die Felder über `api/mappers.ts` auf `camelCase`-UI-Objekte.