# Frontend-Setup (Vite + React)

> Installation und Konfiguration des offiziellen UntisX-Webclients.

---

## 1. Voraussetzungen

- Node.js 18+
- npm

---

## 2. Installation

```bash
git clone https://github.com/UntisX/WebVersion.git
cd WebVersion/client
npm install
```

---

## 3. Scripts

| Script | Befehl | Zweck |
|--------|--------|-------|
| `dev` | `vite --host` | Dev-Server auf http://localhost:5173 (im Netz erreichbar) |
| `build` | `node ./node_modules/vite/bin/vite.js build` | Produktions-Build inkl. Obfuskierung |
| `preview` | `vite preview` | Produktions-Build lokal testen |

---

## 4. Umgebungsvariablen

Erstelle `.env` aus `.env.example`:

```env
# Verschlüsselungs-Secret (MUSS dem Backend-ENCRYPTION_SECRET entsprechen)
VITE_ENC_SECRET=MeinSichererLangerSchluessel123!
```

> Es gibt **keine weitere** nötige Variable. `VITE_ENC_SECRET` ist die einzige, die im Code gelesen wird (`src/vite-env.d.ts`).

---

## 5. Der Vite-Proxy

Die API-Basis ist immer same-origin `/api`. Das Vite-Proxy leitet weiter:

```ts
// vite.config.ts
server: {
  port: 5173,
  proxy: {
    '/api': {
      target: 'http://localhost:3000', // Backend
      rewrite: (path) => path.replace(/^\/api/, '') // /api wird entfernt
    }
  }
}
```

Konkretes Beispiel:

| Frontend-Aufruf | Nach Proxy | Server-Endpunkt |
|-----------------|------------|-----------------|
| `GET /api/users` | `GET /users` | `/users` |
| `POST /api/auth/login` | `POST /auth/login` | `/auth/login` |
| `GET /api/events?enc=1` (Auth per `Authorization`-Header) | `GET /events?enc=1` | `/events` |

### In Produktion

In Produktion sollte ein Reverse-Proxy (Nginx/Caddy) dasselbe machen: `/api/` an den Rust-Server (Port 3000) weiterleiten und `/api`-Präfix entfernen.

---

## 6. Build & Obfuskierung

Der Produktions-Build obfuskiert den `src/`-Code mit `vite-plugin-javascript-obfuscator`:

- Variablennamen → hex-Buchstaben
- Strings → Base64-Array  
- `debugProtection` (1000ms-Intervall)
- Console-Ausgabe deaktiviert

Das betrifft **nur** `src/`, nicht `node_modules`. Ziel: Schutz des API-Vertrags.

---

## 7. Dev-Server Optionen

- Port: `5173`
- `allowedHosts: ['.localhost']` für LAN-Zugriff konfiguriert.
- Preview-Server nutzt denselben Proxy – funktioniert also auch gegen lokales Backend.

---

## 8. Häufige Fehler

| Fehler | Ursache | Lösung |
|--------|---------|--------|
| 401 dauerhaft | `VITE_ENC_SECRET` stimmt nicht mit `ENCRYPTION_SECRET` überein | Secrets angleichen, Build neu erzeugen |
| Verschlüsselungsfehler | Secret könnte Sonderzeichen enthalten | Langes alphanumerisches Secret verwenden |
| Page leeres Dashboard | Backend-Tab-Konfiguration leer | In AdminPanel oder `settings_tabs`-Tabelle setzen |
| Build dauert lange | Obfuskierung | War einmalig; mit `--no-obfuscate` ggf. nur für Dev |