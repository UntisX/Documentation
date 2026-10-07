# API-Client & Netzwerk

> Wie die Android-App mit dem Backend kommuniziert: `ApiClient` (OkHttp + Retrofit), der Auth-Interceptor, der `/api`-Pfad-Präfix und die SSE-Infrastruktur für den Chat.

---

## Übersicht

| Datei | Funktion |
|-------|----------|
| `data/api/ApiClient.kt` | Baut Retrofit-Instanzen, OkHttp-Client, Interceptor, SSE-Helfer |
| `data/api/UntisXApi.kt` | Retrofit-Interface mit allen Endpunkten |

## ApiClient (@Singleton)

```kotlin
class ApiClient @Inject constructor(
    context: Context,
    tokenManager: TokenManager,
    serverConfig: ServerConfig
)
```

### Retrofit-Instanzen

- `getApi(): UntisXApi` – liefert die **gecachte** Instanz. Basis-URL = `serverConfig.serverUrl`. Ist die URL nicht gesetzt, fällt sie auf `http://10.0.2.2:3000` (Android-Emulator-Loopback) zurück. Bei URL-Änderung wird eine neue Retrofit-Instanz gebaut und gecacht.
- `buildApiForUrl(baseUrl)` – baut eine **separate** Instanz für eine explizite URL (wird nur vom Bootstrap verwendet, bevor die Server-URL gespeichert ist).
- JSON-Parsing: **Moshi** mit `KotlinJsonAdapterFactory`.

### OkHttp-Setup

```
connectTimeout / readTimeout / writeTimeout = 30 s
Interceptor-Order:
  1. authInterceptor
  2. loggingInterceptor (HttpLoggingInterceptor mit Level.BODY)
```

### Auth-Interceptor

Fügt bei **jeder** Anfrage hinzu:

1. **Pfad-Präfix `/api`**: Wird automatisch vor den Pfad gesetzt (`/timetable` → `/api/timetable`), außer:
   - Host ist `localhost`, `10.0.2.2` oder `127.0.0.1` (lokale Entwicklung, kein Präfix), oder
   - der Pfad beginnt bereits mit `/api/`.
2. **Header**: `Content-Type: application/json` (immer).
3. **Header**: `Authorization: Bearer <token>` – sofern ein Token im DataStore liegt.

Daraus ergibt sich, dass die App dieselben `/api/...`-Endpunkte nutzt wie das Web-Frontend.

### Logging

`HttpLoggingInterceptor.Level.BODY` – auch im Release-Build (siehe [Sicherheit](05-security.md)).

## UntisXApi – Endpunkt-Mapping

Alle Endpunkte aus der [API-Dokumentation](../README.md) sind als `suspend fun` im Interface vorhanden, jeweils ohne `/api`-Präfix (das übernimmt der Interceptor):

### Auth & Bootstrap

| Methode | Endpoint | Retrofit-Funktion |
|---------|----------|-------------------|
| GET | `/health` | `healthCheck()` |
| POST | `/auth/login` | `login(LoginRequest)` |
| POST | `/auth/activate` | `activate(ActivateRequest)` |
| POST | `/auth/logout` | `logout()` |
| GET | `/auth/me` | `getMe()` |
| POST | `/bootstrap` | `bootstrap(X-Bootstrap-Token, BootstrapRequest)` |

### User & Schule (Admin)

| Methode | Endpoint | Retrofit-Funktion |
|---------|----------|-------------------|
| GET/POST | `/users` | `getUsers()` / `createUser(CreateUsersRequest)` |
| PUT/DELETE | `/users/{id}` | `updateUser` / `deleteUser` |
| POST | `/users/{id}/activation-reset` | `resetActivation(id)` |
| GET | `/sync/versions` | `getSyncVersions()` |
| GET/PATCH | `/school/settings` | `getSchoolSettings()` / `patchSchoolSettings(...)` |
| CRUD | `/school/subjects` | `getSubjects/createSubject/updateSubject/deleteSubject` |
| CRUD | `/school/rooms` | `getRooms/createRoom/updateRoom/deleteRoom` |
| CRUD | `/school/bell-schedule` | `getBellSchedule/createBellScheduleEntry/updateBellScheduleEntry/deleteBellScheduleEntry` |

### Stundenplan & Hausaufgaben

