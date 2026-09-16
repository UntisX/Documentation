# Backend-Setup (Docker)

> Konfiguration und Betrieb des UntisX-Servers (Rust/Axum) mit PostgreSQL über Docker Compose.

---

## 1. Komponenten

| Container | Image | Port | Zweck |
|-----------|-------|------|-------|
| `postgres` | `postgres:17-alpine` | 5432 (Dev nur!) | Datenbank |
| `server` | Eigenbau (Multi-Stage-Rust) | **3000** | API Server |

---

## 2. Umgebungsvariablen

### `.env.example` (Vorlage)

```env
POSTGRES_USER=admin
POSTGRES_PASSWORD=########           # ← MUSS gesetzt werden
POSTGRES_DB=db
BOOTSTRAP_TOKEN=########             # ← MUSS gesetzt werden (openssl rand -hex 32)
CORS_ORIGIN=localhost:5173,localhost:3000
ENCRYPTION_SECRET=########           # ← Muss mit Frontend VITE_ENC_SECRET übereinstimmen
```

### Alle Runtime-Variablen

| Variable | Pflicht | Standard | Beschreibung |
|----------|---------|----------|--------------|
| `DATABASE_URL` | **Ja** | – | PostgreSQL-Connection-String (wird von `main.rs` aus `POSTGRES_*` in Compose zusammengesetzt) |
| `BOOTSTRAP_TOKEN` | Nein | – | Header-Wert für `X-Bootstrap-Token` (Admin-Erstellung) |
| `BIND_ADDR` | Nein | `0.0.0.0:3000` | TCP-Listen-Adresse |
| `REQUEST_TIMEOUT_SECS` | Nein | `30` | Globales Request-Timeout |
| `DATABASE_MAX_CONNECTIONS` | Nein | cores*2 (5–50) | Pool-Größe |
| `DATABASE_MIN_CONNECTIONS` | Nein | `1` | Mindest-Pool |
| `DATABASE_ACQUIRE_TIMEOUT_SECS` | Nein | `10` | Pool-Timeout |
| `ENCRYPTION_SECRET` | **In Release ja** | Dev-Fallback | AES-256-GCM Ableitung |
| `CORS_ORIGIN` | Nein | `localhost:5173,localhost:3000` | Erlaubte Origins, kommasepariert |

### Compose-only Variablen

| Variable | Standard |
|----------|----------|
| `POSTGRES_USER` | `admin` |
| `POSTGRES_PASSWORD` | (in `.env` setzen) |
| `POSTGRES_DB` | `db` |

---

## 3. Ablauf

```bash
cp .env.example .env
# → .env editieren: POSTGRES_PASSWORD + BOOTSTRAP_TOKEN + ENCRYPTION_SECRET

docker compose up --build -d
```

### Hinweise

- **Produktion:** `ports: 5432:5432` aus der Compose-Datei **entfernen**, damit die DB nicht von außen erreichbar ist.
- Datenbank-Persistenz: Docker-Volume `database_data`.
- Migrations: Der Server fährt bei Start alle 36 SQL-Migrationen selbst hoch (`sqlx::migrate!`).
- Logs: `docker compose logs -f server`

---

## 4. Erstes Bootstrap (Admin anlegen)

```http
POST /bootstrap
Header: X-Bootstrap-Token: <BOOTSTRAP_TOKEN>
Body (JSON):
{
  "user_name": "admin",
  "real_name": "Administrator",
  "email": "admin@schule.de",
  "password": "DeinSicheresPasswort"
}
```

| Fall | Antwort |
|------|---------|
| Erfolg | `201 Created` |
| Benutzername schon vergeben / Admin existiert | `409 Conflict` |

### Sicherheits-Hinweise

- Bootstrap kann nur EINMAL erfolgreich sein (ON CONFLICT DO NOTHING).
- Danach allen Zugriff über normale `/auth/login` nutzen.
- Bootstrap ist die einzige Möglichkeit, ohne Token einen Admin zu erzeugen.

---

## 5. Health-Check

```bash
curl http://localhost:3000/health
# → "Server is alive"
```

---

## 6. Struktur der Server-Repos

```
UntisX-Server/
├── api/                    # DTOs (nur Typen, keine Logik)
│   ├── Cargo.toml          # serde, serde_json, chrono
│   └── src/routs/…         # Typen je Bereich (auth, user, chats, …)
├── server-basis/           # Framework
│   ├── src/server.rs       # Server Supertrait (35 Services)
│   ├── src/lib.rs          # server<S: Server>() → Router
│   ├── src/crypto.rs       # AES-256-GCM Middleware
│   ├── src/validate.rs     # validate_user, validate_admin, …
│   ├── src/default_routs.rs# /health, 404, 405
│   └── src/routs/…         # Jeder Bereich ein Router
├── server-default/         # PostgreSQL-Implementierung
│   ├── Dockerfile          # Multi-Stage Build
│   ├── src/main.rs         # Binary: PgPool + broadcast + Migrationen
│   ├── src/routs/…         # SQL-Implementierungen
│   └── migrations/         # 40 SQL-Migrationen
├── Compose.yaml
├── .env.example
└── README.md
```

---

## 7. Build-Optimierung (Release-Profil)

In `server-default/Cargo.toml`:

```toml
[profile.release]
opt-level = 3
lto = "fat"
codegen-units = 1
panic = "abort"
strip = "symbols"
```

→ Sehr kleine, schnelle Binaries. Build dauert dafür länger.

---

## 8. Troubleshooting

| Problem | Lösung |
|---------|--------|
| `connection refused` bei DB | Erst `postgres` starten (healthcheck `pg_isready`), Server wartet automatisch |
| Bootstrap gibt 409 | Admin existiert bereits – evtl. andere DB-Daten löschen: `docker compose down -v` |
| CORS-Fehler im Browser | `CORS_ORIGIN` um Frontend-Origin ergänzen |
| Migrationsfehler | DB ist älter als Repo → neue Migrationen laufen automatisch; bei Konflikten: `docker compose down -v` (Datenverlust!) |
| Port 3000 belegt | `BIND_ADDR=0.0.0.0:3001` + Frontend-Proxy/Port anpassen |