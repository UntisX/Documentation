# Repositories – Übersicht

> Alle Repositories der App, das gemeinsame `ApiResult<T>`-Pattern und die einheitliche Fehlerbehandlung.

---

## Muster

Jedes Repository ist `@Singleton`, bekommt `ApiClient` (und ggf. `TokenManager`) per Konstruktor-Injection und enthält ausschließlich `suspend fun`s:

```kotlin
@Singleton
class XRepository @Inject constructor(
    private val apiClient: ApiClient
) {
    suspend fun load(): ApiResult<...> {
        return try {
            val api = apiClient.getApi()
            val response = api.endpoint(...)
            if (response.isSuccessful) {
                ApiResult.Success(response.body()!!)
            } else {
                ApiResult.Error(parseError(response), response.code())
            }
        } catch (e: Exception) {
            ApiResult.Error(e.message ?: "Verbindungsfehler")
        }
    }
}
```

### Erfolgswerte

- 2xx-Status → `ApiResult.Success(data)`.
- Einige Endpunkte akzeptieren auch 204 (`activate`) bzw. 201 (`bootstrap`) als Erfolg, wenn `response.body()` irrelevant ist.

### Fehlerwerte

- Nicht-2xx → `ApiResult.Error(message, code)`, wobei `message` aus dem Error-Body geparst wird (`ErrorResponse.message`), sonst `"Fehler <code>"`.
- Exception → `ApiResult.Error(e.message ?: "Verbindungsfehler")`.

> Der gemeinsame Fehler-Parser `parseError()` ist aktuell in `AuthRepository.kt` definiert. Die anderen Repositories halten dasselbe Muster ein.

## `ApiResult<T>` (Definition in `AuthRepository.kt`)

```kotlin
sealed class ApiResult<out T> {
    data class Success<T>(val data: T) : ApiResult<T>()
    data class Error(val message: String, val code: Int = 0) : ApiResult<Nothing>()
}
```

## Repository-Liste

| # | Repository | Datei | Zuständig |
|---|------------|-------|-----------|
| 1 | [AuthRepository](auth-repository.md) | data/repository/AuthRepository.kt | Login, Logout, Activate, Bootstrap, getMe |
| 2 | [TimetableRepository](timetable-repository.md) | data/repository/TimetableRepository.kt | Stundenplan, Monatsdaten, Custom Events |
| 3 | [HomeworkRepository](homework-repository.md) | data/repository/HomeworkRepository.kt | Hausaufgaben CRUD |
| 4 | [GradeRepository](grade-repository.md) | data/repository/GradeRepository.kt | Noten + Durchschnitt |
| 5 | [MessageRepository](message-repository.md) | data/repository/MessageRepository.kt | Nachrichten (Ordner) |
| 6 | [ChatRepository](chat-repository.md) | data/repository/ChatRepository.kt | Chats, Nachrichten, Mitglieder |
| 7 | [AbsenceRepository](absence-repository.md) | data/repository/AbsenceRepository.kt | Fehlzeiten + Statistik |
| 8 | [AdminRepository](admin-repository.md) | data/repository/AdminRepository.kt | Benutzer, Fächer, Räume, Schule, Vertretung, Audit |

## ViewModel-Nutzung

ViewModels konsumieren den Rückgabewert per `when`:

```kotlin
when (val result = repository.load()) {
    is ApiResult.Success -> state.value = result.data
    is ApiResult.Error   -> error.value = result.message
}
```