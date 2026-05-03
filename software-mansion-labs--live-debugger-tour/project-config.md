---
trigger: always_on
description: This is the **LiveDebugger Tour** — a Phoenix v1.8 LiveView application that serves as an interactive, step-by-step tutorial for the [LiveDebugger](https://github.com/software-mansion-labs/live-debugger) tool. It is meant to be used **locally in dev mode** with LiveDebugger as a dependency. Each tour step is a self-contained LiveView that introduces a LiveDebugger feature with a minimal, hands-on example the user can inspect and debug in real time.
---

This is the **LiveDebugger Tour** — a Phoenix v1.8 LiveView application that serves as an interactive, step-by-step tutorial for the [LiveDebugger](https://github.com/software-mansion-labs/live-debugger) tool. It is meant to be used **locally in dev mode** with LiveDebugger as a dependency. Each tour step is a self-contained LiveView that introduces a LiveDebugger feature with a minimal, hands-on example the user can inspect and debug in real time.

## Project overview

### Purpose

The goal is to make LiveDebugger approachable by offering a curated set of easily accessible, interactive feature demonstrations. Examples are kept as simple as possible to teach concepts before advancing to more advanced LiveDebugger features.

### LiveDebugger dependency

LiveDebugger is included from **github branch** (`{:live_debugger, github: "software-mansion/live-debugger", branch: "live-debugger-tour", only: [:dev, :test]}`).
It is on a special branch that exposes a tour API (see "Tour API" section below). The dependency is dev-only — this app is not intended for production use.

### Tour topics

Each topic maps to a numbered tour step LiveView:

1. [x] **Start debugging** — Explore the Node Info panel to identify the process PID, module path, and learn how to jump from the debugger to the code editor.
2. [x] **Inspecting assigns** — Monitor LiveView state changes in real-time. Use the interactive demo to see how values update, search complex data, pin variables, and track mutation history.
3. [x] **Callback traces** — Start/stop traces to analyze LiveView lifecycle execution times, filter specific events, and manage memory by removing old traces.
4. [x] **Dead LiveView & Exceptions** — Trigger a deliberate crash with a "Boom" button to see how the debugger displays the final state of a dead process and identifies its successor.
5. [x] **Components tree** — Visualize complex UI hierarchies with multiple LiveComponents, using the highlight feature to map the tree structure to the browser view.
6. [x] **Async jobs** — Observe `assign_async` behavior to see how background tasks transition from loading states to successful data injections in real-time.
7. [x] **Streams** — Examine how Phoenix Streams are handled by the debugger, showing how large collections are efficiently managed and updated.
8. [ ] **Global callback traces** — Analyze cross-node communication by filtering messages sent from a child LiveComponent to its parent LiveView.
9. [ ] **Analyzing diffs** — Dive into the actual data payloads (diffs) sent over the wire to the browser to understand how Phoenix optimizes UI updates.
10. [ ] **Resources** — Monitor real-time performance graphs and watch how specific user interactions cause spikes or changes in system resource usage.
11. [x] **Active LiveViews** — Use the dynamic dashboard to see all currently running LiveView processes across the application as they connect and disconnect.
12. [x] **Settings** — Overview of the customization options available to tweak the debugger UI and connection parameters.

## Project guidelines

- Use `mix precommit` alias when you are done with all changes and fix any pending issues
- Use DaisyUI classes for UI components (cards, buttons, badges, alerts, etc.) — the project already includes DaisyUI with custom themes
- This is a **dev-only** tutorial app — do not add production concerns (authentication, deployment, etc.)

### Tour page architecture

#### Creating a new tour page

Each page is a LiveView module that uses both `LiveDebuggerTourWeb, :live_view` and `LiveDebuggerTour.Page`:

```elixir
defmodule LiveDebuggerTourWeb.Live.MyPageLive do
  use LiveDebuggerTourWeb, :live_view
  use LiveDebuggerTour.Page,
    number: 1,
    title: "Start Debugging",
    description: "Explore the Node Info panel..."

  @tour_steps [...]

  @impl true
  def mount(_params, _session, socket) do
    {:ok, tour_page_assigns(socket, @tour_steps)}
  end

  @impl true
  def render(assigns) do
    ~H"""
    <Layouts.app flash={@flash}>
      <%!-- Page content here --%>
    </Layouts.app>
    """
  end
end
```

Required `Page` metadata keys: `number`, `title`, `description`. The URL path is auto-generated from the title (e.g. "Start Debugging" becomes `/pages/start-debugging`).

Pages are **auto-discovered** at runtime by `LiveDebuggerTour.PageDiscovery` — it scans all compiled modules that export `__page_meta__/0` and sorts them by number. The homepage lists all discovered pages automatically.

#### Adding a route

Routes are **auto-generated** by `LiveDebuggerTour.PageDiscovery.routes()` in the router — no manual route entry is needed when creating a new page.

#### Page file naming convention

- Module: `LiveDebuggerTourWeb.Live.<PageName>Live` (e.g. `StartDebuggingLive`)
- File: `lib/live_debugger_tour_web/live/<page_name>_live.ex` (e.g. `start_debugging_live.ex`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [software-mansion-labs/live-debugger-tour](https://github.com/software-mansion-labs/live-debugger-tour) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
