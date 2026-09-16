# Eigenes Backend verbinden

> Schritt-für-Schritt-Anleitung: baue dein eigenes Backend (oder deinen eigenen Dienst, der mit UntisX interagiert) und verbinde ihn mit dem UntisX-System.

---

## Grundsätzliche Möglichkeiten

| Ziel | Ansatz | Authentifizierung |
|------|--------|-------------------|
| **Einfacher HTTP-Client** (Lesen) | `GET /external/*` | `X-Api-Key` Header |
| **Automatisierung** (Schreiben) | Alle normalen Endpunkte | `Bearer`-Token (muss sich als Benutzer einloggen) |
| **Eigener Server, der UntisX ersetzt** | Server-Trait-Modell übernehmen | Gleiche Verträge wie Rust-Basis |

---

## Variante 1: Externer Key (Nur Lesen)

Für reine Lese-Integrationen (z.B. Anzeige auf einer Homepage) gibt es die `/external/*`-Endpunkte:

```
GET /external/classes        → Liste aller Klassen
GET /external/subjects       → Liste aller Fächer
GET /external/rooms          → Liste aller Räume
GET /external/bell-schedule  → Klingelzeiten
```

### API-Key erzeugen

1. Als Admin einloggen: `POST /auth/login`
2. Key anlegen:

```http
POST /api-keys
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "name": "Homepage-Anzeige",
  "scopes": ["/external/classes", "/external/subjects"],
  "api_type": "read"
}
```

Antwort:

```json
{
  "id": 3,
  "name": "Homepage-Anzeige",
  "key": "untisx_VGhpcyBpc1RoZVNlY3JldEtleQ...",   // NUR JETZT sichtbar!
  "key_hint": "untisx_VGhpcy...",
  "scopes": ["/external/classes", "/external/subjects"],
  "api_type": "read",
  "active": true
}
```

> Der volle Schlüssel wird **nur einmal** (bei Erstellung) zurückgegeben. Er ist nie wieder abrufbar – speichere ihn sofort!

### Verwendung

```
GET /external/subjects
Header: X-Api-Key: untisx_VGhpcyBpc1RoZVNlY3JldEtleQ...
```

### Scope-System

Jede Route ist ein Scope. `api_type` bestimmt die Obergrenze:

| `api_type` | Erlaubt | Hinweis |
|------------|---------|---------|
| `read` | Nur GET | Sicher für öffentliche Anzeigen |
| `write` | GET + POST + PUT + DELETE | Schreibzugriff auf die Scopes |
| `full` | Alles | Volle Kontrolle über die Scopes |

Der Server prüft pro Request gegen die erlaubten Scopes – nicht genutzte Pfade liefern 403.

---

## Variante 2: Als Benutzer agieren (Lesen + Schreiben)

Dein eigener Dienst kann sich mit einem normalen Benutzerkonto anmelden und dann ALLES tun, was dieser Benutzer darf:

```python
import requests

BASE = "http://localhost:3000"

# Login
login = requests.post(f"{BASE}/auth/login", json={
    "user_name": "teacher1",
    "password": "Passwort",
}).json()
token = login["token"]

headers = {"Authorization": f"Bearer {token}"}

# Noten abrufen (Lehrer)
grades = requests.get(f"{BASE}/students/grades", headers=headers).json()

# Hausaufgabe anlegen
hw = requests.post(f"{BASE}/timetable/homework", headers=headers, json={
    "subject_id": 2,
    "title": "Übungsblatt 5",
    "content": "Aufgaben 1–8",
    "date": "2026-09-20"
}).json()
```

### Empfohlene Pattern

| Aufgabe | Endpunkt |
|---------|----------|
| Stundenplan einlesen | `GET /timetable/class-plan?class_id=…` |
| Vertretungsplan abrufen | `GET /timetable/vertretungsplan?date=YYYY-MM-DD` |
| Monatsübersicht | `GET /timetable/month-data?year=&month=` |
| Fehlzeiten pflegen | `POST /students/absences` |
| Noten vergeben | `POST /students/grades` |
| Benachrichtigung per Chat | `POST /chats` + `POST /chats/{id}/messages` |

---

## Variante 3: UntisX-Framework erweitern (Rust)

Das Backend ist als **Framework + Implementierung** aufgebaut. Ein eigenes Backend (z.B. mit anderer DB) erbt einfach das Framework und implementiert die Traits:

