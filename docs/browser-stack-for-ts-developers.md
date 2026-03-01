# Браузерная версия — стек и аналогии для PHP/TS-разработчиков

> Для разработчиков с опытом PHP и TypeScript/Node.js, без опыта в Kotlin.

## Что получается на выходе

- Gradle собирает Kotlin → **WebAssembly** + JS-обвязка
- Результат: `composeApp/build/dist/wasmJs/productionExecutable/` — статический сайт (HTML, JS, WASM)
- Деплой: GitHub Pages (как обычный SPA)

---

## Аналогии с PHP / TypeScript

| Kotlin / KMP | PHP / TS аналог |
|--------------|-----------------|
| **Kotlin** | TypeScript: статическая типизация, nullable (`?`), data classes |
| **Compose** | React: декларативный UI, composable-функции вместо компонентов |
| **Koin** | DI-контейнер (аналог NestJS, Inversify или простого фабричного слоя) |
| **Ktor Client** | `fetch` / axios — HTTP-клиент |
| **KStore + kstore-storage** | `localStorage` / IndexedDB через обёртку |
| **OPFS (Origin Private File System)** | Файловая система в браузере (аналог `fs` в Node, но ограниченный) |
| **kotlinx.coroutines** | `async/await` + промисы, но с корутинами вместо промисов |
| **kotlinx.serialization** | `JSON.stringify` / `JSON.parse` или библиотеки сериализации |
| **expect/actual** | Абстракция + разные реализации под платформы (как интерфейс + имплементации) |

---

## Структура браузерного кода

```
composeApp/src/
├── commonMain/          ← общая логика (≈ shared TS-модули)
│   └── kotlin/.../ui/   ← Compose UI (≈ React-компоненты)
├── wasmJsMain/          ← только для браузера (≈ browser-specific TS)
│   ├── main.kt          ← точка входа (≈ index.tsx / main.ts)
│   ├── PlatformModule   ← DI-модуль (≈ провайдеры в Nest/React)
│   ├── interop/         ← обёртки над Web API (AudioContext, MediaRecorder, Geolocation)
│   ├── services/       ← JSAudioRecordingService, OPFSFileSystemService, WasmJSUserLocationProvider
│   └── resources/
│       ├── index.html   ← HTML-оболочка (как в Vite/CRA)
│       └── styles.css
```

---

## Ключевые моменты для Wasm-версии

1. **`main.kt`** — аналог `main.tsx` в React: монтирует Compose в `document.body`.
2. **`PlatformModule`** — регистрирует реализации для браузера: аудио (MediaRecorder), геолокация (Geolocation API), хранилище (OPFS).
3. **`interop/`** — Kotlin-обёртки над Web API (`external class` ≈ `declare` в TS для глобальных объектов).
4. **`index.html`** — подключает `composeApp.js` (сгенерированный Gradle), как `index.html` в Vite подключает бандл.
5. **Сборка** — `./gradlew wasmJsBrowserDistribution` ≈ `npm run build` для SPA.

---

## Синтаксис Kotlin vs TypeScript

| Kotlin | TypeScript |
|--------|------------|
| `val x: String?` | `const x: string \| null` |
| `fun foo(): Unit` | `function foo(): void` |
| `data class User(val name: String)` | `interface User { name: string }` |
| `suspend fun` | `async function` |
| `listOf(1, 2, 3)` | `[1, 2, 3]` |
| `?.` (safe call) | `?.` (optional chaining) |
| `!!` (force unwrap) | `!` (non-null assertion) |

---

## Запуск браузерной версии

```bash
# dev-сервер (≈ npm run dev)
./gradlew :composeApp:wasmJsBrowserDevelopmentRun

# production-сборка (≈ npm run build)
./gradlew wasmJsBrowserDistribution
# Артефакт: composeApp/build/dist/wasmJs/productionExecutable/
```

**Требования к браузеру:** Chrome 119+, Firefox 120+ (или 119 с `javascript.options.wasm_gc`).
