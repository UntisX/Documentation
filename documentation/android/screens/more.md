# Profil-Screen (More)

> Profil, Einstellungen, Konten-Verwaltung und Einstieg in Sub-Screens.

**Route:** `more` (Bottom-Navigation: Tab 5 „Profil")
**Datei:** `ui/screens/MoreScreen.kt`
**ViewModel:** `MoreViewModel` (`@HiltViewModel`)
**Kontext:** Rolle/Name/User-ID kommen aus dem `MainViewModel` (`AppNavigation.kt`)

---

## UI-Bausteine

| Composable | Funktion |
|-----------|----------|
| `MoreScreen` | Haupt-UI: Profil + Menüliste |
| `ProfileHeader` | Avatar, Name, Rolle, Fehlstunden-Badges |
| `AbsenceStatItem` | Statistik-Zeile (Label + Wert + Farbe) |
| `AccountSwitcherSheet` | Bottom-Sheet: gespeicherte Konten wechseln/hinzufügen |
| `MoreMenuItem` | Menü-Eintrag (Icon + Label, klickbar) |

## Funktionen

- **Profil-Header:** Realname, Login-Name, Rolle.
- **Fehlzeiten-Statistik** im Header (entschuldigt/unentschuldigt).
- **Dunkel-Modus-Toggle** (persistiert via `TokenManager.setDarkMode`).
- **Account-Switcher:** gespeicherte Konten wechseln (Multi-Account), neuen Account hinzufügen, Logout.
- **Menü zu Sub-Screens** (abhängig von der Rolle):
  - Schüler: Noten
  - Admin: Vertretungsplan, Admin-Panel, Audit-Protokoll, Schüler, Lehrer, Fächer, Räume, Klingelzeiten
- **Logout:** `onLogout` → `viewModel.logout()` (AuthRepository.logout) → Navigation zu `login` mit `popUpTo(0){inclusive}`.

## Verwandt

- [Auth & Login (Multi-Account)](../02-auth-login.md)
- [Navigation & Theme](../04-navigation-theme.md)
- [Screens – Übersicht](README.md)