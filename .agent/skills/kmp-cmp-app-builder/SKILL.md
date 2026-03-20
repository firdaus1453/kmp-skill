---
name: kmp-cmp-app-builder
description: >
  Build production-ready Kotlin Multiplatform (KMP) and Compose Multiplatform (CMP) applications
  targeting Android, iOS, and Desktop from scratch. Covers multi-module Clean Architecture with
  convention plugins, Gradle version catalog, Koin DI, Ktor networking, Room database, Navigation
  Compose, DataStore, Firebase push notifications, security (bearer auth, token refresh, API key
  management), offline-first architecture, testing (unit tests, Kover coverage, Chucker HTTP
  debugging), code conventions, and platform-specific integrations. Use when the user asks to
  create a KMP/CMP app, add a new feature module, set up multi-module architecture, configure
  Gradle for multiplatform, implement cross-platform features like networking, database, or
  authentication, set up offline-first data sync, configure test coverage, or follow code conventions.
compatibility: >
  Requires Android Studio Ladybug or later (or IntelliJ IDEA with KMP plugin), JDK 17+,
  Xcode 15+ for iOS targets, Kotlin 2.1+, and Compose Multiplatform 1.7+.
metadata:
  author: kmp-expert
  version: "2.0"
---

# KMP/CMP Application Builder

This skill guides building production-grade Kotlin Multiplatform (KMP) apps with Compose Multiplatform (CMP) UI, targeting **Android**, **iOS**, and **Desktop**. It covers the complete project lifecycle: setup, architecture, feature development, and platform-specific integration.

## When to use this skill

- Creating a new KMP/CMP project from scratch
- Setting up multi-module Clean Architecture for a multiplatform project
- Adding new feature modules (auth, chat, profile, settings, etc.)
- Configuring Gradle convention plugins and version catalog
- Implementing networking (Ktor), local database (Room), DI (Koin), or navigation
- Setting up security: bearer auth with token refresh, API key management, session storage
- Building offline-first features with Room as single source of truth + network sync
- Adding connectivity observers and WebSocket real-time connections
- Setting up testing: unit tests, integration tests, Kover coverage reports
- Integrating Chucker for HTTP traffic debugging on Android
- Adding platform-specific code (Android, iOS, Desktop)
- Setting up Firebase push notifications for multiplatform
- Enforcing code conventions and naming rules

## Architecture overview

Use a **multi-module Clean Architecture** with these layers:

```
root/
├── build-logic/              # Gradle convention plugins
│   └── convention/
├── composeApp/               # Main app entry point (all platforms)
├── core/                     # Shared core modules
│   ├── data/                 # Core data (networking, session, HttpClientFactory)
│   ├── domain/               # Core domain (models, Result type, Error types)
│   ├── presentation/         # Core UI utilities (shared composables, UiText)
│   └── designsystem/        # Design system (Theme, colors, typography, components)
├── feature/                  # Feature modules
│   └── <feature-name>/
│       ├── data/             # Feature-specific data sources & repositories
│       ├── database/         # Feature-specific Room database (if needed)
│       ├── domain/           # Feature-specific use cases & models
│       └── presentation/    # Feature-specific screens & ViewModels
├── gradle/
│   └── libs.versions.toml   # Version catalog (single source of truth)
├── iosApp/                   # iOS Xcode project wrapper
├── settings.gradle.kts
└── build.gradle.kts
```

### Module dependency rules

**CRITICAL: Follow these dependency rules strictly:**

1. **domain** modules have ZERO dependencies on other layers — they contain pure Kotlin only (models, interfaces, use cases). No framework imports.
2. **data** modules depend on their **domain** module. They implement repository interfaces and contain networking/database logic.
3. **presentation** modules depend on their **domain** module. They contain ViewModels, screens, and UI state.
4. **database** modules depend on their **domain** module. They contain Room entities, DAOs, and database setup.
5. **composeApp** is the composition root — it depends on ALL modules and wires DI.
6. Feature modules NEVER depend on other feature modules directly.
7. Core modules are shared — feature modules depend on relevant core modules.

```
composeApp → core/* + feature/*/
feature/*/presentation → feature/*/domain + core/presentation + core/designsystem
feature/*/data → feature/*/domain + core/domain + core/data
feature/*/database → feature/*/domain
core/data → core/domain
core/presentation → core/domain + core/designsystem
```

## Step-by-step: Creating a new KMP/CMP project

