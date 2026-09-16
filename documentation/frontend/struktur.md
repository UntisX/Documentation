# Frontend: Verzeichnisstruktur

> Der komplette Dateibaum des offiziellen Clients (`WebVersion/client`).

---

## Überblick

```
client/
├── index.html                 Vite-Einstiegs-HTML (#root + /src/main.tsx)
├── package.json               Dependencies & Scripts
├── tsconfig.json              TypeScript-Konfiguration
├── tsconfig.node.json         Config für vite.config.ts
├── vite.config.ts             Vite-Config (Proxy, Obfuskierung, Port)
├── .env.example               Vorlage: VITE_ENC_SECRET
├── public/vite.svg            Favicon
├── dist/                      Build-Ausgabe (nach npm run build)
└── src/
    ├── main.tsx               React-Einstieg (Provider-Kette)
    ├── App.tsx                Routen-Definition
    ├── vite-env.d.ts          Env-Typing (VITE_ENC_SECRET)
    ├── api/
    │   ├── client.ts          fetch-Wrapper (Auth, Verschlüsselung, Fehler)
    │   ├── endpoints.ts       Pfad-Mapping-Tabelle + API_ROOT
    │   ├── crypto.ts          AES-256-GCM verschlüsseln/entschlüsseln
    │   └── mappers.ts         API-Antwort → UI-Objekte
    ├── components/            Wiederverwendbare UI-Bausteine (21 Dateien)
    ├── contexts/              AuthContext, ThemeContext, ToastContext
    ├── hooks/                 useRealtime, useChatStream (SSE)
    ├── pages/                 30 Seiten (1 je Route)
    ├── styles/global.css      Komplettes Styling
    └── utils/
        ├── format.ts          Deutsche Datums-/Fehler-Formatierung
        ├── navigation.ts      Rollen-basierte Startseite
        └── pdfGenerator.ts    jsPDF-Aktivierungsbriefe
```

---

## `src/api/` – Backend-Kommunikation

| Datei | Zweck |
|-------|-------|
| `client.ts` | `apiRequest<T>(endpoint, options)` – setzt Token, verschlüsselt, wirft `ApiError`, 401-Redirect |
| `endpoints.ts` | `API_ROOT='/api'` + `ENDPOINT_MAP` (Frontend-Pfad → Server-Pfad) + `resolveEndpoint()` |
| `crypto.ts` | `encryptJson`, `tryDecryptBody`, `tryDecryptSseData` |
| `mappers.ts` | `mapUser`, `mapGrade`, … (snake_case → camelCase) |

---

## `src/components/` – Geteilte Bausteine

| Datei | Zweck |
|-------|-------|
| `Layout.tsx` | App-Shell: Sidebar, Breadcrumb, SearchOverlay, SSE-Subscription, Outlet |
| `Sidebar.tsx` | Rollen-basierte Navigation (lädt `GET /settings/tabs`) |
| `ErrorBoundary.tsx` | Render-Fehler → freundlicher Reload-Screen |
| `ProtectedRoute.tsx` | Guard: lädt, gegen /login redirectet, rollen-geprüft |
| `Breadcrumb.tsx` | Brotkrumen-Pfad aus `location.pathname` |
| `FeatureTour.tsx` | 5-Schritte-Onboarding (persistiert via /preferences) |
| `Modal.tsx` | Reuse-Modal (Overlay, Esc, wide-Variante) |
| `ConfirmDialog.tsx` | Bestätigungsdialog (danger-UI) |
| `PageHeader.tsx` | Titel-Leiste mit optionaler Aktion |
| `Pagination.tsx` | Pagination mit Ellipsen |
| `BatchToolbar.tsx` | Sammel-Editor (Löschen, Feld-Massenänderung) |
| `EmptyState.tsx` | Platzhalter bei leerem Inhalt |
| `Skeleton.tsx` | Lade-Skeletons (text/circle/rect/card/row + SkeletonTable) |
| `ExportMenu.tsx` | Dropdown: ICS/CSV/PDF-Export |
| `Notifications.tsx` | Glocke + Flyout (pollt /notifications alle 15s) |
| `SearchOverlay.tsx` | Ctrl+K-Suche (`GET /search`) |

---

## `src/contexts/` – Globaler State

