---
trigger: always_on
description: Notes techniques sur le projet pour les contributeurs et les sessions
---

# CLAUDE.md

Notes techniques sur le projet pour les contributeurs et les sessions
Claude Code. Pas de prose marketing — uniquement les pièges et décisions
non-évidentes à la lecture du code.

## Projet

Gestionnaire de fenêtres Dofus Unity en Rust (Windows). Permet de focus
n'importe quel compte lancé via raccourcis clavier/souris globaux.

## État actuel

Application fonctionnelle. `cargo test` doit rester vert et `cargo build`
sans warning. À jour à la phase 15 (modal "À propos").

## Conventions

- **Crate name** : `rorganizer` (le dossier reste `ROrganizer`).
- **License** : double `MIT OR Apache-2.0`.
- **Manifeste Windows** : `asInvoker`. Pas de `requireAdministrator`
  malgré la phase 5 — les hooks LL marchent en user-process tant que
  Dofus tourne aussi en user. Élever le manifeste forcerait l'utilisateur
  à passer une UAC à chaque lancement sans gain.
- **Pas de logging fichier**. L'app n'écrit que dans
  `%APPDATA%\rorganizer\config.json`. Aucun temp file, aucun log keystrokes.
- **Pas de réseau** : zéro socket sortant.

## Stack & deps clés

- `eframe 0.29` avec `default-features = false, features = ["glow", "default_fonts"]`.
  ⚠ Ne PAS retirer plus de features — tester `default-features = false`
  sur `egui` casse la création de fenêtre (font lookup).
- `egui_extras 0.29` avec uniquement le feature `svg`. Le PNG loader n'est
  PAS compilé : un `Image::from_bytes("bytes://*.png", ...)` rend le
  placeholder rouge d'erreur. Décoder les PNGs via la crate `image` puis
  `ctx.load_texture(...)` à la place. Voir `App::about_icon`.
- `tray-icon 0.19`.
- `windows 0.58` features : `Win32_Foundation`, `Win32_UI_WindowsAndMessaging`,
  `Win32_UI_Input_KeyboardAndMouse`, `Win32_System_Threading`,
  `Win32_System_ProcessStatus`, `Win32_Globalization`, `Win32_Security`,
  `Win32_UI_Accessibility`.
- `raw-window-handle 0.6`.
- `arc-swap 1` pour `WindowsSnapshot` (lecture lock-free hot path).
- `webbrowser 1` pour ouvrir les URLs (évite la console qui flashe avec
  `cmd /C start`).
- `image 0.25` (png only) + `ico 0.3` en build-deps pour générer le `.ico`
  multi-résolution.
- Pas de `tokio`, pas d'`async-runtime`. Threads natifs uniquement.

## Build & run

Toolchain : `stable-x86_64-pc-windows-msvc`.

```powershell
$env:Path = "$env:USERPROFILE\.cargo\bin;$env:Path"
cargo build --release          # ~50 s, ne pas distribuer un debug build
.\target\release\rorganizer.exe
```

`windows_subsystem = "windows"` en release retire la console — `eprintln!`
est perdu. Pour du debug rapide, build sans le flag (`cargo build`) et
lancer depuis un terminal. Pas de log fichier en prod.

## Gotchas critiques

### eframe + viewport caché

Quand `Visible(false)`, eframe **parque sa main loop**. Conséquences :
- `Context::request_repaint()` ne réveille PAS `update()`.
- `send_viewport_cmd` s'accumule sans être traité.
- `mpsc::channel` consommés dans `update()` ne sont jamais drainés.

**Toute action depuis un handler tray doit être faite en Win32 direct**
(les handlers `MenuEvent::set_event_handler` et
`TrayIconEvent::set_event_handler` tournent automatiquement sur le
thread main) :
- **Show** : `ShowWindow(hwnd, SW_SHOW)` + `SetForegroundWindow(hwnd)`.
- **Close** : `ShowWindow(hwnd, SW_SHOW)` PUIS
  `PostMessageW(hwnd, WM_CLOSE)` (sans le ShowWindow préalable, WM_CLOSE
  reste dans la queue jusqu'au prochain réveil).

Le HWND principal est capturé en `App::new` via `cc.window_handle()` →
`tray::MAIN_HWND`.

### Tray Activer ↔ état viewport eframe désynchronisé

Quand on toggle Activer/Désactiver depuis le tray, le handler fait un
Win32 ShowWindow direct (pour réveiller la loop parquée). Mais eframe ne
sait pas que la fenêtre est maintenant visible. Si on lui envoie
`Visible(false)` ensuite, il dédupe (croit déjà caché) → no-op → fenêtre
reste visible.

**Fix** : toujours envoyer `Visible(true)` AVANT `Visible(false)` dans le
handler `TrayEvent::ToggleRequested` pour resync l'état. Voir
`App::update`.

### Single-instance — race au démarrage

Le `CreateEventW` doit être fait **avant** `eframe::run_native`, pas dans
`App::new`. Sinon une 2ème instance lancée pendant les ~50-200ms de boot
fait `OpenEventW` qui échoue silencieusement. La 1ère instance n'est pas
réveillée.

Pattern correct : `main.rs` fait `acquire_or_signal_existing()` qui crée
le mutex ET l'event, puis passe le HANDLE à `App::new` qui spawn le
waiter thread.

Sur `CreateMutexW` failure (rare), faire **exit(1)**, pas de fake
`First` qui contournerait silencieusement la protection.

### LL hooks et focus de notre propre process

Les LL keyboard hooks installés sur un thread secondaire (notre `runner`)
ne tirent **pas** quand notre propre fenêtre est foreground. Reproduit
avec compteurs : `KB_FIRES=0` quand l'app est focused, `KB_FIRES=1` quand
une autre fenêtre est focused.

Conséquence : on ne peut pas utiliser le LL hook pour la **capture de
bindings** (modal "press a key") quand notre fenêtre a le focus.
Workarounds essayés et abandonnés :
- Déplacer le hook sur le thread main → ne fixe pas + cause un freeze.
- Defocus la fenêtre via `SetForegroundWindow(GetShellWindow())` pendant
  la capture → marche mais introduit un focus flicker visible.

**Décision finale** : la capture passe par les events egui

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Loulouw/ROrganizer](https://github.com/Loulouw/ROrganizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
