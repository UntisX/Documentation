# AuthRepository

> Authentifizierung: Login (mit `auth/me`-Fallback), Logout, Aktivierung und Bootstrap.

**Datei:** `data/repository/AuthRepository.kt`
**Singleton:** ja (`@Singleton`)

---

## Methoden

### `login(userName: String, password: String): ApiResult<LoginResponse>`

```
POST /api/auth/login  → LoginResponse (token, role)
GET  /api/auth/me     → UserResponse  (id, realName, email, …)
```

1. Login-Aufruf.
2. Bei Erfolg Zusatzaufruf `getMe()`, um `id`, `realName`, `email`, `shortName` zu vervollständigen.
3. `tokenManager.saveAuth(...)`:
   - Erfolgt mit den Me-Daten, wenn `getMe()` erfolgreich.
   - **Fallback:** bei fehlgeschlagenem `getMe()` mit `id = 0` und `userName` als `realName` (Login funktioniert trotzdem).
4. Gibt `ApiResult.Success(body)` des Login-Responses zurück.

### `getMe(): ApiResult<UserResponse>`

- `GET /api/auth/me`; gibt `Success(data)` oder `Error`.

### `logout()`

- `POST /api/auth/logout` in `try/catch` (Fehler werden ignoriert).
- Danach `tokenManager.clearAuth()` (löscht alle Auth-Keys, `saved_accounts` bleibt).

### `activate(userName, activationKey, email, password): ApiResult<Unit>`

- `POST /api/auth/activate` (`ActivateRequest`).
- Erfolg bei HTTP 200 **oder 204**.

### `bootstrap(baseUrl, bootstrapToken, userName, realName, email, password): ApiResult<Unit>`

- Verwendet `apiClient.buildApiForUrl(baseUrl)` – eine **separate** Retrofit-Instanz (Server-URL ist noch nicht gespeichert).
- `POST /api/bootstrap` mit Header `X-Bootstrap-Token`.
- Erfolg bei HTTP 200 **oder 201**.

## Fehlerbehandlung

- Nicht-2xx → `parseError(response)` liefert die Server-Message (`ErrorResponse.message`) oder `"Fehler <code>"`.
- Ausnahme → `ApiResult.Error(e.message ?: "Verbindungsfehler")`.

## Verwandt

- [Auth & Login](../02-auth-login.md) – DataStore-Keys, Multi-Account, Login-Flow