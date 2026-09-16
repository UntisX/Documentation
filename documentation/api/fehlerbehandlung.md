# Fehlerbehandlung

> Wie die UntisX-API Fehler meldet – ein Standard für alle Endpunkte.

---

## Fehler-Format

Alles Fehler kommen als JSON auf einem zuverlässigen Pfad:

```json
{ "message": "Human-lesbare Fehlermeldung auf Deutsch" }
```

Einige Endpunkte liefern zusätzlich strukturierte Details (z.B. Validierungsfehler). Das Frontend zeigt `message` direkt in Toasts/Alert-Dialogen an.

---

## Statuscodes

| Code | Bedeutung | Häufige Ursachen |
|------|-----------|------------------|
| **200** | OK | Erfolgreiche GET/PUT/PATCH |
| **201** | Erstellt | Erfolgreiche POST (Benutzer, Chats, Hausaufgaben, …) |
| **204** | Kein Inhalt | Erfolgreiches DELETE/Markieren |
| **400** | Ungültige Anfrage | Nicht-parsebares JSON, falsches Envelope-Format |
| **401** | Nicht authentifiziert | Fehlender/abgelaufener Token, ungültiger API-Key |
| **403** | Verboten | Rolle reicht nicht (z.B. Schüler will Fächer anlegen), Key-Scope überschritten |
| **404** | Nicht gefunden | Falsche Route oder Ressource gelöscht |
| **409** | Konflikt | Doppelter `user_name`, Bootstrap bereits durchgeführt |
| **422** | Validierungsfehler | Leere Felder, zu lange Strings, ungültige Farbe, falsche `role` |
| **500** | Interner Fehler | Unerwarteter Fehler (Details NIE im Response) |

---

## Validierungsfehler (422)

- Strings: Längencheck via `validate_string` / `validate_string_with_chars`.
- E-Mail: `validate_email` (Standard-Formatprüfung).
- IDs: `validate_id` (positive Ganzzahl).
- Farben: `validate_color` (`#RRGGBB`).
- Rollen: Whitelist `admin`/`teacher`/`student`.

---

## Auth-Fehler im Kontext

| Zustand | Resultat |
|---------|----------|
| Kein `Authorization`-Header | 401 |
| Token abgelaufen | 401 |
| Token ungültig (Session-Hash nicht gefunden) | 401 |
| Gültig, aber Rolle falsch | 403 |
| Client bekommt 401 mit gültigem Token | Frontend leert Tokens und leitet zu `/login` |

---

## Fehlerbehandlung im Frontend

```
apiRequest() →
  res.ok? → returnen
    : → parse {message} → new ApiError(message, status)
       → status == 401 && Token war gesetzt
            → clearTokens(); window.location.href = '/login'
```

### Beispiel

```js
try {
  const u = await apiRequest('POST', '/users', {...}, token);
} catch (e) {
  if (e.message === 'Benutzername existiert bereits') {
    // Feld-Highlight statt Toast
  }
}
```

---

## Bekannte Einschränkungen (Status 2026)

| Bereich | Einschränkung |
|---------|---------------|
| Audit-Filter | `action`/`user_id`/`from`/`to` werden ignoriert (nur limit/offset) |
| Message-Namen | `sender_*_name`/`receiver_*_name` sind immer `null` |
| Absence-Stats | `late` immer `0` |
| Periodenzeiten | Hard-codiert, `bell_schedule` beeinflusst sie nicht |
| `moodle_url` | Stub (`enabled: false`) |
| Proxy | Stub (liefert nur Kommentar) |
| Substitutions-DELETE | Soft: `status='cancelled'` |
| Rooms/Subjects-DELETE | Soft: `active=false` |

---

## Best Practices für eigene Clients

1. **`message` aus dem Body lesen**, nicht nur den Statuscode.
2. **401-Token-Flow** global behandeln (nicht pro Request).
3. **Retry mit Backoff** bei 429/500-Flüchtigen Fehlern.
4. **204 nicht `res.json()`** – leere Bodies können den Parser crashen.
5. **Validierung serverseitig** verlassen – trotzdem clientseitig spiegeln (UX).