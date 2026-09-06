---
trigger: always_on
description: This is not a plain WinUI template application. It is a packaged, NativeAOT-oriented WinUI 3 desktop app for managing Codex authentication accounts and switching the active Codex account by writing `auth.json` under the user's `.codex` directory.
---

# CliAccountSwitcher.WinUI Agent Guide

## What Makes This WinUI Project Different

This is not a plain WinUI template application. It is a packaged, NativeAOT-oriented WinUI 3 desktop app for managing Codex authentication accounts and switching the active Codex account by writing `auth.json` under the user's `.codex` directory.

Key differences from a typical WinUI project:

- `Program.cs` owns startup because `CliAccountSwitcher.WinUI.csproj` defines `DISABLE_XAML_GENERATED_MAIN`.
- Startup is single-instance. `Program.Main` registers `CliAccountSwitcherWinUI_SingleInstance`; secondary activations are redirected to the current instance.
- `App.xaml.cs` is the service composition root. It creates and stores static service instances, starts background services, handles startup-task launches, handles notification activations, and creates `MainWindow`.
- The app is tray-aware. `Views/MainWindow.xaml` uses `H.NotifyIcon.WinUI`; closing the main window hides it instead of exiting unless the close comes from system shutdown or the tray Exit command.
- The main window derives from `WinUIEx.WindowEx`, uses a custom WinUI `TitleBar`, and hosts page navigation inside `AppFrame`.
- Main navigation is split between `Views/MainWindow.xaml.cs` and `Pages/MainPage.xaml.cs`. `MainWindow` owns the top selector and broadcasts `MainPageNavigationSection` changes through `WeakReferenceMessenger`; `MainPage` swaps the section page in `SectionContentFrame`.
- The app uses DevWinUI controls and visuals, including `SelectorBarSegmented`, `BlurEffectControl`, and `LoadingIndicator`.
- The project targets `net10.0-windows10.0.26100.0`, uses `LangVersion` `preview`, has `PublishAot` enabled, and packages through MSIX tooling.
- The WinUI project references `..\CliAccountSwitcher.Api\CliAccountSwitcher.Api.csproj` for Codex API, OAuth, authentication document, and usage client behavior.

## Runtime Flow

When the app starts:

1. `Program.Main` initializes WinRT COM wrappers, registers or redirects the single app instance, then starts `App`.
2. `App.App()` initializes settings, localization, theme, notification, startup registration, store update, account, and restart services.
3. `StoreUpdateService.Start()` and `CodexAccountService.Start()` begin background work before the main window is shown.
4. `App.OnLaunched` creates `MainWindow`. Startup-task activations keep the app in the tray unless the app has been explicitly asked to show the window.
5. `MainWindow` configures icon, title bar, tray commands, window subclassing, theme, localization, and navigates `AppFrame` to `MainPage`.
6. `MainPage` defaults to `DashboardPage`; selector changes navigate to `DashboardPage`, `AccountsPage`, `AboutPage`, or `SettingsPage`.

Notification activation can request navigation to the Accounts section. If the window or `MainWindow.Instance` is not ready yet, `App` stores the pending navigation and applies it after launch completes.

## Account And Data Model

- The active Codex account is the authentication document stored at `Constants.CurrentAuthenticationFilePath`, which resolves to `%USERPROFILE%\.codex\auth.json`.
- App-owned data is stored under `Constants.UserDataDirectory`, which resolves to `%LOCALAPPDATA%\CodexAccountSwitch.WinUI`.
- Saved accounts are persisted in `accounts.json`; settings are persisted in `config.json`; backups default to the `backups` subfolder.
- `CodexAccountService` keeps an in-memory account list guarded by `_accountsLock`, persists through `CodexAccountJsonSerializerContext`, and watches `.codex\auth.json` with `FileSystemWatcher` to detect active-account changes.
- Usage refresh runs in background loops with separate active-account and inactive-account intervals. Settings changes reset those schedules.
- Keep serialization NativeAOT-safe. Add new serialized model types to `Helpers/CodexAccountJsonSerializerContext.cs` instead of relying on reflection-based JSON serialization.

## Localization

- User-facing strings live in `Strings/en-US/Resources.resw` and `Strings/ko-KR/Resources.resw`.
- XAML should prefer `x:Uid` where practical.
- Code-behind localization should go through `App.LocalizationService`.
- `LocalizationService` uses `Microsoft.Windows.ApplicationModel.Resources.ResourceLoader`, not UWP-era resource APIs.
- When adding or changing user-visible text, update both English and Korean resources unless the string is intentionally not localized.

## NuGet And License Metadata

When updating, adding, or removing any NuGet package in `CliAccountSwitcher.WinUI.csproj`, also update `Pages/AboutPage.xaml.cs`.

Specifically, keep `s_thirdPartyLicensePackages` and `CreateThirdPartyLicensePackages()` synchronized with every package reference version, license, author, and project URL shown in the About page's third-party licenses dialog. This is required; do not leave package versions in the project file and About page out of sync.

## Project Structure

- `Controls/`: reusable WinUI controls, currently including active account quota UI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airtaxi/Cli-Account-Switcher](https://github.com/airtaxi/Cli-Account-Switcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
