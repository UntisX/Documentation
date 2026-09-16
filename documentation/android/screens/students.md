# Schüler-Screen (Admin)

> Liste aller Schüler mit Suche – Teil von `ResourceScreens.kt`.

**Route:** `students` (Sub-Screen, Admin)
**Datei:** `ui/screens/ResourceScreens.kt` (`StudentsScreen`)
**ViewModel:** `StudentsViewModel` (`@HiltViewModel`)
**Backing:** `AdminRepository`

---

## Funktionen

- Zeigt alle Benutzer mit Rolle `student` aus `GET /api/users` (gefiltert clientseitig auf `role == "student"`).
- Suchfeld (Name/Login-Name, case-insensitive; Filter rein lokal).
- Karten je Schüler: Realname, Login-Name, E-Mail, Erstellungsdatum (`createdAt`).
- Lade-/Leer-Zustände via `LoadingBox` / `EmptyState`.

> Hinweis: Kein Löschen über die Schüler-Liste in der Android-App (die Lehrer-Liste bietet Löschen an). CSV-Import ist hier nicht implementiert (nur im Web-Frontend).

## Daten

- `GET /api/users` über `AdminRepository.getUsers()`.

## Verwandt

- [AdminRepository](../repositories/admin-repository.md)