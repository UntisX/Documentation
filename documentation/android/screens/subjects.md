# Fächer-Screen (Admin)

> Fach-Verwaltung mit Anlegen, Löschen und Detail-Dialog – Teil von `ResourceScreens.kt`.

**Route:** `subjects` (Sub-Screen, Admin)
**Datei:** `ui/screens/ResourceScreens.kt` (`SubjectsScreen`, `SubjectDetailDialog`, `DetailRow`)
**ViewModel:** `SubjectsViewModel` (`@HiltViewModel`)
**Backing:** `AdminRepository`

---

## Funktionen

- Liste aller Fächer (`GET /api/school/subjects`) mit Farb-Indikator.
- Klick auf Karte → `SubjectDetailDialog`: Kürzel, Farbe, Aktiv/Inaktiv-Status.
- **Neues Fach:** FAB → Dialog (Name + Kürzel) → `POST /api/school/subjects`.
- **Fach löschen:** Löschen-Icon → `ConfirmDeleteDialog` → `DELETE /api/school/subjects/{id}` reload.

## ViewModel-Methoden

| Methode | Aufruf |
|---------|--------|
| `loadSubjects()` | `GET /api/school/subjects` |
| `deleteSubject(id)` | `DELETE /api/school/subjects/{id}` → reload |
| `createSubject(name, shortName)` | `POST /api/school/subjects` (`SubjectCreateRequest(name, shortName)`) → reload |

## Hinweise

- Farbwerte kommen als Hex-String (`subject.color`) und werden per `Color.parseColor` geparst (Fallback `Accent`).
- `DetailRow` ist ein generisches Label/Wert-Composable (auch in anderen Details verwendbar).

## Verwandt

- [AdminRepository](../repositories/admin-repository.md)
- [Subject-DTOs](../07-datenmodelle.md)