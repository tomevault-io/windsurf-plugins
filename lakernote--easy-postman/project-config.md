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
├── easy-postman-foundation      # Lowest non-UI base layer: shared models, constants, paths, JSON, system/settings/i18n utilities
├── easy-postman-request-core    # Headless request spec models: HttpRequestItem, SavedResponse, headers/body/auth/protocol DTOs
├── easy-postman-http-runtime    # Headless HTTP transport runtime: PreparedRequest, HttpResponse, OkHttp/SSL/SSE/Cookie/redirect runtime ports
├── easy-postman-collection-core # Headless collection domain models and Postman collection parser
├── easy-postman-plugin-api      # Stable plugin SPI and service contracts: EasyPostmanPlugin, PluginContext, PluginDescriptor, GitPluginService, ClientCertificatePluginService
├── easy-postman-platform        # Host platform framework: custom IOC + update discovery core; startup/welcome/help/settings orchestration later
├── easy-postman-ui              # Common Swing UI base components, FontsUtil, IconUtil, NotificationCenter, EditorThemeUtil, ModernColors, ModernButtonFactory
├── easy-postman-performance-core           # Headless performance domain core: editable plan contracts, run plan.json, runtime contracts, stats, report snapshots
├── easy-postman-plugin-runtime  # Plugin scan/load/lifecycle: PluginRuntime, PluginScanner, PluginLoader, PluginRegistry
├── easy-postman-plugins/        # Official plugins (each builds an independent JAR)
│   ├── plugin-manager           # Host-side plugin management exception: catalog parsing, online/offline install facade
│   ├── plugin-client-cert
│   ├── plugin-capture
│   ├── plugin-redis
│   ├── plugin-kafka
│   └── plugin-decompiler
└── easy-postman-app             # Host application; consumes plugin-registered capabilities
```

When choosing a module, use this rule set:

- Put shared non-UI foundation logic in `easy-postman-foundation`: DTOs, enums, constants, paths, JSON, system utilities, user-setting helpers, i18n mechanism, and generic parsing/formatting helpers such as Cron, JSON Path, XML, file-size, file-extension, time-display, and HTTP header constants.
- Put request specification models in `easy-postman-request-core`: `HttpRequestItem`, `SavedResponse`, headers, params, form body rows, cookies, request auth/body/protocol enums, redirect and transport-auth metadata. This module must not depend on Swing, OkHttp, app service/panel code, plugin runtime, or concrete send/render implementations.
- Put HTTP transport runtime in `easy-postman-http-runtime`: runtime exchange snapshots (`PreparedRequest`, `HttpResponse`, `HttpEventInfo`), runtime settings/provider, OkHttp adapters, TLS/client certificate ports, Cookie store, SSE callbacks, redirect execution, UI-neutral interaction sinks, and network observation sinks. This module must not depend on Swing/AWT, panel code, app `SettingManager`, app plugin-host accessors, platform IOC, or JavaFX/Swing-specific adapters.
- Put collection domain models and neutral import parsing in `easy-postman-collection-core`: `RequestGroup`, `CollectionNode`, `CollectionNodeType`, `CollectionParseResult`, collection auth parsing helpers, and Postman collection parsing. This module may depend on foundation and request-core, but must not depend on Swing/AWT, OkHttp, app panel/service/runtime, platform, plugin-runtime, IOC, or concrete send/render implementations.
- Put plugin-facing extension contracts in `easy-postman-plugin-api`: plugin SPI, service interfaces, toolbox/script/snippet contracts.
- Put host platform framework in `easy-postman-platform`: custom IOC, update discovery core, then startup, welcome/help, settings center, and theme/font application orchestration when dependencies are ready.
- Put shared Swing design-system code in `easy-postman-ui`: reusable components, UI singleton base/factory, toolbar buttons (`EditButton`, `SaveButton`, `WrapToggleButton`), form controls (`EasyComboBox`, `EasyJSpinner`, `EasyPasswordField`), fonts, icons, notification UI, editor theme helpers, semantic colors, and UI resources directly used by those components.
- Keep icon resource ownership unique: generic action/control/status icons (`save`, `copy`, `paste`, `search`, `clear`, `cancel`, `close`, `delete`, `duplicate`, `eye`, `info`, `warning`, arrows, chevrons, wrap, start/stop, send, connect, collapse/expand, more/detail, import/export) belong in `easy-postman-ui`; app/domain icons belong in their owning app or plugin module. Official plugins may use icons from their own resources or `easy-postman-ui`, not app-only resources. Do not duplicate the same `icons/*.svg` path across app and ui.
- Keep plugin loading, classloaders, descriptor parsing, registry, and lifecycle in `easy-postman-plugin-runtime`.
- Keep host-specific composition, app panels, menus, concrete startup wiring, and app-only services in `easy-postman-app` until each dependency is ready to migrate into `easy-postman-platform`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lakernote/easy-postman](https://github.com/lakernote/easy-postman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
