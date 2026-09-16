# API: Proxy (`/proxy`)

> Lädt externe Seiten und gibt sie als HTML zurück – Basis für iframe-Integrationen (z.B. Moodle).

---

## GET /proxy

**Eingeloggt.** Query: `?url=<url-encoded>`

### Request-Beispiel

```
GET /proxy?url=https%3A%2F%2Fmoodle.schule.de%2Fdashboard
```

### Response (200)

```json
{ "html": "<!-- Proxied content from https://moodle.schule.de/dashboard -->" }
```

---

## Aktueller Status

- Die aktuelle Implementierung ist ein **Stub**: sie liefert nur einen HTML-Kommentar und holt **nicht wirklich** die externe Seite.
- Wenn du eine echte Integration (Moodle & Co.) einbauen willst, musst du die `proxy.rs`-Implementierung in `server-default` ersetzen (z.B. mit `reqwest` + HTML-Sanitierung).

---

## Frontend-Verwendung

- `MoodlePage.tsx` rendert eine Integration (z.B. Moodle) in einem `<iframe>` über `/proxy?url=…`.
- Die Integration wird über `GET /settings/tabs` → `integrations[]` konfiguriert (`id`, `name`, `url`, `icon`, `enabled`, `roles`).

---

## Sicherheits-Aspekte (wenn du Proxy ausbaust)

| Maßnahme | Begründung |
|----------|-----------|
| Nur getraute URLs erlauben | Verhindert SSRF |
| HTML/JS bereinigen | Verhindert Manipulation |
| Timeout & Größenlimit | Verhindert DoS |
| Cookies getrennt halten | Verhindert Session-Hijacking |