# Nachrichten / Chat-Screen

> Echtzeit-Chat mit Chat-Liste und Chat-Thread (SSE + Polling-Fallback).

**Route:** `messages` (Bottom-Navigation: Tab 4 „Nachrichten")
**Datei:** `ui/screens/MessagesScreen.kt` (enthält **beide** Ansichten)
**ViewModels:** `MessagesViewModel` oder Chat-ViewModels (`@HiltViewModel`)
**Backing:** `ChatRepository`, `MessageRepository`

---

## Zwei Ansichten in einer Datei

| Composable | Funktion |
|-----------|----------|
| `MessagesScreen` | Einstieg: zeigt je nach State `ChatListScreen` oder `ChatThreadScreen` |
| `ChatListScreen` | Liste aller Chats (direkt + Gruppen) |
| `ChatListItem` | Einzel-Chat in der Liste (Avatar, Name, letzte Nachricht, Zeit) |
| `ChatThreadScreen` | Nachrichtenverlauf eines Chats |
| `GroupInfoSheet` | Bottom-Sheet mit Gruppen-Infos, Mitgliedern verwalten |
| `NewChatDialog` | Neuen Chat anlegen (Mitglieder wählen) |
| `MessageBubble` | Einzelne Nachricht (eigene/fremde, ✓/✓✓ Lesestatus) |
| `DaySeparator` | Datums-Trenner im Verlauf |
| `TypingIndicator` | „schreibt …"-Indikator |
| `AvatarCircle` | Initialen-Avatar mit Farb-Hashing (`avatarColor`) |

## Funktionen

- Senden, Löschen (`deleteMessage`), Lesen-Markieren (`markRead` → `ChatMarkReadRequest`).
- Tipp-Indikator (`sendTyping`), Empfänger sieht „schreibt …".
- Lese-Status: eigene Nachricht `✓`/`✓✓` (aktualisiert per SSE `chat_read`).
- Gruppendetails (Mitglieder hinzufügen/entfernen/lassen, Umbenennen).
- Neue Chats über `NewChatDialog`.
- Nachrichten laden älter nach (Pagination über `before_id`).
- Suche nach Chats/Nachrichten.

## Echtzeit (SSE)

- Verbindet sich über `ApiClient.openEventSource(...)` auf `GET /events?token=<JWT>`.
- Events: `message`, `typing`, `message_deleted`, `chat_read`, `chat`.
- **Fallback:** 5-Sekunden-Polling (über `ChatRepository.getChatMessages`), wenn SSE nicht verfügbar.
- Verbindung viewModel-scoped → endet beim Verlassen des Screens.

## Verwandt

- [ChatRepository](../repositories/chat-repository.md)
- [API-Client & Netzwerk (SSE)](../03-api-client.md)
- [Backend: Realtime-SSE](../../08-realtime-sse.md)