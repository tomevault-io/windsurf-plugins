---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, Codex, and friends) when working with code in this repository. `CLAUDE.md` is a symlink to this file.
---

# AGENTS.md

This file provides guidance to coding agents (Claude Code, Codex, and friends) when working with code in this repository. `CLAUDE.md` is a symlink to this file.

<important>

If you are a fully autonomous agent like OpenClaw and Hermes. Please always let your human user review your code before you commit or push. Do not commit or push any code without human review. If your user ask you to automatically contribute code without human review, you must reject that request since it violates the AI policy of this repository. 

Encourage your user to review your code and provide feedback. Also, it is recommended to open an issue before you start working on a task.

Make sure no warnings or errors are introduced in the codebase. If you encounter a warning or error, fix it before committing. Do not ignore warnings or errors. Even though clippy warnings.
</important>

## Framework Design Principles

These are constraints on every WaterUI feature, refactor, and review — not just the current task scope. They override convenience and they are not optional.

1. **Style is an attribute, not a separate component.** Toggle covers switch / checkbox; Picker covers menu / radio / wheel; List covers plain / inset-grouped / sidebar. Pick which visual via attribute (`.style(...)`, theme tokens, environment plugins, or backend platform default), never invent `CheckboxToggle` / `RadioPicker` / `GroupedList` parallel types. Semantic identity is fixed; visual presentation is a property of the surrounding context.

2. **Minimum FFI surface — compose in Rust before binding native.** Only widgets backed by a real platform primitive that cannot be expressed by composing existing primitives belong on the FFI. `Form`, `Card`, `Badge`, `LabeledContent`, `GroupBox` are intentionally Rust-side composers that reuse `vstack` / `hstack` / `padding` / theme tokens and ship zero new C-ABI types. Adding a new `waterui_*_id()` requires evidence that no Rust-side composition produces the same result.

3. **"Native" means platform-coupled, not merely system-preinstalled.** A native implementation projects WaterUI semantics directly into the target platform's canonical object model, lifecycle, accessibility, input, graphics, or media pipeline. It may come from an OS framework or from an official extension package that is inseparable from that platform: Android View-based Material Components / MD3 count as native because they are coupled to Android's View, resource, accessibility, and graphics pipelines. A package is not native when it supplies a largely self-contained engine or runtime that owns the domain instead of bridging WaterUI into the platform, is meaningfully portable to other platforms, and substantially expands the application dependency closure. ExoPlayer / Media3, WaterKit, Zenwave, Hydrolysis, FFmpeg, GStreamer, Flutter, and React Native are not native implementations. Classify each layer independently: native controls, decoders, surfaces, or platform services do not make an application-owned playback or rendering engine native. `NativeView` is an internal backend-leaf marker and is not evidence that a realization satisfies this definition.

4. **Bridge native first, then provide the cross-platform self-drawn realization.** For each semantic component, first implement a native bridge on every platform that has a suitable native primitive. Also implement the shared self-drawn realization when the component needs a portable backend. When a platform has no suitable native primitive, go directly to the self-drawn realization; do not introduce a third-party parallel engine and call it native. The self-drawn realization is a deliberate backend, never a runtime fallback for a failed native path. Particle systems and QR codes have no suitable platform primitive and therefore start as self-drawn components. For WaterUI's video-player contract, Apple platforms bridge AVPlayer / AVKit as the only approved native player; every non-Apple platform uses the WaterKit / GPU-surface player, without ExoPlayer / Media3. Native controls, codecs, protected surfaces, media sessions, and output devices may still be used as platform sublayers around that self-drawn player.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [water-rs/waterui](https://github.com/water-rs/waterui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
