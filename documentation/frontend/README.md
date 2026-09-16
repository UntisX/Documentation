# Frontend-Dokumentation

> Überblick, Verzeichnisstruktur, API-Client und alle Komponenten des offiziellen React-Frontends (`WebVersion/client`).

---

## Kurzprofil

| Aspekt | Wert |
|--------|------|
| **Name** | `untisx-client` 1.0.0 |
| **Stack** | React 18 · TypeScript 5 · Vite 5 · React Router 6 |
| **UI-Sprache** | Deutsch |
| **HTTP** | Natives `fetch` über `api/client.ts` (kein axios) |
| **State** | React Context + lokale Hooks (kein Redux/Zustand) |
| **Styling** | `src/styles/global.css` (≈4300 Zeilen, CSS-Variablen, kein Framework) |
| **Verschlüsselung** | AES-256-GCM via WebCrypto |
| **Realtime** | SSE über 2 Hooks (`useRealtime`, `useChatStream`) |
| **PDF** | `jspdf` + `qrcode` für Aktivierungsbriefe |

---

## Navigation

| Kapitel | Inhalt |
|---------|--------|
| [Verzeichnisstruktur](struktur.md) | Alle Dateien & Ordner |
| [API-Client](api-client.md) | `apiRequest`, Endpunkt-Mapping, Fehler, Tokens |
| [Verschlüsselung](verschluesselung.md) | AES-256-GCM im Detail |
| [Komponenten](komponenten.md) | Alle wiederverwendbaren UI-Bausteine |
| [Seiten & Routen](seiten.md) | Alle 34 Seiten mit Zweck |
| [State Management](state-management.md) | Contexts, Hooks, localStorage |

---

## Die große Idee

Das Frontend ist bewusst **fast ohne Abhängigkeiten** gebaut:

```
Ein fetch-Wrapper (api/client.ts)
  ↕ AES-256-GCM (api/crypto.ts)
  ↕ Pfad-Mapping (api/endpoints.ts)
  ↕ mappers (api/mappers.ts)
↕
   21 geteilte Komponenten
   30 Seiten
   3 Contexts + 2 Hooks
   1 globales CSS
```

**Für dein eigenes Frontend** kannst du dir den API-Client abschauen (siehe [API-Client](api-client.md)) und ihn 1:1 wiederverwenden – er ist frameworks-unabhängig.

---

## Provider-Hierarchie (main.tsx)

```
<React.StrictMode>
  <ErrorBoundary>
    <BrowserRouter>
      <AuthProvider>        ← Token, user, login/logout
        <ThemeProvider>     ← light/dark + accent
          <ToastProvider>   ← Toast-Benachrichtigungen
            <App />         ← Routen
```

---

## Routen-Übersicht (App.tsx)

| Pfad | Seite | Schutz |
|------|-------|--------|
| `/terminal` | Terminal (Super-Admin) | – |
| `/login` | Login | – |
| `/setup` | Setup (Bootstrap) | – |
| `/activate` | Aktivierung | – |
| `/infoscreen` | Öffentlicher Infoscreen | – |
| `/video/join/:room` | Video-Meeting beitreten (Link) | auth |
| `/dashboard` | Dashboard | auth |
| `/students` | Schüler | auth |
| `/timetable` | Planer | auth |
| `/stundenplan` | Stundenplan-Editor | admin |
| `/vertretungsplan` | Vertretungsplan | admin+teacher |
| `/absences` | Fehlzeiten | auth |
| `/chats` | Chats | auth |
| `/class-book` | Digitales Klassenbuch | admin+teacher |
| `/bookings` | Buchungssystem | auth |
| `/video` | Video-Konferenz | auth |
| `/grades` | Noten | admin+teacher+student |
| `/subjects` | Fächer | admin |
| `/teachers` | Lehrer | admin |
| `/admins` | Admins | admin |
| `/rooms` | Räume | admin |
| `/homework` | Hausaufgaben | auth |
| `/admin` | Admin-Panel | admin |
| `/admin/audit` | Audit-Log | admin |
| `/classes` | Klassen | auth |
| `/classes/:id` | Klassendetails | auth |
| `/settings` | Einstellungen | auth |
| `/moodle` | Moodle-Proxy | auth |
| `/help` | Hilfe | auth |
| `/` | → /login | – |
| `*` | 404 | – |