```rust
// 1. Trait-Supertrait importieren (server-basis)
use server_basis::server::Server;

// 2. Eigene Struktur definieren
#[derive(Clone)]
struct MyServer {
    database: MyOwnDb,            // deine DB
    event_tx: broadcast::Sender<ChannelEvent>,
}

// 3. Server-Trait implementieren – ALLE 30 Services
impl Server for MyServer { /* ... */ }

// 4. Starten – das komplette Routing + Middleware kommt vom Framework
#[tokio::main]
async fn main() {
    let state = AppState::new(MyServer { ... });
    server_basis::server::server::<MyServer>().with_state(state)
        .await;
}
```

> **Wichtig:** Der `Server`-Trait fordert exakt die 30 Services (Messages, Chats, Timetable, Grades, …). Fehlt einer, kompiliert nichts – das garantiert Vollständigkeit.

---

## Server-seitige Push-Updates (Webhooks-Äquivalent)

UntisX hat kein Webhook-System, aber eine SSE-API. Wenn du Ereignisse **pushen** (empfangen) willst, verbinde dich als Benutzer:

```python
import json
import requests

token = requests.post("http://localhost:3000/auth/login", json={
    "user_name": "admin", "password": "..."
}).json()["token"]

sse_url = f"http://localhost:3000/events?token={token}"

stream = requests.get(sse_url, stream=True, timeout=300)
for line in stream.iter_lines():
    if line and line.startswith(b"data:"):
        event = json.loads(line[5:])
        print(event["kind"], event.get("title"))
        # kinds: message, message_deleted, chat, typing, chat_read, ...
```

---

## Wichtige Regeln für ein eigenes Backend

1. **Auth:** Tokens sind opaque (kein JWT). Der Server hashed dein Token mit SHA-256 und vergleicht mit der DB. Du kannst dir durch `/auth/login` ein Token einholen.
2. **Session-Cleanup:** Abgelaufene Sessions werden automatisch stündlich gelöscht (die Logout-Logik löscht deine Session sofort).
3. **Timeout:** Alle Requests haben ein 30s-Timeout, Maximale Body-Größe 16 MB.
4. **Rate-Limits:** Der Server hat KEINE Rate-Limits (nur globale Limits: Timeout, Body-Limit).
5. **Ids:** Es gibt keine UUIDs – nur sequentielle BIGINT-IDs (`id: 1, 2, 3, …`).
6. **Verschlüsselte Anfragen:** Wenn du Bodies verschlüsselst (`X-Enc: 1`), brauchst du denselben `ENCRYPTION_SECRET`. Sonst liefert der Server `400 Bad Request`.

---

## Fehlerstatus-Codes

| Code | Bedeutung | Typische Ursache |
|------|-----------|------------------|
| `400` | Formatfehler | Falsche Felder, ungültiger Body, falsche Zeitangaben |
| `401` | Nicht authentifiziert | Fehlender/abgelaufener Token |
| `403` | Verboten | Nicht die nötige Rolle (z.B. Schüler will Fächer anlegen) |
| `404` | Nicht gefunden | Falscher Pfad oder ID |
| `409` | Konflikt | Benutzername bereits vergeben, Bootstrap schon gemacht |
| `422` | Validierungsfehler | Feld zu lang/short, ungültige Farbe |
| `500` | Server-Fehler | Interne Exception (Details nie im Response!) |

---

## Komplettes Beispiel: Stundenplan aller Klassen → JSON-Datei

```python
import json, requests

def get_api_key():
    r = requests.post("http://localhost:3000/auth/login", json={"user_name":"admin","password":"..."})
    token = r.json()["token"]
    k = requests.post("http://localhost:3000/api-keys", headers={"Authorization":f"Bearer {token}"},
                      json={"name":"export-bot","scopes":["/external/classes","/external/subjects"],"api_type":"read"})
    return k.json()["key"]

key = get_api_key()
h = {"X-Api-Key": key}

classes = requests.get("http://localhost:3000/external/classes", headers=h).json()
subjects = requests.get("http://localhost:3000/external/subjects", headers=h).json()

data = {"classes": classes, "subjects": subjects}
with open("untisx_export.json", "w") as f:
    json.dump(data, f, indent=2, ensure_ascii=False)
```