### Step 1: Initialize the project

Use the Kotlin Multiplatform Wizard or create manually. The critical files to set up first:

1. `settings.gradle.kts` — module declarations
2. `gradle/libs.versions.toml` — all version and dependency declarations
3. `build.gradle.kts` (root) — plugin declarations with `apply false`
4. `build-logic/` — convention plugins

See [the project setup reference](references/PROJECT_SETUP.md) for complete file templates.

### Step 2: Create convention plugins

Convention plugins eliminate duplication across modules. Create these plugins inside `build-logic/convention/`:

| Plugin | Purpose | What it configures |
|--------|---------|-------------------|
| `KmpLibraryConventionPlugin` | Pure Kotlin multiplatform library | Android library, KMP targets, serialization |
| `CmpLibraryConventionPlugin` | Compose Multiplatform library | Extends KmpLibrary + adds Compose plugin & Material3 |
| `CmpFeatureConventionPlugin` | Feature presentation module | Extends CmpLibrary + adds navigation, Koin, ViewModel, core deps |
| `CmpApplicationConventionPlugin` | Main app module | Android app, all KMP targets, Compose |
| `RoomConventionPlugin` | Database modules | Room compiler, KSP, SQLite bundled |
| `BuildKonfigConventionPlugin` | Build config values | BuildKonfig for sharing build constants across platforms |

See [the convention plugins reference](references/CONVENTION_PLUGINS.md) for complete implementation.

### Step 3: Set up core modules

Create the four core modules in order:

1. **core/domain** — `convention.kmp.library` plugin
   - `Result<D, E>` sealed interface for error handling
   - `DataError` sealed interface hierarchy
   - Core model classes
   - Repository interfaces shared across features

2. **core/data** — `convention.kmp.library` plugin
   - `HttpClientFactory` (expect/actual per platform)
   - Auth token interceptor / bearer token handling
   - Session management with DataStore
   - Core networking utilities

3. **core/designsystem** — `convention.cmp.library` plugin
   - `AppTheme` composable with Material3
   - Color scheme (light/dark)
   - Typography scale
   - Reusable UI components (buttons, text fields, loading indicators)

4. **core/presentation** — `convention.cmp.library` plugin
   - `UiText` sealed class for string resources
   - Shared composables
   - Extension functions for UI utilities

See [the core modules reference](references/CORE_MODULES.md) for code examples.
See [the security reference](references/SECURITY.md) for auth, token management, and API key patterns.

### Step 4: Create feature modules

For each feature (e.g., auth, chat, profile):

1. Create the **domain** submodule first (models + interfaces)
2. Create the **data** submodule (repository implementations)
3. Create the **presentation** submodule (ViewModel + screens)
4. Optionally create a **database** submodule (Room entities + DAOs)

See [the feature module reference](references/FEATURE_MODULES.md) for the complete pattern.
See [the offline-first reference](references/OFFLINE_FIRST.md) for offline-first repository, connectivity, and WebSocket patterns.

### Step 5: Wire everything in composeApp

The `composeApp` module is the composition root:

1. Define the Koin module graph (all feature + core DI modules)
2. Set up navigation with `NavHost` and type-safe routes
3. Initialize the app theme from `core/designsystem`
4. Handle platform-specific initialization (splash screen, Firebase, etc.)

See [the app wiring reference](references/APP_WIRING.md) for complete examples.

### Step 6: Platform-specific setup

- **Android**: `AndroidManifest.xml`, `MainActivity`, splash screen, `google-services.json`
- **iOS**: Xcode project, `MainViewController.kt`, `GoogleService-Info.plist`
- **Desktop**: `main()` function, window configuration, system theme detection

### Step 7: Set up testing & coverage

1. Apply Kover convention plugin to testable modules
2. Add root-level Kover aggregation
3. Write unit tests with fakes/stubs for ViewModels and repositories
4. Integrate Chucker for Android debug HTTP inspection

See [the testing reference](references/TESTING.md) for Kover setup, unit test patterns, and Chucker integration.

### Step 8: Follow code conventions

See [the code conventions reference](references/CODE_CONVENTIONS.md) for naming rules, MVI patterns, and coding standards.

## Adding a new feature module — Checklist

When adding a new feature to an existing project:

