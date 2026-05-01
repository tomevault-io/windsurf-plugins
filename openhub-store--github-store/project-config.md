---
trigger: always_on
description: GitHub Store is a cross-platform app store for GitHub releases, built with **Kotlin Multiplatform (KMP)** and **Compose Multiplatform**. Targets **Android** (min API 26) and **Desktop** (Windows, macOS, Linux via JVM).
---

# CLAUDE.md - GitHub Store

## Project Overview

GitHub Store is a cross-platform app store for GitHub releases, built with **Kotlin Multiplatform (KMP)** and **Compose Multiplatform**. Targets **Android** (min API 26) and **Desktop** (Windows, macOS, Linux via JVM).

Package: `zed.rainxch.githubstore` | Version: 1.6.2 (code 13) | Target SDK: 36

## Build & Run Commands

```bash
# Android
./gradlew :composeApp:assembleDebug
./gradlew :composeApp:assembleRelease

# Desktop (run in dev mode)
./gradlew :composeApp:run

# Desktop installers
./gradlew :composeApp:packageExe :composeApp:packageMsi   # Windows
./gradlew :composeApp:packageDmg :composeApp:packagePkg   # macOS
./gradlew :composeApp:packageDeb :composeApp:packageRpm   # Linux

# Full build check
./gradlew build
```

**Requirements:** JDK 21+ (Temurin recommended), Android SDK for Android builds.

## Project Structure

```
composeApp/                          # Main app module (entry points, navigation, DI wiring)
  src/commonMain/                    # Shared UI & app wiring
  src/androidMain/                   # Android entry point (MainActivity)
  src/jvmMain/                       # Desktop entry point (DesktopApp.kt)
core/
  domain/                            # Shared interfaces, models, use cases (no framework deps)
  data/                              # Shared repos, networking (Ktor), database (Room), DI
  presentation/                      # Shared theming (Material 3) & reusable UI components
feature/
  apps/                              # Installed applications management
  auth/                              # GitHub OAuth device flow authentication
  details/                           # Repository details, releases, readme, downloads
  dev-profile/                       # Developer/user profile display
  favourites/                        # Saved favorite repositories (presentation-only)
  home/                              # Main discovery screen (trending, hot, popular)
  profile/                           # User profile, settings, appearance, proxy, Shizuku installer
  search/                            # Repository search with filters
  starred/                           # Starred repositories (presentation-only)
build-logic/convention/              # Custom Gradle convention plugins
```

Each feature has up to 3 sub-modules: `domain/` (interfaces & models), `data/` (implementations & DI), `presentation/` (screens & ViewModels). Some features (favourites, starred) are presentation-only and use core repositories directly.

## Architecture

**Clean Architecture + MVVM** with strict layer separation per feature module:

- **Domain** - Repository interfaces, models, use cases (no framework dependencies)
- **Data** - Repository implementations, Ktor API clients, Room DAOs, DTOs, mappers
- **Presentation** - ViewModels with `StateFlow`/`Channel`, Compose screens

### State Management Pattern

Every screen follows the same State/Action/Event pattern:

```kotlin
class XViewModel : ViewModel() {
    private val _state = MutableStateFlow(XState())
    val state = _state.asStateFlow()  // or .stateIn() with WhileSubscribed

    private val _events = Channel<XEvent>()
    val events = _events.receiveAsFlow()

    fun onAction(action: XAction) { ... }
}
```

- `State` - data class holding all UI state
- `Action` - sealed interface for user input (clicks, refreshes, etc.)
- `Event` - sealed interface for one-off effects (navigation, toasts, scroll)

### Navigation

Type-safe navigation using `@Serializable` sealed interface `GithubStoreGraph`:

```
HomeScreen, SearchScreen, AuthenticationScreen, ProfileScreen,
FavouritesScreen, StarredReposScreen, AppsScreen, SponsorScreen
DetailsScreen(repositoryId, owner, repo, isComingFromUpdate)
DeveloperProfileScreen(username)
```

Routes defined in `composeApp/.../app/navigation/GithubStoreGraph.kt`, wired in `AppNavigation.kt`.

### Dependency Injection

**Koin** - modules defined in each feature's `data/di/SharedModule.kt`, registered in `composeApp/.../app/di/initKoin.kt`. ViewModels injected via `koinViewModel()`.

### Core Modules

| Module | Purpose | Key Contents |
|--------|---------|--------------|
| `core/domain` | Shared contracts | Repository interfaces (`FavouritesRepository`, `StarredRepository`, `InstalledAppsRepository`, `ThemesRepository`, `ProxyRepository`, `RateLimitRepository`), models (`GithubRepoSummary`, `GithubRelease`, `InstalledApp`, `ProxyConfig`, `InstallerType`, `ShizukuAvailability`), system interfaces (`Installer`, `InstallerInfoExtractor`, `InstallerStatusProvider`, `PackageMonitor`) |
| `core/data` | Shared implementations | `HttpClientFactory` (Ktor + interceptors), `AppDatabase` (Room), `ProxyManager`, `TokenStore`, `LocalizationManager`, platform-specific clients (OkHttp for Android, CIO for Desktop), Shizuku integration (Android: `ShizukuServiceManager`, `ShizukuInstallerWrapper`, `ShizukuInstallerServiceImpl`, `AndroidInstallerStatusProvider`; Desktop: `DesktopInstallerStatusProvider`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenHub-Store/GitHub-Store](https://github.com/OpenHub-Store/GitHub-Store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
