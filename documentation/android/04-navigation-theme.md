# Navigation & Theme

> Routing-Struktur, Bottom-Navigation, Start-Flow sowie Farb- und Typografie-Konfiguration der Android-App.

---

## Routen-Definition (`Screen.kt`)

`Screen` ist eine `sealed class` – jede Route trägt Route-String, Titel und Icons:

| Objekt | Route | Titel | In Bottom-Nav |
|--------|-------|-------|---------------|
| `Login` | `login` | Anmelden | – |
| `ServerUrl` | `server_url` | Server | – |
| `Planner` | `planner` | Planer | ✅ |
| `Absences` | `absences` | Fehlstunden | ✅ |
| `Homework` | `homework` | Hausaufgaben | ✅ |
| `Messages` | `messages` | Nachrichten | ✅ |
| `More` | `more` | Profil | ✅ |
| `Grades` | `grades` | Noten | (Sub-Screen) |
| `AdminPanel` | `admin_panel` | Admin-Panel | (Sub-Screen) |
| `AuditLog` | `audit_log` | Protokoll | (Sub-Screen) |
| `Vertretungsplan` | `vertretungsplan` | Vertretungsplan | (Sub-Screen) |
| `Students` | `students` | Schüler | (Sub-Screen) |
| `Teachers` | `teachers` | Lehrer | (Sub-Screen) |
| `Subjects` | `subjects` | Fächer | (Sub-Screen) |
| `Rooms` | `rooms` | Räume | (Sub-Screen) |
| `BellSchedule` | `bell_schedule` | Stundenplan | (Sub-Screen) |

`NavItem` mit `isVisible: (role: String) -> Boolean` ermöglicht rollenbasierte Navigation.

`bottomNavItems`: exakt 5 Einträge – Planner, Absences, Homework, Messages, More (alle sichtbar).

## NavHost (`AppNavigation.kt`)

### MainViewModel

- Bündelt `TokenManager` + `ServerConfig` + `AuthRepository`.
- Exponiert: `isLoggedIn`, `needsServerUrl`, `role` (Default `student`), `realName`, `userName`, `currentUserId`, `darkMode`.
- `logout()` → `authRepository.logout()`.

### Start-Destination

```kotlin
val startDestination = when {
    needsServerUrl   -> Screen.ServerUrl.route   // keine Server-URL gesetzt
    !isLoggedIn      -> Screen.Login.route        // Token fehlt
    else             -> Screen.Planner.route      // eingeloggt
}
```

### Bottom-Bar-Sichtbarkeit

Die `NavigationBar` wird überall angezeigt **außer** auf Login/ServerUrl:

- Bottom-Nav-Screens + alle Sub-Screens (Grades, Vertretungsplan, AdminPanel, AuditLog, Students, Teachers, Subjects, Rooms, BellSchedule).
- Navigation zwischen Tabs: `popUpTo(startDestination) { saveState = true }` + `launchSingleTop` + `restoreState` → State bleibt beim Tab-Wechsel erhalten.

### Übergänge

- FadeIn/FadeOut mit `tween(200)`.

### Nach Login / Logout

- `LoginScreen.onLoginSuccess` → `navigate(Planner)` mit `popUpTo(Login){inclusive}`.
- `LoginScreen.onNavigateToServerUrl` → `navigate(ServerUrl)`.
- `MoreScreen.onLogout` → `viewModel.logout()` + `navigate(Login)` mit `popUpTo(0){inclusive}`.
- `MoreScreen.onNavigateToServerUrl` → Server-URL bearbeiten.

## Theme

| Datei | Inhalt |
|-------|--------|
| `ui/theme/Theme.kt` | Material 3 `MaterialTheme`, Light/Dark basierend auf `darkMode` |
| `ui/theme/Color.kt` | Farbpalette (u. a. Primärfarbe) |
| `ui/theme/Type.kt` | `Typography`-Definition |

- Der Dunkel-Modus wird im `MoreScreen` per Schalter umgeschaltet (`TokenManager.setDarkMode`).
- `untisx_prefs`-Key `dark_mode` speichert die Einstellung.

## Common Components

`ui/components/CommonComponents.kt` stellt wiederverwendbare Composables bereit – u. a. `UntisXLogo` (wird im ServerUrl-/Login-Screen verwendet), weitere Cards/Badges für Mehrfachnutzung.