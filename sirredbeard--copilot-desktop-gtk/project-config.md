---
trigger: always_on
description: Maintainer and coding-agent norms. User-facing install docs stay in `README.md`. Do not dump this level of detail into the README.
---

# copilot-desktop-gtk - project instructions

Maintainer and coding-agent norms. User-facing install docs stay in `README.md`. Do not dump this level of detail into the README.

## What this is

Unofficial self-contained Native AOT .NET 11 GTK4/WebKit app for Microsoft Copilot on Linux x86_64. Display name is Copilot, not "Copilot Desktop". Distribution is Flatpak from the GitHub Pages ostree repo (primary, with updates), plus single-file `.flatpak` and bare AOT binary on GitHub Releases.

Not affiliated with Microsoft. Trademark and third-party notices live in `NOTICE`. Icon source is `assets/icons/copilot.png`.

## Non-negotiable norms

1. **Podman for local builds.** `./scripts/build-builder-image.sh` and `./scripts/podman-build-local.sh`. GHCR image: `ghcr.io/sirredbeard/copilot-desktop-gtk-builder`.
2. **Automate in GitHub Actions.** Weekly builder image + manual release dispatch. Prefer fixing Actions over "works on my machine."
3. **Builder image is the product build environment.** Azure Linux 4 base (`mcr.microsoft.com/azurelinux-beta/base/core:4.0`), Fedora 43 repos for GTK/WebKit/Flatpak tooling, side-loaded latest .NET 11 SDK (preview, RC, or GA as available). Flatpak tooling lives in the image. Release must not install Flatpak tooling on the Ubuntu runner; use the builder image. Rebuild weekly and on `container/**` changes. Local image builds use **podman**.
4. **Latest toolchains.** Latest .NET 11 SDK via `scripts/resolve-dotnet-sdk.sh`, newest GirCore that works, GNOME Flatpak runtime 50 (or current stable).
5. **.NET best practices.** `net11.0`, Native AOT (`PublishAot`), trimmed self-contained, `OptimizationPreference=Speed`, file-scoped namespaces, nullable enable. Prefer XDG paths. Binary should NEED only libc/libm; GTK/WebKit load at runtime from host or Flatpak runtime.
6. **Flatpak for the modern desktop path.** `org.gnome.Platform//50`, finish-args for Wayland, network, Pulse/PipeWire, Camera portal (no `--device=all`), CUPS, host fonts, XDG dirs for drag-drop, portals, autostart, persist data dir. No `org.freedesktop.secrets` or bare `org.freedesktop.DBus` talk-name. Single-file `.flatpak` bundle for Releases. Install LICENSE + complete AppStream metainfo so GNOME Software shows name, license, homepage, VCS, and release notes.
7. **GNOME-first. Tray is not a product feature.** Default GNOME (Fedora, Azure Linux with GNOME, etc.) has no system tray. Do not design around tray, do not bundle AppIndicator, do not center docs on tray. Autostart opens a normal window. Soft-load tray only if a host already has StatusNotifier + library; otherwise normal window and close quits.
8. **Login must persist.** `WebKit.NetworkSession` with on-disk cookies and website data under XDG. ITP off enough for MS SSO. Smoke tests may use ephemeral sessions.
9. **Stay inside the WebView for first-party traffic.** `NewWindowAction` must not `Use()` without a create-web-view handler (WebKitGTK will open the default browser). Load allowed hosts in the same window. Allow `copilot.com` / `www.copilot.com` and related Microsoft hosts. Block true external navigations and create-web-view requests (no host browser open).
10. **Writing style.** Docs, comments, commit messages, workflow comments: plain voice. No em-dashes, no decorative emoji, no marketing filler. Spaced hyphen " - " if you need a dash. Precision and brevity over polish. Minimal formatting.
11. **Git authoring.** Never add `Co-authored-by`, Copilot trailers, or π signatures. Iterative commits on `main` are fine.
12. **README hygiene.** User-facing README stays short. Prefer plain lists over tables. Do not mention or compare to other third-party Copilot wrappers. Maintainer GPG and Pages signing detail stays here and in `packaging/gpg/README.md`.
13. **Cancel noise.** Aggressively cancel and delete failed/spurious workflow runs after diagnosis. Get logs, fix, commit, re-run.

## Architecture (short)

- `Program.cs` - Gtk app id, Wayland preference, CLI
- `CopilotApplication.cs` - single-instance, optional tray Hold, Flatpak-aware autostart (windowed)
- `MainWindow.cs` - WebView, chrome, permissions, downloads, print, navigation policy
- `WebKitSession.cs` - persistent NetworkSession / cookies / website data
- `TrayIcon.cs` / `StatusNotifierHost.cs` - optional host tray only; not packaged
- `LoginLogic.cs` / `AppConstants.cs` - allow-listed hosts, URIs, identity strings ("Copilot")

## Builder image details

- Base: `mcr.microsoft.com/azurelinux-beta/base/core:4.0`
- GTK4 / WebKitGTK devel: Fedora 43 Everything + updates (`cost=50`)
- clang AOT link fix: copy Fedora `gcc` multilib into `x86_64-azurelinux-linux` triple
- .NET 11 SDK from release-metadata tarball → `/usr/share/dotnet`
- Flatpak stack in-image; `FLATPAK_USER_DIR=/var/lib/flatpak-builder-user`
- Pre-seeded Flathub install: `org.gnome.Platform//50`, `org.gnome.Sdk//50`, Locale for both, `org.freedesktop.Platform.GL.default//25.08` (+extra), `codecs-extra//25.08-extra` so release does not re-download ~2GB per run. Weekly builder rebuild refreshes them. `build-flatpak.sh` skips install when already present.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sirredbeard/copilot-desktop-gtk](https://github.com/sirredbeard/copilot-desktop-gtk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
