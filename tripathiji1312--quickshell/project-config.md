---
trigger: always_on
description: - Always consult the official QuickShell documentation at https://quickshell.org/docs/v0.2.1/ (QuickShell v0.2.1) for authoritative guidance on configuration and runtime behavior.
---

# GitHub Copilot — Project-specific instructions for quickshel (v0.2)

## Purpose
# GitHub Copilot — Project-specific instructions for quickshel (v0.2)

## Purpose
- Always consult the official QuickShell documentation at https://quickshell.org/docs/v0.2.1/ (QuickShell v0.2.1) for authoritative guidance on configuration and runtime behavior.
- Treat the attached project at `#file:example` as the single source of truth for design, configuration, assets and style decisions. Do not modify files inside the `#file:example` folder; use them only as inspiration for configuration, syntax, and design patterns.
- Always target quickshel v0.2 and Qt 6.10; do not assume other runtime versions unless explicitly instructed.
- Refer to official Qt 6 documentation (https://doc.qt.io/qt-6/) for correctness of QML/Qt APIs and configuration.

## Primary rules
1. Always consult `#file:example` before proposing code, config changes, or design changes. Prefer reusing components, styles and patterns from that tree.
2. For Qt/QML specifics, consult official Qt 6 documentation only for import syntax, QML types, bindings and module configuration. Use Qt 6.10 semantics and APIs.
3. Respect the repository layout and conventions:
    - QML components: `components/`, `modules/`, `services/`
    - Configuration files: `config/`

    ## Purpose
    # GitHub Copilot — Project-specific instructions for quickshel (v0.2)

    ## Purpose
    - Always consult the official QuickShell documentation at https://quickshell.org/docs/v0.2.1/ (QuickShell v0.2.1) for authoritative guidance on configuration and runtime behavior.
    - Treat the attached project at `#file:example` as the single source of truth for design, configuration, assets and style decisions. Do not modify files inside the `#file:example` folder; use them only as inspiration for configuration, syntax, and design patterns.
    - Always target quickshel v0.2 and Qt 6.10; do not assume other runtime versions unless explicitly instructed.
    - Refer to official Qt 6 documentation (https://doc.qt.io/qt-6/) for correctness of QML/Qt APIs and configuration.

    ## Primary rules
    1. Always consult `#file:example` before proposing code, config changes, or design changes. Prefer reusing components, styles and patterns from that tree.
    2. For Qt/QML specifics, consult official Qt 6 documentation only for import syntax, QML types, bindings and module configuration. Use Qt 6.10 semantics and APIs.
    3. Respect the repository layout and conventions:
        - QML components: `components/`, `modules/`, `services/`
        - Configuration files: `config/`
        - Build files: top-level `CMakeLists.txt`, `extras/`, `plugin/`, `nix/` (note: do not introduce or require editing build systems for config-only changes, this #example folde ris for a repositery ignore all cmake and nix related things, becuase thaht system is nix based)
        - Assets, shaders, scripts: `assets/`, `utils/scripts/`
    4. Reuse existing component naming and styles (e.g., `Styled*`, `IconButton`, `MaterialIcon`). Match casing, property names and signal names observed in the tree.
    5. Follow `.clang-format` and project coding style. Keep formatting consistent with existing files.
    6. Prefer QML-first solutions when the project uses QML. Introduce C++ only when strictly required for performance, platform APIs, or when QML cannot express the behavior.
    7. Do not require Nix or CMake workflows for configuration changes. For local verification on the user's environment (Arch Linux + Hyprland), prefer running the QuickShell binary directly using system Qt 6.10 packages.

    ## UI/UX guidance
    - Prioritize fluid, polished animations and motion design consistent with the example tree. When asked to implement visual or interaction changes, implement smooth, performant animations and transitions using existing patterns and assets.
    - Use the project's shaders, easing curves, and components to achieve a responsive look-and-feel; prefer hardware-accelerated effects and avoid blocking the UI thread.
    - Try to implement requested features in a way that respects the existing design language and visual style of the example tree. Avoid introducing jarring or inconsistent UI elements.
    - When adding new UI components, ensure they are responsive and adapt to different screen sizes and orientations, following existing layout patterns.
    - When implementing user interactions (clicks, gestures, keyboard shortcuts), ensure they are intuitive and consistent with existing behavior in the example tree.
    - Use material 3 expressive icons and styles and animations.

    ## Implementation guidance when asked to produce code
    - Provide minimal, complete edits scoped to specific files. Show the exact new file path and full file contents in the response (ready to save).
    - If modifying existing files, provide a concise patch/unified diff or the full updated file content.
    - Prefer small, reviewable changes. When adding features, also add or update a config in `config/` to match project conventions.
    - Use explicit QML import lines that match Qt 6.10, for example:
      ```qml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tripathiji1312/quickshell](https://github.com/tripathiji1312/quickshell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
