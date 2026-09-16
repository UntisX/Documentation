# Server-URL-Screen

> Erster Einstieg der App: Verbindung zu einem UntisX-Server herstellen.

**Route:** `server_url`
**Datei:** `ui/screens/ServerUrlScreen.kt`
**ViewModel:** `ServerUrlViewModel` (`@HiltViewModel`)

---

## Zweck

- Wird als Start-Destination angezeigt, wenn **keine Server-URL** im DataStore gespeichert ist.
- Erlaubt das Eingeben und Speichern der Backend-Basis-URL (z. B. `https://meine-schule.de`).

## UI

- Zentrierte Card mit `UntisXLogo` + Titel „UntisX".
- `OutlinedTextField` für die Server-URL (Label „Server-URL", Placeholder `https://meine-schule.de`).
- Validierungs-/Fehler-Text unter dem Feld (`supportingText`).
- Button „Speichern" mit Lade-Spinner während des Speicherns.

## Validierung (ViewModel)

```kotlin
fun save() {
    val url = _serverUrl.value.trim()
    if (url.isBlank()) → "Bitte gib eine Server-URL ein"
    if (!url.startsWith("http://") && !url.startsWith("https://"))
         → "URL muss mit http:// oder https:// beginnen"
}
```

## Ablauf

1. Beim Start lädt das ViewModel den aktuellen Wert aus `ServerConfig.serverUrl`.
2. Nach erfolgreichem Speichern `setServerUrl(url)` – die App normalisiert die URL (`trimEnd('/')`).
3. `saved=true` → `LaunchedEffect` ruft `onServerUrlSaved()` auf → Navigation zur Login-Seite (`popUpTo(ServerUrl){inclusive}`).

> Erst **nach** gespeicherter URL steht die Basis-URL für `ApiClient.getApi()` fest. Bis dahin fällt der Client auf `http://10.0.2.2:3000` zurück.

## Verwandt

- [Auth & Login](../02-auth-login.md) – Start-Flow
- [Navigation & Theme](../04-navigation-theme.md) – Start-Destination