| Datei | Aufgabe |
|-------|---------|
| `AuthContext.tsx` | `user`, `loading`, Login/Logout, `/auth/me`, Session-Warnung |
| `ThemeContext.tsx` | `theme`, `accentColor`, lokale+Server-Persistenz |
| `ToastContext.tsx` | `addToast()`, Auto-Dismiss ~4,7s |

---

## `src/hooks/` – SSE-Hooks

| Hook | Zweck |
|------|-------|
| `useRealtime.ts` | Generische SSE-Verbindung `/api/events?token=…&enc=1` + Backoff-Reconnect |
| `useChatStream.ts` | Wie oben, aber `ChatEvent`-getypt (kind, conversation_id, …) |

---

## `src/pages/` – Alle Seiten

| Datei | Route | Funktion |
|-------|-------|----------|
| `Login.tsx` | `/login` | Login-Formular |
| `Setup.tsx` | `/setup` | Erster Admin (Bootstrap) |
| `Activate.tsx` | `/activate` | Aktivierungslink/QR |
| `Terminal.tsx` | `/terminal` | Super-Admin-Konsole (eigene API) |
| `Infoscreen.tsx` | `/infoscreen` | Öffentliche Anzeigetafel (kein Auth) |
| `Dashboard.tsx` | `/dashboard` | Drag&Drop-Widget-Grid |
| `Students.tsx` | `/students` | Schüler-CRUD |
| `Teachers.tsx` | `/teachers` | Lehrer-CRUD |
| `Admins.tsx` | `/admins` | Admin-CRUD |
| `Classes.tsx` | `/classes` | Klassenliste |
| `ClassDetail.tsx` | `/classes/:id` | Klasse + Schüler-Zuordnung |
| `Subjects.tsx` | `/subjects` | Fächer-CRUD |
| `Rooms.tsx` | `/rooms` | Räume-CRUD |
| `Timetable.tsx` | `/timetable` | Wochen-/Monatsplaner |
| `Stundenplan.tsx` | `/stundenplan` | Master-Stundeplan-Editor |
| `Vertretungsplan.tsx` | `/vertretungsplan` | Vertretungen/Ausfälle pflegen |
| `Absences.tsx` | `/absences` | Fehlzeiten + Heatmap |
| `Grades.tsx` | `/grades` | Noten-CRUD + Durchschnitt |
| `Homework.tsx` | `/homework` | Hausaufgaben |
| `Chats.tsx` | `/chats` | WhatsApp-artiger Chat |
| `ClassBook.tsx` | `/class-book` | Digitales Klassenbuch |
| `Bookings.tsx` | `/bookings` | Ressourcen-Buchungssystem |
| `VideoConference.tsx` | `/video` | WebRTC-Videokonferenz |
| `VideoJoin.tsx` | `/video/join/:room` | Meeting-Join per Link |
| `AdminPanel.tsx` | `/admin` | Settings, Tabs, Moodle, API-Keys |
| `AuditLog.tsx` | `/admin/audit` | Audit-Protokoll |
| `Settings.tsx` | `/settings` | Theme, Account, Passwort |
| `MoodlePage.tsx` | `/moodle` | iframe-Integration |
| `Help.tsx` | `/help` | FAQ + Glossar |
| `NotFound.tsx` | `*` | 404 |

---

## `src/utils/`

| Datei | Zweck |
|-------|-------|
| `format.ts` | Deutsche Fehlermeldungen/Datumsformatierung |
| `navigation.ts` | Startpfad pro Rolle |
| `pdfGenerator.ts` | Aktivierungsbrief als PDF (jsPDF + QR-Code) |

---

## `vite.config.ts` – wichtigste Details

```ts
plugins: [
  react(),
  obfuscator({ ... }),   // NUR bei Build, NUR für src/ (debugProtection, disableConsole)
]
server: {
  port: 5173,
  proxy: { '/api': { target: 'http://localhost:3000', rewrite: strip /api } }
}
preview: { port: 5173, proxy: identisch }
```

| Aspekt | Wert |
|--------|------|
| Dev-Port | 5173 |
| `allowedHosts` | `['.localhost']` |
| Proxy-Ziel | `http://localhost:3000` (Rust-Server) |
| Rewrite | `/api/…` → `/…` (Präfix wird entfernt) |
| Obfuskierung | nur `src/`, `node_modules` ausgeschlossen |