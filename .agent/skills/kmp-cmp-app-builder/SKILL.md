---
name: kmp-cmp-app-builder
description: >
  Complete guide to build, develop, and ship production-ready Kotlin Multiplatform (KMP) and
  Compose Multiplatform (CMP) applications targeting Android, iOS, and Desktop. Covers the
  full project lifecycle from initial setup through active development to production release.
  Use for: creating new KMP/CMP projects, adding feature modules, implementing networking
  (Ktor), database (Room), DI (Koin), navigation, security (bearer auth, token refresh),
  offline-first architecture, testing (unit tests, Kover coverage, Chucker), code conventions,
  CI/CD, release builds (ProGuard/R8, signing, app store submission), performance optimization,
  and debugging. This skill is the single source of truth for the entire app lifecycle.
compatibility: >
  Requires Android Studio Ladybug or later (or IntelliJ IDEA with KMP plugin), JDK 17+,
  Xcode 15+ for iOS targets, Kotlin 2.1+, and Compose Multiplatform 1.7+.
metadata:
  author: kmp-expert
  version: "3.0"
---

# KMP/CMP Application Builder — Full Lifecycle Guide

This skill covers the **complete lifecycle** of a KMP/CMP application: from initial setup (Phase 1) through active development (Phase 2) to production release (Phase 3). Use it at any stage.

## When to use this skill

### Phase 1: Project Setup (from scratch)
- Creating a new KMP/CMP project
- Setting up multi-module Clean Architecture
- Configuring Gradle convention plugins and version catalog
- Setting up core modules (domain, data, presentation, designsystem)

### Phase 2: Active Development (features & iteration)
- Adding new feature modules (auth, chat, profile, settings, etc.)
- Implementing networking (Ktor), local database (Room), DI (Koin), or navigation
- Setting up security: bearer auth with token refresh, API key management, session storage
- Building offline-first features with Room as single source of truth + network sync
- Adding connectivity observers and WebSocket real-time connections
- Adding platform-specific code with `expect`/`actual` (Android, iOS, Desktop)
- Setting up Firebase push notifications
- Writing unit tests, integration tests, and configuring Kover coverage
- Integrating Chucker for HTTP traffic debugging on Android
- Debugging crashes, network issues, database migrations, and DI resolution errors
- Refactoring existing features while maintaining Clean Architecture rules
- Enforcing code conventions and naming rules

### Phase 3: Production & Release
- Configuring ProGuard/R8 rules for release builds
- Setting up app signing (keystores, provisioning profiles)
- Building release artifacts (APK/AAB, IPA, native distributions)
- CI/CD pipeline setup (GitHub Actions)
- Performance optimization (lazy loading, image caching, memory management)
- App Store / Play Store submission preparation

---

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

---

## Phase 1: Project Setup (from scratch)

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
| `KoverConventionPlugin` | Test coverage | Kover plugin with exclusion filters and minimum bounds |

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

### Step 4: Wire everything in composeApp

The `composeApp` module is the composition root:

1. Define the Koin module graph (all feature + core DI modules)
2. Set up navigation with `NavHost` and type-safe routes
3. Initialize the app theme from `core/designsystem`
4. Handle platform-specific initialization (splash screen, Firebase, etc.)

See [the app wiring reference](references/APP_WIRING.md) for complete examples.

### Step 5: Platform-specific setup

- **Android**: `AndroidManifest.xml`, `MainActivity`, splash screen, `google-services.json`
- **iOS**: Xcode project, `MainViewController.kt`, `GoogleService-Info.plist`
- **Desktop**: `main()` function, window configuration, system theme detection

---

## Phase 2: Active Development

### Adding a new feature module — Checklist

When adding a new feature to an existing project:

- [ ] Step 1: Add module entries to `settings.gradle.kts`
- [ ] Step 2: Create `feature/<name>/domain/` with models and repository interface
- [ ] Step 3: Create `feature/<name>/data/` with repository implementation
- [ ] Step 4: Create `feature/<name>/database/` with Room setup (if needed)
- [ ] Step 5: Create `feature/<name>/presentation/` with ViewModel (MVI), state, events, screens
- [ ] Step 6: Create Koin DI module for the feature
- [ ] Step 7: Register DI module in `composeApp`
- [ ] Step 8: Add navigation route and screen to the `NavHost`
- [ ] Step 9: Apply Kover coverage plugin to testable modules
- [ ] Step 10: Write unit tests (ViewModel + repository) with fakes
- [ ] Step 11: Build and verify on all target platforms
- [ ] Step 12: Run `./gradlew allTests koverHtmlReport` to verify coverage

See [the feature module reference](references/FEATURE_MODULES.md) for the complete pattern.

### Building offline-first features

When a feature requires local persistence with network sync:

- [ ] Create Room database module (`feature/<name>/database/`)
- [ ] Implement `OfflineFirst*Repository` — Room DB as source of truth, network fetch + upsert
- [ ] UI observes Room `Flow`, NOT network responses
- [ ] Add `ConnectivityObserver` (expect/actual for Android/iOS/Desktop)
- [ ] Add retry logic with exponential backoff for failed syncs

See [the offline-first reference](references/OFFLINE_FIRST.md) for complete patterns.

### Implementing security

When adding authentication or protected API access:

- [ ] Configure `HttpClientFactory` with Bearer auth + automatic token refresh
- [ ] Store API keys via `BuildKonfig` from `local.properties` (never commit)
- [ ] Implement `SessionStorage` with DataStore for secure token persistence
- [ ] Create `AuthService` interface in domain, `KtorAuthService` in data
- [ ] Clear tokens on logout (`BearerAuthProvider.clearToken()`)

See [the security reference](references/SECURITY.md) for complete patterns.

### WebSocket real-time features

When building real-time functionality (chat, live updates):

- [ ] Use Ktor WebSocket client (`io.ktor:ktor-client-websocket`)
- [ ] Create `WebSocketConnectionClient` with auto-reconnect on connectivity change
- [ ] Use `ConnectionRetryHandler` for exponential backoff
- [ ] Sync incoming WebSocket messages to Room database

See [the offline-first reference](references/OFFLINE_FIRST.md) for WebSocket patterns.

### Writing tests

When writing or adding tests during development:

- [ ] Create `Fake*` stubs for all interfaces (services, repositories, DAOs)
- [ ] Write ViewModel tests using `runTest` + `Turbine` for Flow/Channel testing
- [ ] Write repository tests with fake service + fake database
- [ ] Write domain unit tests for pure Kotlin logic (Result, mappers)
- [ ] Run `./gradlew allTests` to ensure all tests pass
- [ ] Run `./gradlew koverHtmlReport` to check coverage

See [the testing reference](references/TESTING.md) for Kover setup, unit test patterns, and Chucker integration.

### Debugging during development

| Problem | Tool/Approach |
|---------|---------------|
| HTTP request/response issues | Enable Chucker (Android), Ktor Logging plugin (all platforms) |
| Network connectivity issues | Check `ConnectivityObserver`, verify expect/actual per platform |
| DI resolution failures | Check Koin module loading order (core before features) |
| Database migration errors | Check Room schema directory, verify migration steps |
| Compose recomposition issues | Use `Modifier.then()`, check state hoisting, avoid side effects in composables |
| Flow collection issues | Ensure `SharingStarted.WhileSubscribed(5_000)`, check lifecycle scope |
| Platform-specific crashes | Separate `expect`/`actual` logic, use Kermit/Timber for logging |
| Memory leaks | Avoid `GlobalScope`, use `viewModelScope`, cancel coroutines properly |

### Code conventions

Follow these conventions throughout development. See [the code conventions reference](references/CODE_CONVENTIONS.md) for the full ruleset, including:

- File/class/package naming
- MVI naming: `*State`, `*Action`, `*Event`
- ScreenRoot vs Screen pattern
- Mapper convention: `toDomain()`, `toEntity()`, `toDto()`
- Koin module naming: `<layer><Feature>Module`
- General coding rules (no `println`, no `GlobalScope`, re-throw `CancellationException`, etc.)

---

## Phase 3: Production & Release

### ProGuard / R8 rules

Create `composeApp/proguard-rules.pro`:

```proguard
# Keep Kotlinx Serialization
-keepattributes *Annotation*, InnerClasses
-dontnote kotlinx.serialization.AnnotationsKt

-keepclassmembers class kotlinx.serialization.json.** {
    *** Companion;
}
-keepclasseswithmembers class kotlinx.serialization.json.** {
    kotlinx.serialization.KSerializer serializer(...);
}
-keep,includedescriptorclasses class com.mycompany.myapp.**$$serializer { *; }
-keepclassmembers class com.mycompany.myapp.** {
    *** Companion;
}
-keepclasseswithmembers class com.mycompany.myapp.** {
    kotlinx.serialization.KSerializer serializer(...);
}

# Keep Ktor
-keep class io.ktor.** { *; }
-dontwarn io.ktor.**

# Keep Room
-keep class * extends androidx.room.RoomDatabase { *; }
-keep @androidx.room.Entity class * { *; }

# Keep Koin
-keep class org.koin.** { *; }

# Keep BuildKonfig
-keep class **.BuildKonfig { *; }
```

Enable in `composeApp/build.gradle.kts` (Android block):

