# Login-Screen (Anmelden)

> Benutzeranmeldung mit Username/Passwort inkl. Multi-Account und Server-URL-Wechsel.

**Route:** `login`
**Datei:** `ui/screens/LoginScreen.kt`
**ViewModel:** `LoginViewModel` (`@HiltViewModel`)

---

## Zweck

- Anmeldung gegen das Backend (JWT).
- Wechsel zur Server-URL-Eingabe, wenn der Server geändert werden soll.
- Aktivierung eines Kontos (falls nötig).

## UI

- Username-Feld
- Passwort-Feld
- Button „Anmelden"
- Link zu „Server-Einstellungen" (Server-URL ändern → `ServerUrlScreen`)
- Fehler-Anzeige bei `ApiResult.Error`
- Mehr-Konto: Zugriff auf gespeicherte Konten (via TokenManager/DataStore)

## Flow

```
LoginScreen
  └─ LoginViewModel.login(userName, password)
       └─ AuthRepository.login(...)
            ├─ POST /api/auth/login
            ├─ GET  /api/auth/me   (Fallback möglich)
            └─ TokenManager.saveAuth(...)  → Konto in saved_accounts
       └─ bei Erfolg → onLoginSuccess() → Navigation zu PlannerScreen
```

- Erfolg: `onLoginSuccess` navigiert zu `planner` mit `popUpTo(Login){inclusive}`.
- „Server ändern": `onNavigateToServerUrl` → `server_url` mit `popUpTo(Login){inclusive}`.

## Fehlerbehandlung

- Falsche Zugangsdaten → Fehlermeldung aus `ApiResult.Error` (Server-Message oder `"Fehler <code>"`).
- Netzwerkprobleme → `"Verbindungsfehler"`.
- Während des Logins: Button disabled + Lade-Indikator.

## Multi-Account

- Beim erfolgreichen Login wird das Konto automatisch in die `saved_accounts`-Liste übernommen (gleicher `userName` wird ersetzt).
- Der Account-Wechsel selbst passiert über den [Profil-Screen](more.md) (AccountSwitcherSheet).

## Verwandt

- [Auth & Login](../02-auth-login.md)
- [Server-URL-Screen](server-url.md)
- [AuthRepository](../repositories/auth-repository.md)