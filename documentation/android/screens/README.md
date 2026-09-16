# Screens – Übersicht

> Alle Screens (Features) der Android-App: Routen, Zugriffsrechte und zugehörige ViewModels.

---

## Routen-Tabelle

| # | Screen | Route | Datei | Zugriff |
|---|--------|-------|-------|---------|
| 1 | [Server-URL](server-url.md) | `server_url` | `ServerUrlScreen.kt` | vor Login (keine Server-URL) |
| 2 | [Login](login.md) | `login` | `LoginScreen.kt` | nicht eingeloggt |
| 3 | [Planner](planner.md) | `planner` | `PlannerScreen.kt` | alle (Bottom-Nav) |
| 4 | [Fehlstunden](absences.md) | `absences` | `AbsencesScreen.kt` | alle (Bottom-Nav) |
| 5 | [Hausaufgaben](homework.md) | `homework` | `HomeworkScreen.kt` | alle (Bottom-Nav) |
| 6 | [Nachrichten / Chat](messages.md) | `messages` | `MessagesScreen.kt` | alle (Bottom-Nav) |
| 7 | [Profil (Mehr)](more.md) | `more` | `MoreScreen.kt` | alle (Bottom-Nav) |
| 8 | [Noten](grades.md) | `grades` | `GradesScreen.kt` | Schüler |
| 9 | [Vertretungsplan](vertretungsplan.md) | `vertretungsplan` | `VertretungsplanScreen.kt` | Admin |
| 10 | [Admin-Panel](admin-panel.md) | `admin_panel` | `AdminPanelScreen.kt` | Admin |
| 11 | [Audit-Protokoll](audit-log.md) | `audit_log` | `AuditLogScreen.kt` | Admin |
| 12 | [Schüler](students.md) | `students` | `ResourceScreens.kt` | Admin |
| 13 | [Lehrer](teachers.md) | `teachers` | `ResourceScreens.kt` | Admin |
| 14 | [Fächer](subjects.md) | `subjects` | `ResourceScreens.kt` | Admin |
| 15 | [Räume](rooms.md) | `rooms` | `ResourceScreens.kt` | Admin |
| 16 | [Klingelzeiten](bell-schedule.md) | `bell_schedule` | `ResourceScreens.kt` | Admin |

## Start-Flow

```
App-Start
├── Keine Server-URL   → ServerUrlScreen → Login
├── Server-URL, ohne Login → Login → Planner
└── Eingeloggt         → Planner
```

## Zugriff

- **Bottom-Navigation** (immer sichtbar): Planner, Fehlstunden, Hausaufgaben, Nachrichten, Profil.
- **Sub-Screens** werden über den `MoreScreen` (Profil) angesteuert – die Sichtbarkeit einzelner Menü-Einträge hängt von der Rolle ab (Admin → Admin-Bereich; Schüler → Noten).
- Die Navigation ist rollen-basiert über `NavItem.isVisible` und die tatsächliche Sichtbarkeit im More-Screen-Menü geregelt.

## Gemeinsame Muster

- Jeder Screen nutzt ein eigenes `@HiltViewModel` mit `StateFlow`-States.
- Laden: `LaunchedEffect` / `init` → Repository-Aufruf → `when(result)` auf `ApiResult`.
- Lade-State (`isLoading`) wird im UI als `CircularProgressIndicator` dargestellt.

## WIP-Einschränkungen in Screens

| Screen | Einschränkung |
|--------|---------------|
| Absences | Create-Dialog ruft API nicht auf |
| Homework | Create-Dialog ist `// TODO` |
| Grades | Delete-Dialog löscht nicht wirklich |
| BellSchedule | Nur Placeholder |
| AdminPanel | Layout-/Integration-Tabs sind UI-only |