# Admin-Panel-Screen

> Zentrale Verwaltungsoberfläche mit mehreren Tabs für Schule, Layout und Integration.

**Route:** `admin_panel` (Sub-Screen, Admin)
**Datei:** `ui/screens/AdminPanelScreen.kt`
**ViewModel:** `AdminPanelViewModel` (`@HiltViewModel`)
**Backing:** `AdminRepository`

---

## UI-Bausteine

| Composable | Funktion |
|-----------|----------|
| `AdminPanelScreen` | Haupt-UI mit Tab-Navigation |
| `SchoolTab` | Schuleinstellungen (Name etc.) |
| `LayoutTab` | Layout-Konfiguration |
| `IntegrationTab` | Integrationen/Einstellungen |

## Funktionen

- **Schule:** Schuleinstellungen anzeigen/ändern → `GET/PATCH /api/school/settings` (`getSchoolSettings`, `updateSchoolSettings`).
- **Layout:** Layout-/Konfigurationsoptionen (UI).
- **Integration:** Integrations-Optionen (UI).

> **WIP:** `LayoutTab` und `IntegrationTab` sind aktuell **UI-only** – es werden nur Oberflächen-Elemente angezeigt, keine echten API-Calls.

## Verwandt

- [AdminRepository](../repositories/admin-repository.md) (getSchoolSettings, updateSchoolSettings)
- [SchoolSettings-DTOs](../07-datenmodelle.md)