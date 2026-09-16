# Auth & Login

> Authentifizierung und lokale Session-Verwaltung der Android-App: Login-Flow, DataStore-Speicherung, Multi-Account-Unterstützung und Bootstrap.

---

## Verantwortliche Klassen

| Klasse | Datei | Aufgabe |
|--------|-------|---------|
| `TokenManager` | `data/local/TokenManager.kt` | Persistiert Token + User-Daten in DataStore, Multi-Account-Verwaltung |
| `ServerConfig` | `data/local/TokenManager.kt` (gleiche Datei) | Persistiert die Server-Basis-URL |
| `AuthRepository` | `data/repository/AuthRepository.kt` | API-Aufrufe: login, logout, activate, bootstrap, getMe |
| `LoginScreen` / `LoginViewModel` | `ui/screens/LoginScreen.kt` | Login-UI (Username/Passwort) |
| `ServerUrlScreen` / `ServerUrlViewModel` | `ui/screens/ServerUrlScreen.kt` | Server-URL-Setup beim ersten Start |

## DataStore

- **Name:** `untisx_prefs`
- **Datei:** `untisx_prefs.preferences_pb` (Preferences DataStore)
- **Zugriff:** `TokenManager` und `ServerConfig` verwenden dieselbe DataStore-Instanz.

### Keys (TokenManager)

| Key | Typ | Beschreibung |
|-----|-----|-------------|
| `auth_token` | String | Bearer-Token (JWT) |
| `user_role` | String | `admin` / `teacher` / `student` |
| `user_name` | String | Login-Name |
| `real_name` | String | Vollständiger Name |
| `user_id` | String | User-ID (als String gespeichert!) |
| `email` | String | E-Mail |
| `short_name` | String | Abkürzung |
| `dark_mode` | Boolean | Dunkel-Modus |
| `saved_accounts` | String | JSON-Array gespeicherter Konten |
| `active_account_index` | Int | Aktiver Account-Index |

### Keys (ServerConfig)

| Key | Typ | Beschreibung |
|-----|-----|-------------|
| `server_url` | String | Basis-URL des Backends (ohne trailing `/`) |

## Login-Flow

```
LoginScreen
  └─ AuthRepository.login(userName, password)
        ├─ POST /api/auth/login              → LoginResponse (token, role)
        ├─ GET  /api/auth/me                 → UserResponse (id, realName, …)
        └─ TokenManager.saveAuth(...)
             ├─ schreibt Keys auth_token, user_role, user_id, user_name, …
             └─ upsertAccountInList(...)     → nimmt Konto in saved_accounts auf
```

1. Beim Login wird zuerst `POST /auth/login` aufgerufen.
2. Falls die Antwort erfolgreich ist, wird zusätzlich `GET /auth/me` geholt, um `id`, `realName`, `email` und `shortName` zu vervollständigen.
3. Der Token und die User-Daten werden via `TokenManager.saveAuth(...)` gespeichert.
4. **Fallback:** Schlägt `GET /auth/me` fehl, werden die Daten direkt aus dem Login-Response mit `id = 0` gespeichert, damit der Login trotzdem funktioniert.

### Fehlerbehandlung

- Nicht-2xx-Antwort → `ApiResult.Error(message, code)` – die Message wird aus dem Error-Body geparst (`ErrorResponse.message`), sonst `"Fehler <code>"`.
- Exception → `ApiResult.Error(e.message ?: "Verbindungsfehler")`.

## Logout

`AuthRepository.logout()`:

1. `POST /auth/logout` wird in einem `try/catch` aufgerufen (Fehler werden ignoriert).
2. `TokenManager.clearAuth()` löscht alle Auth-Keys (`auth_token`, `user_role`, `user_id`, `user_name`, `real_name`, `email`, `short_name`).
3. Gespeicherte Konten (`saved_accounts`) bleiben erhalten.

## Aktivierung (Activate)

- Endpoint: `POST /auth/activate` (`ActivateRequest(userName, activationKey, email, password)`).
- Erfolg gilt bei HTTP 200 **oder 204**.
- Wird benötigt, wenn ein Konto noch nicht aktiviert ist (Aktivierungsschlüssel aus dem Backend).

## Bootstrap (Ersteinrichtung Server)

- Endpoint: `POST /bootstrap` mit Header `X-Bootstrap-Token`.
- Aufruf über `apiClient.buildApiForUrl(baseUrl)` – eine **separate** Retrofit-Instanz für die Ziel-URL, **bevor** die Server-URL gespeichert wird.
- Erfolg gilt bei HTTP 200 **oder 201**.
- Erstellt den ersten Admin-Benutzer, wenn die Server-URL noch unbekannt ist.

## Multi-Account

Die App unterstützt mehrere gespeicherte Konten:

- `saved_accounts` ist ein **JSON-Array** von `SavedAccount`-Objekten:
  - `userName`, `realName`, `serverUrl`, `token`, `role`, `userId`, `email`, `shortName`
- `saveAuth(...)` ruft `upsertAccountInList(...)` auf: Ein Konto mit demselben `userName` wird **ersetzt**, sonst angehängt.
- **Konto-Wechsel** (`switchToAccount`) schreibt die gewählten Daten in die aktiven Keys.
- **Konto löschen** (`removeAccountFromList(index)`) entfernt das Element aus dem JSON-Array.
- Zugriff in der UI über den `MoreScreen` → `AccountSwitcherSheet`.

## Start-Flow

```
App-Start (AppNavigation)
├── Keine Server-URL           → ServerUrlScreen → (eingeben) → Login
├── Server-URL vorhanden,
│   nicht eingeloggt           → Login → Planner
└── Eingeloggt                 → Planner
```

- `ServerUrlViewModel` validiert beim Speichern: URL nicht leer und muss mit `http://` oder `https://` beginnen.
- `ServerConfig.setServerUrl(url)` normalisiert die URL (`trimEnd('/')`).

## Wichtig: ApiResult ist hier definiert

Die Sealed Class `ApiResult<T>` liegt in `AuthRepository.kt` und wird von **allen** Repositories benutzt (siehe [Repositories – Übersicht](repositories/README.md)).