```kotlin
android {
    buildTypes {
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

### App signing

#### Android keystore

```bash
keytool -genkey -v -keystore release.jks -keyalg RSA -keysize 2048 -validity 10000 -alias release
```

Add to `local.properties` (git-ignored):

```properties
RELEASE_STORE_FILE=release.jks
RELEASE_STORE_PASSWORD=your_password
RELEASE_KEY_ALIAS=release
RELEASE_KEY_PASSWORD=your_password
```

Configure in `composeApp/build.gradle.kts`:

```kotlin
android {
    signingConfigs {
        val localProperties = Properties()
        rootProject.file("local.properties").inputStream().use { localProperties.load(it) }

        create("release") {
            storeFile = file(localProperties.getProperty("RELEASE_STORE_FILE"))
            storePassword = localProperties.getProperty("RELEASE_STORE_PASSWORD")
            keyAlias = localProperties.getProperty("RELEASE_KEY_ALIAS")
            keyPassword = localProperties.getProperty("RELEASE_KEY_PASSWORD")
        }
    }

    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
            // ... ProGuard config above
        }
    }
}
```

#### iOS signing

Configure in Xcode: Team, Bundle Identifier, Provisioning Profile in Signing & Capabilities.

### Release build commands

```bash
# Android — Release AAB (for Play Store)
./gradlew :composeApp:bundleRelease

# Android — Release APK
./gradlew :composeApp:assembleRelease

# Desktop — Native distribution
./gradlew :composeApp:packageDistributionForCurrentOS

# Desktop — Platform-specific
./gradlew :composeApp:packageDmg    # macOS
./gradlew :composeApp:packageMsi    # Windows
./gradlew :composeApp:packageDeb    # Linux

