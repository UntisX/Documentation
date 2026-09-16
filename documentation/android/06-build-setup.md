# Build & Setup

> Build-Konfiguration, Version Catalog, Befehle und Hinweise für die Android-App.

---

## Voraussetzungen

- Android Studio (Ladybug oder neuer)
- JDK 17
- Android SDK 35
- (Für die lokale Entwicklung gegen das Backend: Rust-Server auf `localhost:3000`)

## SDK-Versionen

| Eigenschaft | Wert |
|-------------|------|
| Min SDK | 29 (Android 10) |
| Target SDK | 35 |
| Compile SDK | 35 |

## Gradle

- **Gradle:** 8.7
- **AGP:** 8.5.2
- **Version Catalog:** `gradle/libs.versions.toml` (alle Versionen zentral)
- **Module:** `app` (einziges Modul)

## Build-Befehle

```bash
# Debug-Build
./gradlew assembleDebug

# Release-Build (R8/ProGuard aktiviert)
./gradlew assembleRelease

# Installieren auf Gerät/Emulator
./gradlew installDebug
```

## Abhängigkeiten (Version Catalog)

| Library | Version | Zweck |
|---------|---------|-------|
| Compose BOM | 2024.06.00 | UI-Framework |
| Material 3 | (via BOM) | Design-System |
| Hilt | 2.51.1 | Dependency Injection |
| Retrofit | 2.11.0 | HTTP-Client |
| OkHttp | 4.12.0 | Netzwerk + SSE |
| Moshi | 1.15.1 | JSON-Parsing |
| Navigation Compose | 2.7.7 | Screen-Navigation |
| DataStore | 1.1.1 | Lokale Persistenz |
| Coroutines | 1.8.1 | Async-Programmierung |
| Lifecycle ViewModel Compose | (BOM) | ViewModel/StateFlow in Compose |

## Wichtige App-Konfiguration

- **Einstieg:** `MainActivity.kt` (einzige Activity) → `AppNavigation()`; `UntisXApp.kt` ist `@HiltAndroidApp`.
- **Network Security Config:** `res/xml/network_security_config.xml` (siehe [Sicherheit](05-security.md)).
- **App-Keys in DataStore:** `untisx_prefs` (fine – siehe [Auth & Login](02-auth-login.md)).
- **Hilt-Modul:** `di/AppModule.kt` liegt vor, ist aktuell leer (kein explizites Binding nötig, da alle Klassen `@Inject`-Konstruktoren bzw. `@Singleton` haben).

## Emulator-Hinweis

- Ohne gesetzte Server-URL fällt `ApiClient.getApi()` auf `http://10.0.2.2:3000` zurück – das ist der Loopback des Android-Emulators zum Host.
- `10.0.2.2` ist in der `network_security_config.xml` für Cleartext freigegeben.

## Verzeichnis-Übersicht

```
UntisX-Android/
├── app/
│   ├── build.gradle.kts
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── java/com/untisx/app/...
│       └── res/
│           ├── values/themes.xml
│           └── xml/network_security_config.xml
├── build.gradle.kts
├── settings.gradle.kts
└── gradle/libs.versions.toml
```