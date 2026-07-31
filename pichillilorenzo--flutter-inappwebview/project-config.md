---
trigger: always_on
description: This is the **source of truth** for the entire plugin. All types, enums, and platform contracts are defined here.
---


# Platform Interface Package Instructions

This is the **source of truth** for the entire plugin. All types, enums, and platform contracts are defined here.
Used by all platform implementations and the main plugin-facing-API package.
When adding new features here, you must propagate them also to the main `flutter_inappwebview` package.

## Main Platform Contracts

| Contract Class | File | Description |
|----------------|------|-------------|
| `InAppWebViewPlatform` | `inappwebview_platform.dart` | Factory interface - creates all platform-specific instances |
| `PlatformInAppWebViewWidget` | `in_app_webview/platform_inappwebview_widget.dart` | WebView widget contract |
| `PlatformInAppWebViewController` | `in_app_webview/platform_inappwebview_controller.dart` | WebView controller contract |
| `PlatformHeadlessInAppWebView` | `in_app_webview/platform_headless_in_app_webview.dart` | Offscreen WebView contract |
| `PlatformWebView` | `in_app_webview/platform_webview.dart` | Base WebView interface |
| `PlatformInAppBrowser` | `in_app_browser/platform_in_app_browser.dart` | Browser window contract |
| `PlatformChromeSafariBrowser` | `chrome_safari_browser/platform_chrome_safari_browser.dart` | Chrome Custom Tabs / SFSafariViewController |
| `PlatformCookieManager` | `platform_cookie_manager.dart` | Cookie management contract |
| `PlatformWebStorage` | `web_storage/platform_web_storage.dart` | Storage APIs contract |
| `PlatformFindInteractionController` | `find_interaction/platform_find_interaction_controller.dart` | Find-in-page contract |
| `PlatformPrintJobController` | `print_job/platform_print_job_controller.dart` | Print functionality contract |
| `PlatformPullToRefreshController` | `pull_to_refresh/platform_pull_to_refresh_controller.dart` | Pull-to-refresh contract |
| `PlatformWebMessageChannel/Port/Listener` | `web_message/` | postMessage API contracts |
| `PlatformWebAuthenticationSession` | `web_authentication_session/platform_web_authentication_session.dart` | Auth session contract |
| `PlatformHttpAuthCredentialDatabase` | `platform_http_auth_credentials_database.dart` | Credential storage contract |
| `PlatformProcessGlobalConfig` | `platform_process_global_config.dart` | Process configuration contract |
| `PlatformProxyController` | `platform_proxy_controller.dart` | Proxy settings contract |
| `PlatformServiceWorkerController` | `platform_service_worker_controller.dart` | Service worker contract |
| `PlatformTracingController` | `platform_tracing_controller.dart` | Tracing contract |
| `PlatformWebViewEnvironment` | `webview_environment/platform_webview_environment.dart` | WebView2 environment (Windows) |
| `PlatformWebViewFeature` | `platform_webview_feature.dart` | Feature detection contract |

## Directory Structure

```
lib/src/
├── inappwebview_platform.dart           # Main factory interface
├── main.dart                            # Exports
├── in_app_webview/                      # Core WebView contracts
│   ├── platform_inappwebview_widget.dart/.g.dart
│   ├── platform_inappwebview_controller.dart/.g.dart
│   ├── platform_headless_in_app_webview.dart/.g.dart
│   ├── platform_webview.dart/.g.dart
│   ├── in_app_webview_settings.dart/.g.dart   # InAppWebViewSettings
│   ├── android/                         # Android-specific settings
│   └── apple/                           # iOS/macOS-specific settings
├── in_app_browser/                      # InAppBrowser contracts
├── chrome_safari_browser/               # Chrome/Safari browser contracts
├── find_interaction/                    # Find-in-page contracts
├── print_job/                           # Print job contracts
├── pull_to_refresh/                     # Pull-to-refresh contracts
├── web_message/                         # WebMessage contracts
├── web_storage/                         # WebStorage contracts
├── web_authentication_session/          # Auth session contracts
├── webview_environment/                 # WebView2 environment (Windows)
├── context_menu/                        # Context menu types
├── x509_certificate/                    # X509 certificate parsing
├── types/                               # 200+ shared types & enums
│   ├── navigation_action.dart           # Navigation types
│   ├── url_request.dart                 # Request types
│   ├── permission_*.dart                # Permission types
│   ├── ssl_*.dart                       # SSL types
│   └── ...                              # Many more types
├── platform_cookie_manager.dart/.g.dart
├── platform_http_auth_credentials_database.dart/.g.dart
├── platform_process_global_config.dart/.g.dart
├── platform_proxy_controller.dart/.g.dart
├── platform_service_worker_controller.dart/.g.dart
├── platform_tracing_controller.dart/.g.dart
├── platform_webview_asset_loader.dart/.g.dart
├── platform_webview_feature.dart/.g.dart
├── content_blocker.dart                 # Content blocking rules
├── web_uri.dart                         # URI utilities
└── util.dart                            # General utilities
```

## Key Rules

1. **Code Generation**: Files with `@ExchangeableObject`, `@ExchangeableEnum`, or `@SupportedPlatforms` annotations generate `*.g.dart` files. After modifying, always run:
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pichillilorenzo/flutter_inappwebview](https://github.com/pichillilorenzo/flutter_inappwebview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
