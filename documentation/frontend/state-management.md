# Frontend: State Management

> Wie der offizielle Client State verwaltet – ohne externe State-Bibliothek.

---

## Philosophie

**Kein Redux, kein Zustand, kein Jotai.** Drei Mechanismen:

1. **React Context** für globale Anliegen (Auth, Theme, Toasts)
2. **Lokaler State pro Seite** (`useState`/`useEffect`), jede Seite lädt ihre Daten selbst
3. **localStorage** für Persistenz + **Server-Sync** über `/preferences`

---

## 1. Contexts

| Context | Eigenschaften | Persistenz |
|---------|---------------|-----------|
| `AuthContext` | `user`, `loading`, `tokenExpiresAt`, Login/Logout | `accessToken` (localStorage) |
| `ThemeContext` | `theme` ('light'/'dark'), `accentColor` | `untisx-theme`, `untisx_accent_color` + `/preferences` |
| `ToastContext` | `addToast({title, message, type, onClick})` | – (transient) |

### AuthContext im Detail

- Beim App-Start: Token aus localStorage lesen → `GET /auth/me` (bis zu 5 Retries bei transienten Fehlern)
- Bestätigter 401 → Session leeren
- `login(username, password)` → `POST /auth/login` (body `{user_name, password}`) → Token speichern → `/auth/me` laden
- `logout()` → `POST /auth/logout` + Tokens leeren
- `refreshUser()` → `/auth/me`
- `onSessionExpiryWarning(cb)` → Callback 5 Minuten vor Ablauf (TokenExpiry-Warnung)

### ThemeContext im Detail

- `data-theme`-Attribut + `--accent` CSS-Variable auf `<html>`
- Nach Login werden `GET /preferences` geladen, **nur wenn** lokal nicht schon geändert (Dirty-Tracking via Ref)
- Änderungen → `PUT /preferences` (atomar, zusammen mit dashboard_layout)

---

## 2. Datenfluss einer Seite (Beispiel: Grades)

```
1. useState<Grade[]>([])
2. useEffect(() => { apiRequest('/students/grades').then(setGrades) }, [])
3. lokale Filter/Sortierung/Pagination
4. Aktion (Neu/Speichern/Löschen) → apiRequest(...) → state aktualisieren
```

**Kein globaler Cache** – jede Seite ist unabhängig. Nach Änderungen holt sich die Seite selbst neu.

---

## 3. localStorage-Schema

| Key | Zweck | Sync mit Server |
|-----|-------|-----------------|
| `accessToken` | Bearer-Token | – |
| `untisx-theme` | Theme | ✔ `/preferences` theme |
| `untisx_accent_color` | Akzentfarbe | ✔ `/preferences` accent_color |
| `untisx_dash_layout_<userId>` | Widget-Grid-Layout | ✔ `/preferences` dashboard_layout.grid |
| `untisx_dash_bar_<userId>` | Shortcut-Bar | ✔ `/preferences` dashboard_layout.bar |
| `untisx_hw_completed` | „Erledigt"-Häkchen Hausaufgaben | ✘ (nur lokal) |
| `untisx_notif_types` | Aktive Benachrichtigungstypen | ✘ |
| `untisx_notification_settings` | Notifications-Details | ✘ |
| `untisx_recent_searches` | Letzte Suchbegriffe | ✘ |
| `untisx_tour_seen` | Tour gesehen | ✔ `/preferences` tutorial_seen |
| `untisx_stay_logged_in` | „Angemeldet bleiben" | ✘ |
| `terminal_token` | (Separat) Terminal-Login | – |

---

## 4. SSE-Hooks

| Hook | Beschreibung |
|------|--------------|
| `useRealtime` | Ruft `openEncryptedStream('/api/events?enc=1', token, …)` auf – fetch-basiert mit `Authorization`-Header, entschlüsselt Events, ruft `onEvent`. Backoff 1s→30s, max. 10 Versuche, `AbortController`. |
| `useChatStream` | Gleiche Mechanik, aber `ChatEvent`-getypt (`kind`, `conversation_id`, `message_id`, `sender_id`). |

---

## 5. Warum dieser Ansatz?

| Vorteil | Erklärung |
|---------|-----------|
| Wenige Abhängigkeiten | Kein Redux-Boilerplate, kleinere Bundle-Größe |
| Einfache Fehlersuche | Jede Seite unabhängig = klar abgegrenzter Scope |
| Server bleibt Single-Source | Wiederholet-Fetch statt Stale-Cache |
| Schneller Einstieg | Der Code ist fast barrierefrei lesbar |

| Nachteil / Achtung | Hinweis |
|--------------------|---------|
| Kein Caching | Jede Navigation = frische Requests |
| Kein optimistic update | Spinner bis zur Serverantwort |
| 1 globale SSE reicht | Für Chat-Events gibt es einen 2. Stream (größere Last) |

---

## 6. Empfehlungen für eigene Frontends

1. **Übernimm den API-Client** (`api/client.ts` + `crypto.ts`) 1:1 – er ist der stabile Vertrag.
2. **Locker coupling**: Halte Seiten-Data-Fetching lokal (wie original) oder füge einen `react-query`-Layer hinzu – die API erlaubt beides.
3. **Theme/Preferenzen**: Sofort an `GET/PUT /preferences` anbinden, damit Einstellungen reisebegleitet sind.
4. **Realtime**: Eine zentrale Verbindung im Root-Hook (wie `Layout`) reicht fürs Erste; Chats können den 2. Stream nutzen.