# MessageRepository

> Klassische Nachrichten (Ordner-System, E-Mail-artig im Gegensatz zum Chat).

**Datei:** `data/repository/MessageRepository.kt`
**Singleton:** ja (`@Singleton`)

---

## Methoden

| Methode | Endpoint | Rückgabe |
|---------|----------|----------|
| `getMessages(folder: String = "inbox")` | `GET /api/messages?folder=…` | `ApiResult<MessageListResponse>` |
| `create(request: MessageCreateRequest)` | `POST /api/messages` | `ApiResult<Message>` |
| `markRead(id: Long)` | `PUT /api/messages/{id}/read` | `ApiResult<Unit>` |
| `delete(id: Long)` | `DELETE /api/messages/{id}` | `ApiResult<Unit>` |

- Ordner-Parameter (`inbox`, …) wird als Query übergeben.
- `markRead` markiert eine Nachricht als gelesen.

> Hinweis: Der Chat-Bereich (Echtzeit-Chat, Chat-Liste) läuft über das **ChatRepository**, nicht hierher. Der klassische Nachrichten-Bereich ist aktuell in der UI wenig ausgeprägt (der Haupt-Fokus liegt auf Chats).

## Verwandt

- [ChatRepository](chat-repository.md)
- [Nachrichten / Chat-Screen](../screens/messages.md)