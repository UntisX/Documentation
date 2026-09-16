# ChatRepository

> Echtzeit-Chat: Chat-Liste, Nachrichten, Mitglieder, Tipp- und Lese-Status.

**Datei:** `data/repository/ChatRepository.kt`
**Singleton:** ja (`@Singleton`)

---

## Methoden

| Methode | Endpoint | Rückgabe |
|---------|----------|----------|
| `getChats()` | `GET /api/chats` | `ApiResult<ChatListResponse>` |
| `createChat(request: ChatCreateRequest)` | `POST /api/chats` | `ApiResult<Chat>` |
| `getChatDetail(id: Long)` | `GET /api/chats/{id}` | `ApiResult<ChatDetailResponse>` |
| `getChatMessages(chatId, limit = 50, beforeId = null)` | `GET /api/chats/{id}/messages?limit=…&before_id=…` | `ApiResult<ChatMessageListResponse>` |
| `sendMessage(chatId, body)` | `POST /api/chats/{id}/messages` | `ApiResult<ChatMessage>` |
| `deleteMessage(chatId, messageId)` | `DELETE /api/chats/{id}/messages/{messageId}` | `ApiResult<Unit>` |
| `markRead(chatId, messageId)` | `POST /api/chats/{id}/read` | `ApiResult<Unit>` |
| `sendTyping(chatId)` | `POST /api/chats/{id}/typing` | `ApiResult<Unit>` |
| `leaveChat(chatId)` | `POST /api/chats/{id}/leave` | `ApiResult<Unit>` |
| `renameChat(chatId, name)` | `POST /api/chats/{id}/rename` | `ApiResult<Unit>` |

- Pagination der Nachrichten über `beforeId` (ältere Nachrichten nachladen).
- `markRead` → `ChatMarkReadRequest` (letzte gelesene Nachrichten-ID).
- Mitgliederverwaltung (`addChatMembers`, `removeChatMember`) ist im API-Interface vorhanden, wird aber aktuell stark über die UI (GroupInfoSheet/NewChatDialog) angesteuert.

## SSE im Chat

Der Chat nutzt die SSE-Infrastruktur des `ApiClient` zusätzlich zum REST-Call:

- Events: `message`, `typing`, `message_deleted`, `chat_read`, `chat`.
- Fallback: 5-Sekunden-Polling, wenn die SSE-Verbindung nicht steht.
- Verbindung lebt viewModel-scoped (nur solange der Messages-Screen aktiv ist).

## Verwandt

- [Nachrichten / Chat-Screen](../screens/messages.md)
- [API-Client & Netzwerk](../03-api-client.md) (SSE-Abschnitt)