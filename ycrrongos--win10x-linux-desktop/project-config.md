---
trigger: always_on
description: These instructions apply to the entire repository rooted at
---

# Win10X Linux Desktop — Codex repository instructions

These instructions apply to the entire repository rooted at
`/mnt/data/win10x_linux_desktop`.

## Product direction

- Build a touch-first Linux desktop for phones, tablets, and touchscreen PCs,
  including an aarch64 target.
- The general shell and interaction baseline is the late, single-screen
  Windows 10X design. Use `research/windows10x/` for historical evidence and
  `docs/interaction-checklist.md` for acceptance work.
- The full-screen Windows 10 tablet-style Start screen and Live Tiles are an
  explicit user requirement. They are an intentional hybrid with the Win10X
  shell, not a defect to “correct” into the stock Win10X launcher.
- OOBE, lock screen, and login are deferred. Do not spend implementation or
  review time on them unless the user explicitly brings them back into scope.
- Dual-screen hardware, hinge-aware layouts, spanning, and Wonder Bar are
  deferred until after the single-screen version. Keep their research isolated
  under `research/windows10x/dual-screen-hardware/`.
- The previous acrylic implementation was deliberately removed so the material
  can be redesigned from a clean baseline. Do not restore the old wallpaper
  sampler, CPU blur cache, tint/noise stack, HostBackdrop element, or compositor
  GLES blur shader unless the user explicitly asks to begin the new acrylic
  implementation. The Start background currently crossfades to the wallpaper
  and adds a semi-transparent white veil; this is intentional simple alpha
  composition, not acrylic blur.
- Research screenshots and Microsoft marks are study references only. Do not
  redistribute them as product assets.

When documents disagree, current explicit user decisions above take precedence.
Update stale project documentation when the affected behavior is changed.

## Before changing code

1. Read `README.md` and the relevant sections of
   `docs/interaction-checklist.md`.
2. Read the latest relevant entries in `docs/lessons-learned.md` before
   debugging or repeating an earlier experiment.
3. Inspect `git status` and the affected code. The worktree may contain user or
   other-agent changes; preserve unrelated work.
4. For Win10X behavior questions, check the local research index and sources.
   Distinguish the 2019 concept, 2020 dual-screen emulator, and 2021
   single-screen build instead of treating them as one specification.
5. Do not assume a checked item in the interaction checklist has been visually
   accepted. Reproduce the current behavior when the task concerns UI.

## Codex command execution

- Use Codex's normal command/PTY tools with
  `/mnt/data/win10x_linux_desktop` as the working directory.
- Do not use or require `~/.qoder/skills/detached-shell/`. That path belongs to
  `.qoder/rules/agent.md` and is not available in Codex.
- Give builds and other long-running commands a bounded initial yield. Use the
  returned session for continued output or a persistent GUI instead of using a
  blocking foreground wait.
- Launching the graphical shell against the host Wayland compositor may require
  approved GUI access even when `WAYLAND_DISPLAY` is correct.
- When restarting a visual test, stop only the instance launched by the current
  task, rebuild/run the current worktree, and make sure the user is not looking
  at a stale shell instance.
- `cargo check` and test harness builds do not refresh an existing
  `target/debug/win10x-shell`. Before launching that path directly for visual
  acceptance, run `cargo build -p win10x-shell` after the final code edit.

## Implementation and verification loop

For a code change, use this sequence unless the task clearly needs less:

1. Reproduce or identify the exact code path before editing. For animated UI,
   compare both the in-animation and settled drawing branches.
2. Make a focused patch. Preserve unrelated changes and user-selected design
   choices.
3. Format only affected workspace packages. For the usual UI work:

   ```bash
   cargo fmt -p win10x-shell -p win10x-comp
   ```

   Do not use `cargo fmt --all` until the missing example files declared by the
   vendored `third_party/fluent-egui` manifest are restored.
4. Run checks proportional to the change. The normal baseline is:

   ```bash
   cargo check -p win10x-shell -p win10x-comp
   cargo test -p win10x-protocol
   ```

5. For visible or interactive changes, run the latest shell and inspect the
   real UI. A successful compile is not visual acceptance.
6. Watch runtime logs while exercising the changed path. In particular, treat
   egui `changed id between passes` warnings as real defects: debug builds draw
   a red 2 px diagnostic rectangle for them.
7. Keep the test instance running when the user has asked to inspect it. Report
   exactly what was verified and leave final visual judgment to the user.

Follow `docs/testing.md` for broader levels:

- L1: nested/standalone shell on the current Wayland desktop for daily work.
- L2: KVM/QEMU regression before releases.
- L3: real touch hardware and aarch64 build/smoke test.

If a normal system screenshot cannot capture the shell, the shell's one-shot
self-capture can be used after building:

```bash
WINUI_EGUI_SCREENSHOT=/tmp/win10x-shell.png target/debug/win10x-shell
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ycrrongos/win10x-linux-desktop](https://github.com/ycrrongos/win10x-linux-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
