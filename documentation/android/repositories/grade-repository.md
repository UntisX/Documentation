# GradeRepository

> Noten und Notendurchschnitte.

**Datei:** `data/repository/GradeRepository.kt`
**Singleton:** ja (`@Singleton`)

---

## Methoden

| Methode | Endpoint | Rückgabe |
|---------|----------|----------|
| `getGrades(studentId: Long? = null)` | `GET /api/students/grades?studentId=…` | `ApiResult<List<Grade>>` |
| `getAverages(studentId: Long? = null)` | `GET /api/students/grades/average?studentId=…` | `ApiResult<GradeAverages>` |
| `create(request: GradeCreateRequest)` | `POST /api/students/grades` | `ApiResult<Grade>` |
| `update(id: Long, request: GradeUpdateRequest)` | `PUT /api/students/grades/{id}` | `ApiResult<Grade>` |
| `delete(id: Long)` | `DELETE /api/students/grades/{id}` | `ApiResult<Unit>` |

- `studentId` optional: Schüler sehen eigene Noten, Lehrer/Admin übergeben explizit die Schüler-ID.
- `GradeAverages` enthält `SubjectAverage`-Liste + `OverallAverage`.

> Bekanntes WIP: Die UI (`GradeCard`) bietet zwar einen Delete-Dialog an, löscht die Note aber aktuell nicht wirklich (siehe [README](../README.md#bekannte-einschränkungen-wip)).

## Verwandt

- [Noten-Screen](../screens/grades.md)