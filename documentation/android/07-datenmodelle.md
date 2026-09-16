# Datenmodelle (DTOs)

> Alle Datenklassen in `data/model/`. Die App verwendet **Moshi** mit `@JsonClass(generateAdapter = true)` und `@Json(name = "...")` für serverseitige Feldnamen (snake_case).

---

## User & Auth (`User.kt`)

| Klasse | Felder |
|--------|--------|
| `LoginRequest` | userName, password |
| `LoginResponse` | token, role |
| `UserResponse` | id, userName, realName, email?, shortName? |
| `ErrorResponse(message: String)` | Fehler-Nachricht (aus Error-Body) |
| `ActivateRequest` | userName, activationKey, email, password |
| `CreateUsersRequest` | (Benutzer-Daten für Massen-Import) |
| `CreateUsersResponse` | (Ergebnis inkl. generierter Keys) |
| `PutUserRequest` | (Update-Daten) |
| `UserListItem` | (Listen-Item für Admin) |

## Timetable (`Timetable.kt`)

| Klasse | Inhalt |
|--------|--------|
| `TimetableEntry` | Stundenplan-Eintrag (id, subjectId, dayOfWeek, period, weekType, teacher, room, …) |
| `TimetableEntryCreateRequest` | Erstell-/Update-Daten |
| `TimetableMonthData` | Monatsansicht (Liste + verfügbare Jahre/Monate) |
| `TimetableMonthEntry` | Monats-Eintrag |
| `TimetableCancellation` | Ausfall einer Stunde (date, cancelled) |

## Homework (`Homework.kt`)

| Klasse | Inhalt |
|--------|--------|
| `Homework` | id, subjectId, title, description, dueDate, status, … |
| `HomeworkCreateRequest` | subjectId, title, description, dueDate |
| `HomeworkUpdateRequest` | alle Felder optional |

## Grade (`Grade.kt`)

| Klasse | Inhalt |
|--------|--------|
| `Grade` | id, studentId, subjectId, value, kind, date |
| `GradeCreateRequest` | studentId, subjectId, value, kind |
| `GradeUpdateRequest` | alle Felder optional |
| `GradeAverages` | Liste `SubjectAverage` + `OverallAverage` |

## Absence (`Absence.kt`)

| Klasse | Inhalt |
|--------|--------|
| `Absence` | id, studentId, start, end, reason, excused, recordedBy, student*-Felder, subjectId/Name, date, period, type |
| `AbsenceCreateRequest` | studentId, start, end, reason, excused, subjectId?, date?, period?, type? |
| `AbsenceUpdateRequest` | alle Felder optional |
| `AbsenceStats` / `AbsenceStatsData` | Statistik (Gesamt/Entschuldigt/…) |

## Message & Chat (`Message.kt`, `Chat.kt`)

| Klasse | Inhalt |
|--------|--------|
| `Message` | Klassische Nachricht (id, folder, subject, body, …) |
| `MessageListResponse` | Liste + Pagination/Order |
| `MessageCreateRequest` | (Erstellen) |
| `Chat` | Chat-Übersicht (id/roomId, name, kind, …) |
| `ChatMessage` | Chat-Nachricht (id, chatId, senderId, body, createdAt, …) |
| `ChatMember` | chatId, userId, role, name? |
| `ChatListResponse` | Liste Chats |
| `ChatDetailResponse` | Chat + Mitglieder |
| `ChatMessageListResponse` | Nachrichten + Checksums |
| `ChatMessageCreateRequest` | body |
| `ChatCreateRequest` | name?, memberIds, … |
| `ChatMarkReadRequest` | letzte gelesene messageId? |
| `ChatAddMembersRequest` | userIds |
| `ChatRenameRequest` | name |
| `ChatStreamEvent` | SSE-Event-Hülle (Typ + Daten) |

## Vertretung (`Vertretung.kt`)

| Klasse | Inhalt |
|--------|--------|
| `VertretungsplanResponse` | Ausfälle (`VertretungCancellation`) + Vertretungen (`VertretungSubstitution`) |
| `TeacherAbsence` | Lehrer-Abwesenheit (id, teacherId, date, reason) |
| `TeacherAbsenceCreateRequest` | teacherId, date, reason |
| `CancellationCreateRequest` | date |

## Ressourcen

| Datei | Klassen |
|-------|---------|
| `Subject.kt` | `Subject`, `SubjectCreateRequest`, `SubjectUpdateRequest` |
| `Room.kt` | `Room`, `RoomCreateRequest` |
| `BellSchedule.kt` | `BellScheduleEntry`, `BellScheduleCreateRequest` |
| `SchoolSettings.kt` | `SchoolSettings`, `SchoolSettingsPatch` |

## Audit (`Audit.kt`)

| Klasse | Inhalt |
|--------|--------|
| `AuditEntry` | id, userId, action, entityType, entityId, details, ipAddress, createdAt, firstName, lastName |
| `AuditLogResponse` | entries + total |
| `AuditStatsItem` | action/action-Kategorie + count |

## Misc (`Misc.kt`)

| Klasse | Inhalt |
|--------|--------|
| `CustomEvent` | id, title, description, start/end, color?, … |
| `CustomEventCreateRequest` | Erstell-/Update-Daten |
| `SyncVersions` | Versionen der Sync-Tabelle |
| `RealtimeEvent` | Event-Daten (für SSE) |
| `BootstrapRequest` | userName, realName, email, password |

> Die Feldnamen in den DTOs folgen der snake_case-Konvention des Backends; die App-Klassen verwenden camelCase mit `@Json`-Mapping (z. B. `@Json(name = "student_id") val studentId`).