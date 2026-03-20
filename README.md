# 🏗️ KMP/CMP App Builder — Agent Skill

An [Agent Skill](https://agentskills.io) that guides AI coding assistants to build **production-ready Kotlin Multiplatform (KMP)** and **Compose Multiplatform (CMP)** applications from scratch to production release.

## What is this?

This is **not a template project** — it's a set of structured instructions (an Agent Skill) that teaches AI coding agents how to build KMP/CMP apps following industry best practices. Drop it into any AI coding assistant that supports the [Agent Skills specification](https://agentskills.io/specification).

## What it covers

| Phase | Topics |
|-------|--------|
| **🚀 Setup** | Multi-module Clean Architecture, Gradle convention plugins, version catalog, core modules |
| **🔨 Development** | Feature modules (MVI), Ktor networking, Room database, Koin DI, Navigation Compose, offline-first, WebSocket, security (bearer auth, token refresh), Firebase push notifications, testing (Kover, Turbine), Chucker |
| **📦 Production** | ProGuard/R8, app signing, CI/CD (GitHub Actions), performance optimization, pre-release checklist |

## Installation

### Option 1: Git submodule (recommended)

```bash
git submodule add https://github.com/firdaus1453/kmp-skill.git .agent/skills/kmp-cmp-app-builder
```

### Option 2: Manual copy

Copy the `.agent/skills/kmp-cmp-app-builder/` folder into your project's `.agent/skills/` directory.

### Option 3: Clone standalone

```bash
git clone https://github.com/firdaus1453/kmp-skill.git
```

## File structure

```
.agent/skills/kmp-cmp-app-builder/
├── SKILL.md                          # Main skill file (~1,235 tokens)
├── references/
│   ├── PROJECT_SETUP.md              # Gradle, version catalog, build files
│   ├── CONVENTION_PLUGINS.md         # 7 convention plugins
│   ├── CORE_MODULES.md              # Domain, data, presentation, design system
│   ├── FEATURE_MODULES.md           # MVI, Room, Koin DI, navigation
│   ├── APP_WIRING.md               # Koin graph, NavHost, platform entry points
│   ├── SECURITY.md                 # Bearer auth, token refresh, API keys
│   ├── OFFLINE_FIRST.md            # Room + network sync, connectivity, WebSocket
│   ├── TESTING.md                  # Kover, unit tests, Turbine, Chucker
│   ├── CODE_CONVENTIONS.md         # Naming rules, MVI patterns, coding standards
│   └── PRODUCTION.md              # ProGuard, signing, CI/CD, performance
└── scripts/
    └── validate-module.sh          # Module structure validator
```

## Technology stack

| Technology | Purpose |
|-----------|---------|
| Kotlin 2.1+ | Language |
| Compose Multiplatform 1.7+ | UI framework |
| Koin 4.0+ | Dependency injection |
| Ktor 3.0+ | HTTP client + WebSockets |
| Room 2.7+ | Local database (offline-first) |
| Navigation Compose | Type-safe navigation |
| DataStore | Session/key-value storage |
| Kover | Test coverage reports |
| Turbine | Flow/Channel testing |
| Chucker | HTTP debugging (Android) |
| Firebase Messaging | Push notifications (Android + iOS) |
| BuildKonfig | Build-time constants |
| Kermit | Multiplatform logging |
| Coil | Image loading |
| MOKO Permissions | Permissions |

## Architecture

```
composeApp → core/* + feature/*/
feature/*/presentation → feature/*/domain + core/presentation + core/designsystem
feature/*/data → feature/*/domain + core/domain + core/data
feature/*/database → feature/*/domain
core/data → core/domain
core/presentation → core/domain + core/designsystem
```

## Usage examples

Once installed, ask your AI assistant:

- *"Create a new KMP app with auth and chat features"*
- *"Add an offline-first profile feature module"*
- *"Set up Kover test coverage for all modules"*
- *"Configure ProGuard and prepare release build"*
- *"Add bearer token authentication with refresh"*
- *"Set up CI/CD with GitHub Actions"*

## Spec compliance

Built following the [Agent Skills specification](https://agentskills.io/specification):

- ✅ SKILL.md body ~1,235 tokens (recommended < 5,000)
- ✅ Description 769 chars (max 1,024)
- ✅ Progressive disclosure via `references/`
- ✅ Validation script in `scripts/`
- ✅ Imperative description with implicit triggers

## License

MIT
