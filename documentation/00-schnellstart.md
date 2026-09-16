# Schnellstart

> In 5 Minuten von null zum lauffähigen UntisX-System.

---

## Voraussetzungen

- **Docker** mit Docker Compose v2
- **Node.js** 18+ und npm
- **Git**

---

## 1. Backend starten

```bash
# Repository klonen
git clone https://github.com/UntisX/UntisX-Server.git
cd UntisX-Server

# Umgebungsdatei erstellen
cp .env.example .env

# BOOTSTRAP_TOKEN generieren (Windows PowerShell):
# $token = -join ((0..23) | ForEach-Object { '{0:x2}' -f (Get-Random -Max 256) }); Write-Output $token

# BOOTSTRAP_TOKEN in .env eintragen
# POSTGRES_PASSWORD festlegen
# CORS_ORIGIN auf den Frontend-Port setzen (z.B. localhost:5173)
# ENCRYPTION_SECRET setzen (gleicher Wert wie im Frontend)

# Docker starten
docker compose up --build -d
```

Der Server läuft nun unter `http://localhost:3000`.

### Ersten Admin erstellen

```bash
curl -X POST http://localhost:3000/bootstrap \
  -H "Content-Type: application/json" \
  -H "X-Bootstrap-Token: DEIN_BOOTSTRAP_TOKEN" \
  -d '{
    "user_name": "admin",
    "real_name": "Administrator",
    "email": "admin@schule.de",
    "password": "SicheresPasswort123!"
  }'
```

Antwort: `201 Created`

---

## 2. Frontend starten

```bash
# Im separaten Terminal
git clone https://github.com/UntisX/WebVersion.git
cd WebVersion/client

# Dependencies installieren
npm install

# Starten
npm run dev
```

Das Frontend läuft nun unter `http://localhost:5173`.

### Login

1. Öffne `http://localhost:5173`
2. Nutze die Credentials des Admin-Accounts (user_name + password)
3. Du siehst das Dashboard

---

## 3. Verschlüsselung konfigurieren

Für den Produktivbetrieb müssen Frontend und Backend denselben `ENCRYPTION_SECRET` verwenden:

```env
# In UntisX-Server/.env
ENCRYPTION_SECRET=MeinSichererLangerSchluessel123!

# In WebVersion/client/.env
VITE_ENC_SECRET=MeinSichererLangerSchluessel123!
```

> Ohne diesen Wert wird ein Standard-Entwicklungsschlüssel verwendet.

---

## 4. Erste Schritte mit der API

```bash
# Login
TOKEN=$(curl -s -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"user_name":"admin","password":"SicheresPasswort123!"}' \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['token'])")

# Benutzerinfo abrufen
curl -H "Authorization: Bearer $TOKEN" http://localhost:3000/auth/me

# Benutzer anlegen
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "user_name": "mueller",
    "real_name": "Max Müller",
    "short_name": "MM",
    "role": "teacher"
  }'

# Fächer anlegen
curl -X POST http://localhost:3000/school/subjects \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"name":"Mathematik","short_name":"MATH","color":"#FF5733"}'

# Stundenplaneintrag anlegen
curl -X POST http://localhost:3000/timetable \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "subject_id": 1,
    "teacher_id": 1,
    "class_id": 1,
    "first_started_at": "2026-09-01",
    "first_ended_at": "2026-09-01",
    "repeats_every_days": 7,
    "valid_until": "2027-06-30"
  }'
```

---

## 5. Nächste Schritte

- [Architektur verstehen](01-architektur.md)
- [Alle API-Endpunkte ansehen](09-api-suchindex.md)
- [Eigenes Frontend verbinden](04-eigenes-frontend.md)
- [Datenbank-Schema ansehen](07-datenbank.md)

---

## Häufige Fehler

| Fehler | Ursache | Lösung |
|--------|---------|--------|
| `409 Conflict` | Admin existiert bereits | Bootstrap kann nur EINMAL aufgerufen werden |
| `401 Unauthorized` | Token abgelaufen | Erneut einloggen (Token gilt 7 Tage) |
| `403 Forbidden` | Keine Admin-Rechte | Nur Admins können Benutzer/Ressourcen anlegen |
| `Connection refused` | Server nicht erreichbar | Prüfe ob Docker läuft: `docker compose ps` |
