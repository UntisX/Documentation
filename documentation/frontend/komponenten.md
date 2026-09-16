# Frontend: Komponenten

> Alle wiederverwendbaren React-Komponenten in `src/components/` – mit Zweck und wichtigsten Props.

---

## Übersicht

| Komponente | Zweck |
|------------|-------|
| `Layout` | App-Shell für eingeloggte Seiten |
| `Sidebar` | Rollenbasierte Navigation |
| `Breadcrumb` | Brotkrumen |
| `ProtectedRoute` | Route-Guard |
| `ErrorBoundary` | Render-Fehler abfangen |
| `Modal` | Overlay-Dialog |
| `ConfirmDialog` | Bestätigungsdialog |
| `PageHeader` | Seiten-Kopf |
| `Pagination` | Tabellen-Pager |
| `BatchToolbar` | Sammel-Editor |
| `EmptyState` | Leerer-Zustand |
| `Skeleton` | Ladeplatzhalter |
| `ExportMenu` | ICS/CSV/PDF-Menü |
| `FeatureTour` | Onboarding-Tour |
| `Notifications` | Glocke + Inbox |
| `SearchOverlay` | Globale Suche (Ctrl+K) |

---

## Layout

**Rolle:** Stellt `Sidebar` + `Breadcrumb` + `SearchOverlay` + `FeatureTour` + `<Outlet/>` bereit. Hält die **einzige globale SSE-Verbindung** (`useRealtime`) und übersetzt Events in Toasts.

**Realtime→Toast-Mapping:**

| SSE-kind | Toast |
|----------|-------|
| `substitution` | „Neue Vertretung in …" |
| `cancellation` | „Stunde fällt aus …" |
| `grade` | „Neue Note …" |
| `message` | „Neue Nachricht …" |
| `absence` | „Neue Fehlzeit …" |
| `homework` | „Neue Hausaufgabe …" |

**Props:** – (Kinder via react-router `<Outlet>`)

---

## Sidebar

**Rolle:** Navigationsleiste. Lädt `GET /settings/tabs` und zeigt pro Rolle nur die erlaubten Punkte. Dynamische Integrationen aus `integrations[]` werden als weiterführende Links gerendert.

**Elemente:**
- Gruppierte Einträge mit Hover-Flyout-Submenü
- Footer: Benutzerinfo, Glocke (Notifications), Einstellungen-Zahnrad, Logout
- Mobile: Hamburger-Menü

---

## ProtectedRoute

**Rolle:** Guard für eingeloggte/rollengebundene Routen.

**Props:**
| Prop | Typ | Bedeutung |
|------|-----|-----------|
| `roles` | `string[]` (optional) | Erlaubte Rollen |
| (Kinder) | `JSX.Element` | Geschützter Inhalt |

**Verhalten:**
1. Auth lädt → Spinner
2. nicht eingeloggt → `/login`
3. Rolle nicht erlaubt → `/dashboard`

---

## ErrorBoundary

Klassen-basiert. Fängt Render-Fehler ab, zeigt „Etwas ist schief gelaufen" + Reload-Button. Doppelt verschachtelt (main.tsx + App.tsx).

---

## Breadcrumb

Baut Pfad aus `location.pathname` mit `CRUMB_CONFIG`. Handelt dynamische Detailseiten (`/classes/:id` → „Details").

---

## Modal

| Feature | Wert |
|---------|------|
| Überlagerung | eigenes Overlay |
| Schließen | Esc / Klick außerhalb |
| Varianten | `wide` (breiteres Fenster) |

---

## ConfirmDialog

Bestätigung von kritischen Aktionen. Props: Message, `danger`-Style, Callbacks.

---

## PageHeader

Titel + optionaler Aktion-Slot (z.B. „Neu anlegen"-Button).

---

## Pagination

Seitennummern mit Ellipsen (`1 … 4 5 6 … 20`), Zurück/Weiter-Pfeile.

---

## BatchToolbar

Erscheint, wenn Zeilen selektiert sind:
- Auswahl aufheben
- Sammel-Löschen
- Sammel-Feldänderung (Text oder Select) → `PUT` je Zeile

---

## EmptyState

Für leere Listen: Icon + Titel + Beschreibung + optionaler Call-to-Action.

---

## Skeleton

Exportiert:
- `Skeleton` mit Varianten `text | circle | rect | card | row`
- `SkeletonTable` für tabellarische Platzhalter

---

## ExportMenu

Dropdown (ICS/CSV/PDF). Rendert nur Optionen, für die ein Handler existiert:
- Timetable → ICS + CSV + PDF (print)
- Grades/Absences → CSV
- Benutzer → PDF-Aktivierungsbriefe

---

## FeatureTour

| Schritt | Inhalt |
|---------|--------|
| 1 | Willkommen |
| 2 | Dashboard erklärt |
| 3 | Stundenplan |
| 4 | Suche (Ctrl+K) |
| 5 | Hilfe/Einstellungen |

Persistiert `tutorial_seen` via `GET/PUT /preferences` (Fallback localStorage).

---

## Notifications

- Pollt `GET /notifications` alle **15 Sekunden**
- Ungelesen-Badge am Glocken-Icon
- Flyout-Liste; `markRead` via `PUT /notifications/{id}/read`
- „Alle gelesen" via Schleife
- Neue ungelesene → Toast
- Typfilter im localStorage (`untisx_notif_types`)

---

## SearchOverlay

- Öffnen: `Ctrl`/`Cmd` + `K`
- Debounce: 250ms → `GET /search?q=`
- Gruppiert nach: Schüler/Lehrer/Fächer/Räume
- Hervorhebung der Treffer
- Tastennavigation: ↑ / ↓ / Enter / Esc
- Recent-Searches: `untisx_recent_searches`