# iOS — Build via Xcode or:
xcodebuild -workspace iosApp/iosApp.xcworkspace -scheme iosApp -configuration Release
```

### CI/CD with GitHub Actions

Create `.github/workflows/build.yml`:

```yaml
name: Build & Test

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'zulu'

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Create local.properties
        run: |
          echo "API_KEY=${{ secrets.API_KEY }}" >> local.properties
          echo "BASE_URL=${{ secrets.BASE_URL }}" >> local.properties

      - name: Build
        run: ./gradlew build

      - name: Run tests
        run: ./gradlew allTests

      - name: Coverage report
        run: ./gradlew koverHtmlReport

      - name: Verify coverage
        run: ./gradlew koverVerify

      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: kover-report
          path: build/reports/kover/html/

  release-android:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'zulu'

      - name: Decode keystore
        run: echo "${{ secrets.RELEASE_KEYSTORE_BASE64 }}" | base64 --decode > release.jks

      - name: Create local.properties
        run: |
          echo "API_KEY=${{ secrets.API_KEY }}" >> local.properties
          echo "RELEASE_STORE_FILE=release.jks" >> local.properties
          echo "RELEASE_STORE_PASSWORD=${{ secrets.RELEASE_STORE_PASSWORD }}" >> local.properties
          echo "RELEASE_KEY_ALIAS=${{ secrets.RELEASE_KEY_ALIAS }}" >> local.properties
          echo "RELEASE_KEY_PASSWORD=${{ secrets.RELEASE_KEY_PASSWORD }}" >> local.properties

      - name: Build Release AAB
        run: ./gradlew :composeApp:bundleRelease

      - name: Upload AAB
        uses: actions/upload-artifact@v4
        with:
          name: release-aab
          path: composeApp/build/outputs/bundle/release/*.aab
```

### Performance optimization

| Area | Optimization |
|------|-------------|
| **Image loading** | Use Coil with disk cache + memory cache. Set `crossfade(true)` for smooth transitions. |
| **Lazy lists** | Use `LazyColumn`/`LazyRow` with `key` parameter. Never use `Column` with `forEach` for large lists. |
| **State management** | Use `derivedStateOf` for computed values. Avoid recomposing entire screens on small state changes. |
| **Network** | Enable response caching in Ktor. Use pagination for large data sets. |
| **Database** | Add indexes to frequently queried columns. Use `@Transaction` for complex queries. |
| **Startup** | Use splash screen to defer heavy initialization. Init Koin lazily where possible. |
| **Memory** | Cancel coroutines in `onCleared()`. Avoid holding `Context` references in ViewModels. |
| **Build speed** | Enable Gradle build cache, configuration cache, and parallel execution in `gradle.properties`. |

### Pre-release checklist

- [ ] All unit tests pass: `./gradlew allTests`
- [ ] Coverage meets minimum: `./gradlew koverVerify`
- [ ] ProGuard/R8 rules configured and tested
- [ ] Release signing configured (keystore / provisioning profile)
- [ ] `local.properties` secrets are NOT committed
- [ ] Firebase config files are in place but git-ignored
- [ ] Chucker uses `no-op` variant in release builds
- [ ] All `Timber.d()` / `Kermit.d()` debug logs reviewed (no sensitive data)
- [ ] Deep links tested on all platforms
- [ ] App version/code bumped in `libs.versions.toml`
- [ ] Release build tested on physical devices (Android + iOS)
- [ ] Desktop native distribution tested
- [ ] CI/CD pipeline green on main branch
- [ ] Play Store / App Store metadata prepared (screenshots, description)

---

## Gotchas

### Setup & Build
- **Compose Multiplatform version must align with Navigation Compose** — mismatched versions cause cache errors. Always check compatibility matrix.
- **`TYPESAFE_PROJECT_ACCESSORS`** must be enabled in `settings.gradle.kts` for `projects.core.data` syntax to work.
- **`resourcePrefix`** is auto-generated from module path in convention plugin — don't set it manually per module.
- **Desktop target** needs `kotlinx-coroutines-swing` dependency for proper coroutine dispatching.

### Platform-specific
- **iosMain has no `actual` access to Java/Android APIs** — use `expect`/`actual` declarations for platform-specific code.
- **Room KSP requires separate schema directory configuration** per platform. Use the `RoomConventionPlugin` to automate this.
- **DataStore file path** must use `expect`/`actual` — each platform stores preferences in different locations.

### Architecture & DI
- **Koin module loading order matters** — core modules must be loaded before feature modules.
- **Navigation Compose routes** should be `@Serializable` data objects or data classes for type-safe navigation.
- **Feature modules NEVER depend on other feature modules** — communicate via shared domain interfaces or navigation.

### Security
- **Token refresh must skip auth endpoints** — otherwise you get an infinite loop when the refresh token itself expires.
- **Always call `BearerAuthProvider.clearToken()`** on logout — cached tokens persist otherwise.
- **API keys in `local.properties` only** — never commit secrets to version control.

### Data & Offline
- **Offline-first: UI must observe Room `Flow`, not network responses** — this ensures instant cached data display.
- **Never catch `CancellationException`** — always re-throw it in `try-catch` blocks.

### Testing
- **Chucker is Android-only** — use the `no-op` variant for release builds to avoid performance impact.
- **Kover `koverVerify` will fail CI** if coverage drops below the configured minimum bound.
- **Use `Dispatchers.setMain(testDispatcher)`** in test `@BeforeTest` and `Dispatchers.resetMain()` in `@AfterTest`.

### Logging
- Always use **Kermit** (or Timber on Android-only) for logging, never `println()` or `Log.d()`.

### Release
- **ProGuard must keep `@Serializable` classes** — otherwise JSON parsing breaks in release builds.
- **Test release builds on physical devices** — ProGuard/R8 issues only appear in minified builds.
- **Always use `bundleRelease`** (AAB) for Play Store, not `assembleRelease` (APK).

---

## Validation

After generating or modifying code, verify:

### During development
1. Run `./gradlew build` — must compile without errors
2. Run `./gradlew allTests` — all unit tests pass
3. Run `./gradlew koverHtmlReport` — check coverage report
4. Run `./gradlew :composeApp:assembleDebug` — Android debug APK builds
5. Run `./gradlew :composeApp:runDesktop` — Desktop app launches (if configured)
6. Open Xcode and build for iOS simulator (if configured)
7. Check that all DI modules resolve correctly by running the app

### Before release
8. Run `./gradlew koverVerify` — coverage meets minimum bound (60%+)
9. Run `./gradlew :composeApp:assembleRelease` — release APK builds without ProGuard errors
10. Run `./gradlew :composeApp:bundleRelease` — release AAB builds for Play Store
11. Test release build on physical device — verify no crashes from R8 stripping

---

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

## Reference files

| Reference | What it contains |
|-----------|-----------------|
| [PROJECT_SETUP.md](references/PROJECT_SETUP.md) | settings, version catalog, root build, module build.gradle.kts templates |
| [CONVENTION_PLUGINS.md](references/CONVENTION_PLUGINS.md) | All 7 convention plugins with helper extensions |
| [CORE_MODULES.md](references/CORE_MODULES.md) | Core domain/data/presentation/designsystem code patterns |
| [FEATURE_MODULES.md](references/FEATURE_MODULES.md) | Feature module structure, ViewModel MVI, Room, Koin DI, navigation |
| [APP_WIRING.md](references/APP_WIRING.md) | Koin aggregation, NavHost, platform entry points, Firebase |
| [SECURITY.md](references/SECURITY.md) | Bearer auth, token refresh, API key management, session storage |
| [OFFLINE_FIRST.md](references/OFFLINE_FIRST.md) | Offline-first repo pattern, connectivity observers, WebSocket |
| [TESTING.md](references/TESTING.md) | Kover setup, unit test patterns, Turbine, Chucker |
| [CODE_CONVENTIONS.md](references/CODE_CONVENTIONS.md) | Naming rules, MVI conventions, coding standards |
