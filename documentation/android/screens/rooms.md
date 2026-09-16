# Räume-Screen (Admin)

> Raum-Verwaltung mit Anlegen und Löschen – Teil von `ResourceScreens.kt`.

**Route:** `rooms` (Sub-Screen, Admin)
**Datei:** `ui/screens/ResourceScreens.kt` (`RoomsScreen`)
**ViewModel:** `RoomsViewModel` (`@HiltViewModel`)
**Backing:** `AdminRepository`

---

## Funktionen

- Liste aller Räume (`GET /api/school/rooms`).
- Karte je Raum: Name, Gebäude, Kapazität.
- **Neuen Raum:** FAB → Dialog (Name, Gebäude, Kapazität) → `POST /api/school/rooms` (`RoomCreateRequest(name, building, capacity)`).
- **Raum löschen:** Löschen-Icon → `ConfirmDeleteDialog` → `DELETE /api/school/rooms/{id}` reload.

## ViewModel-Methoden

| Methode | Aufruf |
|---------|--------|
| `loadRooms()` | `GET /api/school/rooms` |
| `deleteRoom(id)` | `DELETE /api/school/rooms/{id}` → reload |
| `createRoom(name, building, capacity)` | `POST /api/school/rooms` → reload |

> `capacity.toIntOrNull() ?: 0` – nicht-numerische Eingaben werden zu 0.

## Verwandt

- [AdminRepository](../repositories/admin-repository.md)
- [Room-DTOs](../07-datenmodelle.md)