- [ ] Step 1: Add module entries to `settings.gradle.kts`
- [ ] Step 2: Create `feature/<name>/domain/` with models and repository interface
- [ ] Step 3: Create `feature/<name>/data/` with repository implementation (if needed)
- [ ] Step 4: Create `feature/<name>/database/` with Room setup (if needed)
- [ ] Step 5: Create `feature/<name>/presentation/` with ViewModel, state, events, and screens
- [ ] Step 6: Create Koin DI module for the feature
- [ ] Step 7: Register DI module in `composeApp`
- [ ] Step 8: Add navigation route and screen to the `NavHost`
- [ ] Step 9: Add Kover coverage plugin to testable modules
- [ ] Step 10: Write unit tests (ViewModel + repository) with fakes
- [ ] Step 11: Build and verify on all target platforms
- [ ] Step 12: Run `./gradlew allTests koverHtmlReport` to verify coverage

## Gotchas

- **Compose Multiplatform version must align with Navigation Compose** — mismatched versions cause cache errors. Always check compatibility matrix.
- **iosMain has no `actual` access to Java/Android APIs** — use `expect`/`actual` declarations for platform-specific code.
- **Room KSP requires separate schema directory configuration** per platform. Use the `RoomConventionPlugin` to automate this.
- **`TYPESAFE_PROJECT_ACCESSORS`** must be enabled in `settings.gradle.kts` for `projects.core.data` syntax to work.
- **DataStore file path** must use `expect`/`actual` — each platform stores preferences in different locations.
- **Koin module loading order matters** — core modules must be loaded before feature modules.
- **`resourcePrefix`** is auto-generated from module path in convention plugin — don't set it manually per module.
- Always use **Kermit** (or Timber on Android-only) for logging, never `println()` or `Log.d()`.
- **Desktop target** needs `kotlinx-coroutines-swing` dependency for proper coroutine dispatching.
- **Navigation Compose routes** should be `@Serializable` data objects or data classes for type-safe navigation.
- **Token refresh must skip auth endpoints** — otherwise you get an infinite loop when the refresh token itself expires.
- **Always call `BearerAuthProvider.clearToken()`** on logout — cached tokens persist otherwise.
- **API keys in `local.properties` only** — never commit secrets to version control.
- **Offline-first: UI must observe Room `Flow`, not network responses** — this ensures instant cached data display.
- **Never catch `CancellationException`** — always re-throw it in `try-catch` blocks.
- **Chucker is Android-only** — use the `no-op` variant for release builds to avoid performance impact.
- **Kover `koverVerify` will fail CI** if coverage drops below the configured minimum bound.
- **Use `Dispatchers.setMain(testDispatcher)`** in test `@BeforeTest` and `Dispatchers.resetMain()` in `@AfterTest`.

## Validation

After generating code, verify:

1. Run `./gradlew build` — must compile without errors
2. Run `./gradlew :composeApp:assembleDebug` — Android APK builds
3. Run `./gradlew :composeApp:runDesktop` — Desktop app launches (if Desktop target configured)
4. Open Xcode project and build for iOS simulator (if iOS target configured)
5. Check that all DI modules resolve correctly by running the app
6. Run `./gradlew allTests` — all unit tests pass
7. Run `./gradlew koverHtmlReport` — coverage report generates without errors
8. Run `./gradlew koverVerify` — coverage meets minimum bound (60%+)

## Key technology stack

| Technology | Purpose | Version guidance |
|-----------|---------|-----------------|
| Kotlin | Language | 2.1+ |
| Compose Multiplatform | UI framework | 1.7+ |
| Koin | Dependency injection | 4.0+ |
| Ktor | HTTP client + WebSockets | 3.0+ |
| Room | Local database (offline-first) | 2.7+ |
| Navigation Compose | Screen navigation | Aligned with CMP version |
| DataStore | Key-value storage (session) | 1.1+ |
| Kotlinx Serialization | JSON serialization | 1.7+ |
| Kotlinx Coroutines | Async programming | 1.9+ |
| Kotlinx Datetime | Date/time handling | 0.6+ |
| Coil | Image loading | 3.0+ |
| Kermit | Multiplatform logging | 2.0+ |
| BuildKonfig | Build-time constants | 0.15+ |
| MOKO Permissions | Permission handling | 0.18+ |
| Firebase | Push notifications | via Firebase BOM |
| Kover | Test coverage reports | 0.9+ |
| Turbine | Flow/Channel testing | 1.2+ |
| Chucker | HTTP debugging (Android) | 4.1+ |
