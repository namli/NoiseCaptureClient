# NoiseCapture — обзор кодовой базы

## Описание приложения

**NoiseCapture** — кроссплатформенное приложение (Android / iOS / Web) для измерения шума окружающей среды с помощью микрофона устройства. Позволяет записывать звук, анализировать спектрограммы, привязывать замеры к геолокации и делиться данными с сообществом для создания партисипативных шумовых карт (в рамках инфраструктуры OnoMap SDI). Лицензия — GPLv3.

---

## Стек технологий

| Область | Технологии |
|---------|------------|
| **Язык** | Kotlin 2.3.0, Swift (тонкий слой для iOS) |
| **Мультиплатформа** | Kotlin Multiplatform (KMP) + Compose Multiplatform 1.10.0 |
| **UI** | Jetpack Compose / Material 3, Compose Navigation, Adaptive layouts |
| **DI** | Koin 4.1.1 |
| **Сеть** | Ktor 3.4.0 |
| **Хранение** | KStore 1.0.0, Multiplatform Settings 1.3.0 |
| **Карты** | maps-compose 0.12.0 |
| **Графики** | KoalaPlot 0.10.4 |
| **Сборка** | Gradle (Kotlin DSL), Version Catalog (`gradle/libs.versions.toml`), BuildKonfig |
| **Линтинг** | Detekt |
| **CI/CD** | GitHub Actions (lint, test, build, deploy Wasm на GitHub Pages) |
| **Автоматизация** | Fastlane (iOS TestFlight, Android debug/release) |
| **Целевые платформы** | Android (minSdk 24, targetSdk 36), iOS (arm64, x64, simulator), WasmJS (браузер) |

---

## Структура каталогов

| Каталог | Описание |
|---------|----------|
| **`composeApp/`** | Основной KMP-модуль (библиотека). Содержит всю общую логику и UI: экраны (запись, история, карта, настройки, отладка), сервисы (аудио, геолокация, хранение, права), модели, аудиопроцессинг (FFT, фильтры, спектрограмма). Source sets: `commonMain`, `androidMain`, `iosMain`, `wasmJsMain`, `commonTest`, `androidHostTest`, `iosTest`, `wasmJsTest`. Ресурсы в `composeResources/`. |
| **`androidApp/`** | Тонкая Android-оболочка. `MainActivity` хостит Compose UI из `composeApp`. Манифест, ресурсы, провайдер нотификаций. |
| **`iosApp/`** | SwiftUI-хост для iOS. `ContentView` встраивает Compose через `MainViewController`. Cinterop (`DeviceUtil`), Info.plist, ассеты, Xcode-проект. |
| **`gradle/`** | Gradle wrapper и version catalog (`libs.versions.toml`). |
| **`config/`** | Конфигурация Detekt (`detekt.yml`). |
| **`fastlane/`** | Fastfile: iOS (test, beta), Android (test, build_debug, build_release). |
| **`.github/`** | Workflows: `code_review.yml`, `static.yml` (Wasm deploy). Actions: job-set-up, gradle-cache. |

---

## Архитектура

```
         ┌──────────────┐   ┌──────────┐   ┌──────────────┐
         │  androidApp  │   │  iosApp  │   │  wasmJs/web  │
         │  (thin host) │   │ (SwiftUI)│   │ (browser)    │
         └──────┬───────┘   └────┬─────┘   └──────┬───────┘
                │                │                 │
                └────────┬───────┴─────────┬───────┘
                         │   composeApp   │
                         │ (KMP library)  │
                         ├─ commonMain ───┤  ← общий UI + логика
                         ├─ androidMain   │  ← Android-реализации
                         ├─ iosMain       │  ← iOS-реализации
                         └─ wasmJsMain    │  ← Web-реализации
```

---

## Ключевые пути в `composeApp/src`

| Путь | Назначение |
|------|------------|
| `commonMain/kotlin/org/noiseplanet/noisecapture/` | Точки входа: `App.kt`, `Platform.kt` |
| `commonMain/.../ui/` | Экраны, навигация, компоненты (appbar, spl, plot, audioplayer, map, button), тема |
| `commonMain/.../audio/` | AudioSource, FFT, фильтры, спектрограмма, player |
| `commonMain/.../model/` | DAO, enum'ы |
| `commonMain/.../services/` | measurement, recording, location, storage, audio, permission, settings, statistics |
| `commonMain/.../permission/` | delegates, state |
| `commonMain/.../util/` | Вспомогательные утилиты |
| `commonMain/.../Koin.kt` | DI-модули |
| `androidMain/` | PlatformModule, AndroidRecordingService, AndroidUserLocationProvider, AndroidFileSystemService, AndroidAudioRecordingService, permission delegates, ClipboardUtil, NotificationProvider |
| `iosMain/` | PlatformModule, MainViewController, IOS* services, permission delegates, DeviceUtil cinterop |
| `wasmJsMain/` | PlatformModule, JSAudioRecordingService, OPFSFileSystemService, WasmJSUserLocationProvider, index.html, styles.css |

---

## Локальная разработка

**Требования:** Java 17+ (jenv / OpenJDK 21).

| Платформа | Dev (с hot reload) | Production build | Артефакт |
|----------|-------------------|------------------|----------|
| **Браузер** | `./gradlew :composeApp:wasmJsBrowserDevelopmentRun` | `./gradlew wasmJsBrowserDistribution` | `composeApp/build/dist/wasmJs/productionExecutable/` |
| **Android** | Android Studio или `./gradlew :androidApp:installDebug` | `./gradlew :androidApp:assembleRelease` | APK в `androidApp/build/outputs/` |
| **iOS** | Xcode: `iosApp/iosApp.xcodeproj` | Fastlane `beta` | — |

**Прочее:** `./gradlew detekt` — линтинг. `./gradlew test` — Android-тесты. `./gradlew :composeApp:iosSimulatorArm64Test` — iOS-тесты.

---

## Версии (из `gradle.properties`)

- `appVersionName=0.7.1`
- `appVersionCode=10`
- `appNamespace=org.noiseplanet.noisecapture`
- `appPackageName=org.noiseplanet.noisecapturekmp`

---

## Документация

- [Браузерная версия — стек и аналогии для PHP/TS-разработчиков](browser-stack-for-ts-developers.md)
