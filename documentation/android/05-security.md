# Sicherheit

> Sicherheitsbetrachtung der Android-App: TLS, Token-Speicherung, Berechtigungen, Network-Security-Config und bekannte Risiken.

---

## Übersicht

| Thema | Status |
|-------|--------|
| HTTPS/TLS | Erzwungen durch **lokale** Enforcement-Regeln, aber kein globales Verbot von Cleartext |
| Token-Speicherung | DataStore Preferences (**nicht verschlüsselt**) |
| Verschlüsselung der Payloads | **Keine** – reine JSON-Übertragung, kein AES/X-Enc wie im Web-Frontend (Server unterstützt verschlüsselte Envelopes, siehe [Verschlüsselung](../frontend/verschluesselung.md)) |
| Header | `Authorization: Bearer <token>`, `Content-Type`, `X-Bootstrap-Token` (nur Bootstrap) |
| Logging | `HttpLoggingInterceptor.Level.BODY` – auch in Release-Builds |
| Network Security | Cleartext global erlaubt (`base-config cleartextTrafficPermitted="true"`) |
| Berechtigungen | minimal (INTERNET + ACCESS_NETWORK_STATE) |
| R8/ProGuard | Aktiviert für Release-Builds |

## Berechtigungen

| Permission | Zweck |
|------------|-------|
| `INTERNET` | API-Kommunikation |
| `ACCESS_NETWORK_STATE` | Netzwerk-Status prüfen |

Keine Berechtigungen für Benachrichtigungen, Standort, Kamera, Speicher oder Kontakte.

## Token-Speicherung (DataStore)

- Der JWT wird unter dem Key `auth_token` in `untisx_prefs.preferences_pb` gespeichert.
- DataStore Preferences sind **nicht** verschlüsselt (keine EncryptedSharedPreferences, kein Keystore).
- Damit ist der Token auf dem Gerät im Klartext lesbar (rooted Device).
- Manuell verschlüsselbar wäre z. B. über Android Keystore + `AES/GCM/NoPadding` – aktuell **nicht** implementiert.

## Network Security Config

```xml
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain>10.0.2.2</domain>
        <domain>localhost</domain>
        <domain>127.0.0.1</domain>
    </domain-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

- Für lokale Hosts (`10.0.2.2`, `localhost`, `127.0.0.1`) ist Cleartext explizit erlaubt (nötig für die Entwicklung gegen den lokalen Rust-Server).
- **Risiko:** `base-config` erlaubt Cleartext auch für **alle** anderen Domains – d. h. `http://` funktioniert überall.

## Auth-Interceptor-Verhalten (Sicherheitsrelevanz)

- Fügt pro Request `Authorization: Bearer <token>` hinzu – Token wird also bei **jeder** Anfrage gesendet.
- Setzt Pfad-Präfix `/api`, außer bei Localhost-Hosts oder wenn Pfad schon `/api/...` ist.

## Logging

- `HttpLoggingInterceptor` läuft mit `Level.BODY` → vollständige Anfrage- und Antwort-Bodies (inkl. Token-führender Headsets) werden geloggt.
- Im Release-Build nicht deaktiviert → **Risiko** bezüglich Log-Leaks (empfohlen: Level in Release auf `NONE`/`BASIC`).

## Bedrohungen & Empfehlungen

| Bedrohung | Aktueller Status | Empfehlung |
|-----------|------------------|------------|
| Man-in-the-Middle (HTTP) | Möglich (`cleartextTrafficPermitted=true`) | `base-config` auf `false`, nur lokale Domains erlauben |
| Token-Leak durch Logs | Möglich (`Level.BODY`) | Release-Log-Level senken; sensitive Header filtern |
| Token-Diebstahl auf Gerät | DataStore im Klartext | Keystore-gebundene Verschlüsselung |
| Replay von Requests | Server-seitig für **verschlüsselte** Requests abgesichert (`X-Req-Id`-Cache, 300 s); die Android-App sendet plaintext und nutzt es daher nicht | Payload-Verschlüsselung auch in der App einführen |
| Deep-Link/Exported-Komponenten | nur `MainActivity` einzeln | Exported-Flags prüfen |

## Bereits vorhandene Schutzmechanismen

- R8/ProGuard aktiv für Release-Builds (wacht z. B. verhindert Code-Inlining durch).
- Minimale Berechtigungsfläche.
- Token wird im `Authorization`-Header übertragen – **auch für SSE**: Der Server akzeptiert das Token seit dem Security-Update in `.events`/`video/signals/stream/{room}` bevorzugt per `Authorization`-Header; `?token=<JWT>` ist nur noch der Abwärts-Fallback. Neue Integrations-Code sollte den Header verwenden (kein Token-Leak in Logs/Historie).