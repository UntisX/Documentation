# AdminRepository

> Verwaltungs-Endpunkte für Admins: Benutzer, Fächer, Räume, Schuleinstellungen, Vertretungsplan und Audit-Protokoll.

**Datei:** `data/repository/AdminRepository.kt`
**Singleton:** ja (`@Singleton`)

---

## Methoden

### Benutzer

| Methode | Endpoint |
|---------|----------|
| `getUsers()` | `GET /api/users` |
| `createUser(request: CreateUsersRequest)` | `POST /api/users` |
| `updateUser(id: Long, request: PutUserRequest)` | `PUT /api/users/{id}` |
| `deleteUser(id: Long)` | `DELETE /api/users/{id}` |

- Liefert `List<UserListItem>` bzw. `CreateUsersResponse` (inkl. generierter Aktivierungsschlüssel beim Import).

### Fächer

| Methode | Endpoint |
|---------|----------|
| `getSubjects()` | `GET /api/school/subjects` |
| `createSubject(request: SubjectCreateRequest)` | `POST /api/school/subjects` |
| `deleteSubject(id: Long)` | `DELETE /api/school/subjects/{id}` |

### Räume

| Methode | Endpoint |
|---------|----------|
| `getRooms()` | `GET /api/school/rooms` |
| `createRoom(request: RoomCreateRequest)` | `POST /api/school/rooms` |
| `deleteRoom(id: Long)` | `DELETE /api/school/rooms/{id}` |

### Schuleinstellungen

| Methode | Endpoint |
|---------|----------|
| `getSchoolSettings()` | `GET /api/school/settings` |
| `updateSchoolSettings(request: SchoolSettingsPatch)` | `PATCH /api/school/settings` |

### Vertretungsplan

| Methode | Endpoint |
|---------|----------|
| `getVertretungsplan(date: String)` | `GET /api/timetable/vertretungsplan?date=…` |
| `createCancellation(entryId: Long, request: CancellationCreateRequest)` | `POST /api/timetable/{entryId}/cancel` |
| `deleteCancellation(entryId: Long, date: String)` | `DELETE /api/timetable/{entryId}/cancel?date=…` |

### Audit-Protokoll

| Methode | Endpoint |
|---------|----------|
| `getAuditLog(limit = 100, offset = 0, action: String? = null)` | `GET /api/administration/audit?limit=…&offset=…&action=…` |

> `getAuditLog` kapselt die weiteren API-Parameter (`user_id`, `from`, `to`) aktuell nicht; nur limit/offset/action werden durchgereicht.

## Verwandt

- [Admin-Panel](../screens/admin-panel.md), [Audit-Protokoll](../screens/audit-log.md), [Vertretungsplan](../screens/vertretungsplan.md)
- [Schüler](../screens/students.md), [Lehrer](../screens/teachers.md), [Fächer](../screens/subjects.md), [Räume](../screens/rooms.md)