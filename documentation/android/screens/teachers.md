# Lehrer-Screen (Admin)

> Lehrer-Verwaltung mit Anlegen und Löschen – Teil von `ResourceScreens.kt`.

**Route:** `teachers` (Sub-Screen, Admin)
**Datei:** `ui/screens/ResourceScreens.kt` (`TeachersScreen`, `CreateTeacherDialog`)
**ViewModel:** `TeachersViewModel` (`@HiltViewModel`)
**Backing:** `AdminRepository`

---

## Funktionen

- Zeigt alle Benutzer mit Rolle `teacher` aus `GET /api/users` (gefiltert clientseitig).
- **Neuen Lehrer anlegen:** FloatingActionButton → `CreateTeacherDialog` (Benutzername, Vollständiger Name, E-Mail optional) → `POST /api/users` mit `role = "teacher"`.
- **Lehrer löschen:** Löschen-Icon je Karte → `ConfirmDeleteDialog` → `DELETE /api/users/{id}`, danach Liste neu laden.

## ViewModel-Methoden

| Methode | Aufruf |
|---------|--------|
| `loadTeachers()` | `GET /api/users` → Filter `role == "teacher"` |
| `deleteTeacher(id)` | `DELETE /api/users/{id}` → reload |
| `createTeacher(userName, realName, email)` | `POST /api/users` (`CreateUsersRequest(userName, realName, null, "teacher")`) → reload |

> Hinweis: CSV-Import existiert nur im Web-Frontend, nicht in der Android-App.

## Verwandt

- [AdminRepository](../repositories/admin-repository.md)