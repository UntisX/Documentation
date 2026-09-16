# HomeworkRepository

> Hausaufgaben: Liste abrufen und CRUD-Operationen.

**Datei:** `data/repository/HomeworkRepository.kt`
**Singleton:** ja (`@Singleton`)

---

## Methoden

| Methode | Endpoint | Rückgabe |
|---------|----------|----------|
| `getHomework(subjectId: Long? = null)` | `GET /api/timetable/homework?subjectId=…` | `ApiResult<List<Homework>>` |
| `create(request: HomeworkCreateRequest)` | `POST /api/timetable/homework` | `ApiResult<Homework>` |
| `update(id: Long, request: HomeworkUpdateRequest)` | `PUT /api/timetable/homework/{id}` | `ApiResult<Homework>` |
| `delete(id: Long)` | `DELETE /api/timetable/homework/{id}` | `ApiResult<Unit>` |

- Filterung nach Fach optional (leeres `subjectId` = alle).
- `Homework`-Modelle siehe [Datenmodelle](../07-datenmodelle.md).

## Verwandt

- [Hausaufgaben-Screen](../screens/homework.md)