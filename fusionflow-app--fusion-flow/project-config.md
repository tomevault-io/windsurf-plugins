---
trigger: always_on
description: The goal of this system is to create a visual flow orchestrator that leverages the full power of the BEAM (Erlang VM) to build a resilient, scalable, and real-time automation platform.
---

# Agents: Visual Flow Orchestrator

The goal of this system is to create a visual flow orchestrator that leverages the full power of the BEAM (Erlang VM) to build a resilient, scalable, and real-time automation platform.

## BEAM Capabilities
We extract the best features of the BEAM to ensure robustness:
- **Concurrency**: utilizing lightweight processes to handle massive numbers of simultaneous workflows.
- **Fault Tolerance**: relying on supervision trees (Supervisors) to isolate failures and self-heal, ensuring that one failing node or flow doesn't bring down the system.
- **Soft Real-time**: guaranteeing predictable latency and responsiveness, essential for orchestration.
- **Distribution**: enabling the system to scale horizontally across multiple nodes effortlessly.

## Backend: Oban & Workflow Orchestration
For the execution engine, we utilize **Oban**, a proven and robust job processing library. Oban serves as the reliable backend orchestrator, managing job queues, retries, scheduled jobs, and workflow execution states, providing a solid foundation for complex automation sequences.

## Frontend: Rete.js, LiveView & Lit
The visual editor is built with a modern stack designed for interactivity and customizability:
- **Rete.js**: A modular framework for visual programming, used to render the node editor.
- **Phoenix LiveView**: Powers the real-time communication and state management, keeping the server and client in sync.
- **Lit**: Used to build custom, lightweight Web Components for the nodes, allowing for a highly customizable and performant UI within the editor.
- **Theming**: The application features fully supported **Light and Dark modes**. All components and modifications must be implemented with both themes in mind to ensure a consistent and high-quality user experience.

---

This is a web application written using the Phoenix web framework.

## Project guidelines

- Use `mix precommit` alias when you are done with all changes and fix any pending issues
- Use the already included and available `:req` (`Req`) library for HTTP requests, **avoid** `:httpoison`, `:tesla`, and `:httpc`. Req is included by default and is the preferred HTTP client for Phoenix apps

### Phoenix v1.8 guidelines

- **Always** begin your LiveView templates with `<Layouts.app flash={@flash} ...>` which wraps all inner content
- The `MyAppWeb.Layouts` module is aliased in the `my_app_web.ex` file, so you can use it without needing to alias it again
- Anytime you run into errors with no `current_scope` assign:
  - You failed to follow the Authenticated Routes guidelines, or you failed to pass `current_scope` to `<Layouts.app>`
  - **Always** fix the `current_scope` error by moving your routes to the proper `live_session` and ensure you pass `current_scope` as needed
- Phoenix v1.8 moved the `<.flash_group>` component to the `Layouts` module. You are **forbidden** from calling `<.flash_group>` outside of the `layouts.ex` module
- Out of the box, `core_components.ex` imports an `<.icon name="hero-x-mark" class="w-5 h-5"/>` component for for hero icons. **Always** use the `<.icon>` component for icons, **never** use `Heroicons` modules or similar
- **Always** use the imported `<.input>` component for form inputs from `core_components.ex` when available. `<.input>` is imported and using it will save steps and prevent errors
- If you override the default input classes (`<.input class="myclass px-2 py-1 rounded-lg">)`) class with your own values, no default classes are inherited, so your
custom classes must fully style the input

### JS and CSS guidelines

#### Color Specifications
The application uses a specific color palette defined in `app.css`. You **MUST** use the following CSS variables or their corresponding Tailwind utility classes to ensure consistency across Light and Dark modes.

-   **Primary Color**:
    -   Variable: `--color-primary`
    -   Tailwind: `primary`, `text-primary`, `bg-primary`
    -   Usage: Main actions, active states, highlights.
    -   *Note*: In Dark mode, this is a Purple/Indigo shade (`oklch(58% 0.233 277.117)`). In Light mode, it is Indigo 600 (`oklch(45% 0.24 277)`).

-   **Dark Mode Implementation**:
    -   Dark mode is driven by the `data-theme="dark"` attribute, **not** `prefers-color-scheme`. The `dark:` Tailwind variant is remapped via `@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *))` in `app.css`.
    -   The slate scale is overridden in `app.css` to near-black values (e.g. `slate-950` = `oklch(3% 0 0)`, `slate-900` = `oklch(5% 0 0)`).

-   **Backgrounds**:
    -   Variable: `--color-base-100` (Main bg), `--color-base-200` (Secondary/Card bg), `--color-base-300` (Borders/Dividers)
    -   Usage: Page backgrounds, card backgrounds, node backgrounds.
    -   *Rule*: **Never** hardcode hex values like `#ffffff` or `#000000`. Use `bg-gray-50` for light mode and `bg-slate-950` for dark mode on the main page. Use `bg-white dark:bg-slate-900` for sidebar/cards. Layering should use `slate-800` for elevated components in dark mode.

-   **Text**:
    -   Variable: `--color-base-content`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FusionFlow-app/fusion_flow](https://github.com/FusionFlow-app/fusion_flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
