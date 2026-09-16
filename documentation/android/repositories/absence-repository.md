# AbsenceRepository

> Fehlzeiten (An-/Abwesenheiten) und Statistik.

**Datei:** `data/repository/AbsenceRepository.kt`
**Singleton:** ja (`@Singleton`)

---

## Methoden

| Methode | Endpoint | Rückgabe |
|---------|----------|----------|
| `getAbsences(studentId: Long? = null)` | `GET /api/students/absences?studentId=…` | `ApiResult<List<Absence>>` |
| `getStats(studentId: Long? = null)` | `GET /api/students/absences/stats?studentId=…` | `ApiResult<AbsenceStats>` |
| `create(request: AbsenceCreateRequest)` | `POST /api/students/absences` | `ApiResult<Absence>` |
| `update(id: Long, request: AbsenceUpdateRequest)` | `PUT /api/students/absences/{id}` | `ApiResult<Absence>` |
| `delete(id: Long)` | `DELETE /api/students/absences/{id}` | `ApiResult<Unit>` |

- `studentId` optional (Schüler = eigene Fehlzeiten, Lehrer/Admin = fremde Schüler-ID).
- `AbsenceStats` speist die Statistik-Karten im Fehlstunden-Screen (Gesamt/Entschuldigt/Unentschuldigt/Verspätet).

> Bekanntes WIP: Der `CreateAbsenceDialog` in der UI ruft `create()` noch nicht auf (nur Dialog vorhanden).

## Verwandt

- [Fehlstunden-Screen](../screens/absences.md)