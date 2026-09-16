# Architektur

> Überblick über die Schichten der Android-App `UntisX-Android`, den Datenfluss und die tatsächliche Projektstruktur.

---

## Schichten

```
┌─────────────────────────────────────────────────┐
│                 Compose UI                       │
│  (Screen-Composables sammeln State via           │
│   collectAsState() aus ViewModels)               │
└──────────────────────┬──────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────┐
│              ViewModel (@HiltViewModel)           │
│  (StateFlow<State>, ruft Repository auf)         │
└──────────────────────┬──────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────┐
│              Repository (@Singleton)              │
│  (Gibt ApiResult<T> zurück: Success/Error)       │
└──────────────────────┬──────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────┐
│              ApiClient / UntisXApi (Retrofit)     │
│  (OkHttp + Auth-Interceptor, SSE-Infrastruktur)  │
└──────────────────────┬──────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────┐
│              Backend HTTP API / SSE               │
│  (UntisX-Server, Rust/Axum)                      │
└─────────────────────────────────────────────────┘
```

## Datenfluss

1. **Screen-Composable** (z. B. `PlannerScreen.kt`) sammelt per `collectAsState()` den State aus dem zugehörigen ViewModel.
2. **ViewModel** (@HiltViewModel) hält `StateFlow<State>`, ruft Methoden des Repositories auf und aktualisiert den State.
3. **Repository** (@Singleton) ruft die Retrofit-API auf und kapselt das Ergebnis in `ApiResult<T>` (Success/Error).
4. **ApiClient** baut die `UntisXApi`-Instanz dynamisch auf Basis der gespeicherten Server-URL. Ein OkHttp-Interceptor fügt `Authorization: Bearer <token>` sowie den Pfad-Präfix `/api` hinzu.
5. **SSE**: Chat-Meldungen laufen als Server-Sent Events über OkHttp (`openEventSource`).

## Projektstruktur (tatsächlich)

```
UntisX-Android/
├── app/
│   ├── build.gradle.kts          (App-Dependencies, SDK-Versions)
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── java/com/untisx/app/
│       │   ├── MainActivity.kt           (einzige Activity)
│       │   ├── UntisXApp.kt              (@HiltAndroidApp)
│       │   │
│       │   ├── data/
│       │   │   ├── api/
│       │   │   │   ├── ApiClient.kt      (OkHttp + Retrofit Setup, SSE)
│       │   │   │   └── UntisXApi.kt      (Retrofit-Interface, alle Endpunkte)
│       │   │   ├── local/
│       │   │   │   └── TokenManager.kt   (DataStore-Keys, Token + ServerConfig)
│       │   │   ├── model/                (alle DTOs, siehe [Datenmodelle](07-datenmodelle.md))
│       │   │   └── repository/           (8 Singleton-Repositories, siehe [Repositories](repositories/README.md))
│       │   │
│       │   ├── ui/
│       │   │   ├── navigation/
│       │   │   │   ├── AppNavigation.kt  (NavHost, Start-Destination)
│       │   │   │   └── Screen.kt         (Route-, Titel-, Icon-Definitionen)
│       │   │   ├── screens/              (FLACHE Struktur: alle Screens direkt hier)
│       │   │   │   ├── ServerUrlScreen.kt
│       │   │   │   ├── LoginScreen.kt
│       │   │   │   ├── PlannerScreen.kt
│       │   │   │   ├── AbsencesScreen.kt
│       │   │   │   ├── HomeworkScreen.kt
│       │   │   │   ├── MessagesScreen.kt (Chat-Liste + Chat-Thread in EINER Datei)
│       │   │   │   ├── GradesScreen.kt
│       │   │   │   ├── MoreScreen.kt
│       │   │   │   ├── VertretungsplanScreen.kt
│       │   │   │   ├── AdminPanelScreen.kt
│       │   │   │   ├── AuditLogScreen.kt
│       │   │   │   └── ResourceScreens.kt (Students + Teachers + Subjects + Rooms + BellSchedule)
│       │   │   ├── components/
│       │   │   │   └── CommonComponents.kt  (wiederverwendbare Composables, UntisXLogo u. a.)
│       │   │   └── theme/
│       │   │       ├── Theme.kt          (Material 3 Farbschema, Dark/Light)
│       │   │       ├── Color.kt          (Farbpalette)
│       │   │       └── Type.kt           (Typografie)
│       │   │
│       │   └── di/
│       │       └── AppModule.kt          (Hilt-Modul, aktuell leer)
│       │
│       └── res/
│           ├── values/themes.xml
│           └── xml/network_security_config.xml
├── build.gradle.kts              (Project-Level Plugins)
├── settings.gradle.kts
└── gradle/libs.versions.toml    (Version Catalog)
```

> Hinweis: In der Datei `MessagesScreen.kt` sind sowohl die Chat-Liste als auch der Chat-Thread enthalten (`ChatListScreen` und `ChatThreadScreen`). In `ResourceScreens.kt` liegen alle vier Ressourcen-Screens (Students, Teachers, Subjects, Rooms) plus der BellSchedule-Screen zusammen.

## Core-Patterns

- **ApiResult<T>** – Sealed Class (Success/Error), definiert in `AuthRepository.kt`, wird von allen Repositories genutzt.
- **StateFlow im ViewModel** – jeder Screen besitzt ein `@HiltViewModel` mit `MutableStateFlow`-Feldern.
- **Hilt DI** – Repositories und Manager sind `@Singleton`, ViewModels `@HiltViewModel`.
- **Komponenten** – wiederverwendbare Composables in `ui/components/CommonComponents.kt`.