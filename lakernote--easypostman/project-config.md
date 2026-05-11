---
trigger: always_on
description: EasyPostman is a Java 17 + Swing desktop API testing app. Entry point: `com.laker.postman.App`. Build tool: Maven multi-module.
---

## Project Overview

EasyPostman is a Java 17 + Swing desktop API testing app. Entry point: `com.laker.postman.App`. Build tool: Maven multi-module.

---

## Module Structure

```
easy-postman-parent (root pom.xml, revision = host version)
├── easy-postman-plugin-api      # Stable plugin SPI: EasyPostmanPlugin, PluginContext, PluginDescriptor
├── easy-postman-plugin-bridge   # Shared bridge contracts, models, utils (ConfigPathConstants, AppConstants, I18nUtil, MessageKeys, SystemUtil, UserSettingsUtil, JsonUtil, AppRuntimeLayout); bridge service interfaces (GitPluginService, ClientCertificatePluginService); shared models (Workspace, WorkspaceType, GitAuthType, GitCommitInfo, GitOperationResult, etc.)
├── easy-postman-plugin-ui       # Shared Swing UI base components, FontsUtil, IconUtil, NotificationUtil, EditorThemeUtil, ModernColors, ModernButtonFactory
├── easy-postman-plugin-runtime  # Plugin scan/load/lifecycle: PluginRuntime, PluginScanner, PluginLoader, PluginRegistry
├── easy-postman-plugins/        # Official plugins (each builds an independent JAR)
│   ├── plugin-manager           # Catalog parsing, online/offline install facade
│   ├── plugin-client-cert
│   ├── plugin-capture
│   ├── plugin-redis
│   ├── plugin-kafka
│   └── plugin-decompiler
└── easy-postman-app             # Host application; consumes plugin-registered capabilities
```

When adding shared non-UI logic accessible by both host and plugins, put it in `easy-postman-plugin-bridge`. When adding shared UI utilities, put them in `easy-postman-plugin-ui`. Do not put bridge/SPI code directly into `easy-postman-app`.

---

## Build Commands

```bash
# Full build (all modules + all plugins), skip tests
mvn clean package -DskipTests

# Build only the host app (fastest iteration)
mvn -pl easy-postman-app -am -DskipTests clean package

# Build host app + one plugin
mvn -pl easy-postman-app,easy-postman-plugins/plugin-redis -am clean package -DskipTests

# Quick compile check (no jar, fast)
mvn -q -pl easy-postman-app -am -DskipTests compile

# Run tests for a specific class in headless mode
mvn -q -pl easy-postman-app -am -Dtest=<TestClass> -Dsurefire.failIfNoSpecifiedTests=false -Djava.awt.headless=true test
```

Output: `easy-postman-app/target/easy-postman-${revision}.jar`

Native installers are produced by `build/mac.sh`, `build/win-exe.bat`, `build/linux-deb.sh`, `build/linux-rpm.sh` — these call `jpackage` and reference a fixed filename `easy-postman.jar` (not the versioned one).

---

## Startup Sequence

```
App.main()
  -> configurePlatformSpecificSettings()   // Linux: FlatLaf window decorations
  -> SwingUtilities.invokeLater()
       -> SimpleThemeManager.initTheme()   // reads easy_postman_settings.properties
       -> FontManager.applyFontSettings()
       -> SplashWindow or direct SwingWorker
            -> StartupCoordinator.prepareMainFrame()
                 -> BeanFactory.init("com.laker.postman")   // scans @Component beans
                 -> PluginRuntime.initialize()               // scan, load, lifecycle
                 -> MainFrame (EDT)
  -> registerShutdownHook()
       -> PluginRuntime.shutdown() + BeanFactory.destroy()
```

---

## Custom IOC Container

The project uses its **own lightweight IOC container** (`com.laker.postman.ioc`), not Spring. Do not import Spring annotations.

| Annotation | Purpose |
|---|---|
| `@Component` | Marks a class as a managed bean (auto-scanned from `com.laker.postman`) |
| `@Autowired` | Field/constructor/method injection |
| `@PostConstruct` | Called after all fields are injected |
| `@PreDestroy` | Called on `BeanFactory.destroy()` |

Retrieve beans outside of injection: `BeanFactory.getBean(MyService.class)`.

Three-level circular dependency cache is implemented in `ApplicationContext` — if you see a circular dependency crash, check bean design rather than patching the cache.

---

## Swing Panel Conventions

All UI panels that are logically singletons must:
1. Extend `SingletonBasePanel`
2. Be obtained via `SingletonFactory.getInstance(MyPanel.class)` — **never `new MyPanel()`**
3. Implement `initUI()` for component creation and `registerListeners()` for event wiring
4. Call `safeInit()` after obtaining the instance (this calls `initUI()` then `registerListeners()`)

`SingletonBaseMenuBar` follows the same pattern for menu bars.

---

## Key Constant Files (in `easy-postman-plugin-bridge`)

- `AppConstants` — `APP_NAME`, `BASE_PACKAGE`
- `ConfigPathConstants` — all data file paths (`EASY_POSTMAN_SETTINGS`, `COLLECTIONS`, `ENVIRONMENTS`, `DEFAULT_WORKSPACE_DIR`, etc.)
- `JsonUtil` — Jackson-based JSON serialization/deserialization (supports JSON5/comments); use this instead of raw Jackson calls
- `AppRuntimeLayout` — resolves portable mode (`isPortableMode(Class<?>)`) and key directory paths (`applicationRootDirectory`, `codeSourceDirectory`); portable mode is triggered by a `.portable` marker file or the `easyPostman.portable` system property

Data root: `SystemUtil.getEasyPostmanPath()` — returns `<user.home>/EasyPostman/` in normal mode, or `<app-dir>/data/` in portable mode.

---

## Workspace & Git Sync


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lakernote/EasyPostman](https://github.com/lakernote/EasyPostman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