| Methode | Endpoint | Retrofit-Funktion |
|---------|----------|-------------------|
| GET | `/timetable` | `getTimetable()` |
| CRUD | `/timetable` | `createTimetableEntry/updateTimetableEntry/deleteTimetableEntry` |
| GET | `/timetable/month-data` | `getTimetableMonthData(year, month)` |
| GET/POST | `/timetable/homework` | `getHomework(subjectId?)` / `createHomework(...)` |
| PUT/DELETE | `/timetable/homework/{id}` | `updateHomework` / `deleteHomework` |

### Noten, Fehlstunden, Vertretungsplan

| Methode | Endpoint | Retrofit-Funktion |
|---------|----------|-------------------|
| CRUD | `/students/grades` | `getGrades(studentId?)/createGrade/updateGrade/deleteGrade` |
| GET | `/students/grades/average` | `getGradeAverages(studentId?)` |
| CRUD | `/students/absences` | `getAbsences(studentId?)/createAbsence/updateAbsence/deleteAbsence` |
| GET | `/students/absences/stats` | `getAbsenceStats(studentId?)` |
| GET | `/timetable/vertretungsplan` | `getVertretungsplan(date)` |
| POST/DELETE | `/timetable/{entryId}/cancel` | `createCancellation` / `deleteCancellation(entryId, date)` |
| CRUD | `/timetable/teacher-absences` | `getTeacherAbsences(date)/createTeacherAbsence/deleteTeacherAbsence` |

### Nachrichten & Chats

| Methode | Endpoint | Retrofit-Funktion |
|---------|----------|-------------------|
| GET/POST | `/messages` | `getMessages(folder)` / `createMessage(...)` |
| PUT | `/messages/{id}/read` | `markMessageRead(id)` |
| DELETE | `/messages/{id}` | `deleteMessage(id)` |
| GET/POST | `/chats` | `getChats()` / `createChat(...)` |
| GET | `/chats/{id}` | `getChatDetail(id)` |
| GET/POST | `/chats/{id}/messages` | `getChatMessages(id, limit, beforeId)` / `sendChatMessage(...)` |
| DELETE | `/chats/{id}/messages/{messageId}` | `deleteChatMessage(...)` |
| POST | `/chats/{id}/read` | `markChatRead(id, ChatMarkReadRequest)` |
| POST | `/chats/{id}/typing` | `sendChatTyping(id)` |
| POST/DELETE | `/chats/{id}/members` | `addChatMembers` / `removeChatMember(id, userId)` |
| POST | `/chats/{id}/leave` | `leaveChat(id)` |
| POST | `/chats/{id}/rename` | `renameChat(id, ChatRenameRequest)` |

### Custom Events & Audit

| Methode | Endpoint | Retrofit-Funktion |
|---------|----------|-------------------|
| CRUD | `/custom-events` | `getCustomEvents(year, month)/createCustomEvent/updateCustomEvent/deleteCustomEvent` |
| GET | `/administration/audit` | `getAuditLog(limit, offset, action, userId, from, to)` |

## Echtzeit (SSE)

Die SSE-Infrastruktur ist im `ApiClient` vorhanden:

```kotlin
suspend fun buildSseUrl(path: String): String?   // Basis-URL + Pfad
fun openEventSource(fullUrl: String, listener: EventSourceListener): EventSource
```

- Die URL wird aus `serverConfig.serverUrl` gebaut.
- Genutzt wird sie aktuell **nur im Chat** (`MessagesScreen`), z. B. `GET /events?enc=1`. Der Server akzeptiert das Token bevorzugt im `Authorization`-Header (`?token=` nur Fallback) – neue Implementierungen sollten den Header schicken.
- Events: `message`, `typing`, `message_deleted`, `chat_read`, `chat` (siehe [08-realtime-sse.md](../08-realtime-sse.md)).
- **Fallback:** Läuft die SSE-Verbindung nicht, wechselt der Chat auf 5-Sekunden-Polling.
- Verbindung lebt viewModel-scoped: Sie existiert nur solange der Messages-Screen aktiv ist.

## Kein HTTPS-Erzwingen auf App-Seite

- Die App validiert nur das URL-Schema (`http://`/`https://`), erzwingt aber kein TLS.
- `network_security_config.xml` erlaubt Cleartext für alle Domains (siehe [Sicherheit](05-security.md)).