---
trigger: always_on
description: - Build a new React dashboard for sing-box and Clash API backends.
---

# Agent Notes

## User goals

- Build a new React dashboard for sing-box and Clash API backends.
- Use zashboard as the reference for API behavior and page ideas.
- Keep the layout responsive and single-column on the connections and proxies pages.
- Add multi-instance management and mobile adaptation.
- Use a light business-style theme with minimal border radius.
- Keep the implementation incremental; more features will come later.
- Ship a self-contained static build and a zip artifact for offline use.

## Important constraints

- Use Node + pnpm.
- Avoid creating unnecessary system clutter.
- Document that the project is largely AI-built in the README.
- Keep future work compatible with zashboard-style API flows.

## Current implementation notes

- Connections use dense clickable rows with search, status tabs, sorting, close actions, and a detail modal.
- Proxy groups stay in one vertical column while their nodes use a compact responsive grid. Groups can collapse, nodes and groups can be tested, and provider health checks are supported.
- The desktop sidebar quick-settings area contains only the fast mode switch. Mihomo/Clash backends expose their `GLOBAL`/`全局` group in Proxies and show only that group while the backend mode is `global`; sing-box backends hide those reserved groups and use per-instance Settings values for the custom global mode and global group, showing only that configured group while the custom mode is active. Mobile keeps the instance picker in the top toolbar.
- Telemetry history is kept per instance in React memory, so instance switching preserves charts while a browser refresh clears them.
- Rule Flow UI state is kept in an in-memory store keyed by instance ID, so route navigation restores positions, selections, filters, zoom, and dynamic color allocation; a full browser refresh clears it. Full-layout positions are kept separately from the temporary compact filtered layout, so leaving a filter restores the complete canvas without reusing filtered row/column coordinates. Initial placement, new-node placement after a data refresh, and explicit layout reset share the same full-layout position generator and the instance's first-render layout baseline; later DOM measurements do not expand the full-view row spacing. A refresh with unchanged nodes reuses the existing position object, while a changed rule/target set creates a new baseline.
- Display names are `Clash API` and `sing-box API`.
- Rules are visualized in Overview as a compact draggable rule-flow board; Logs remains a separate page alongside Connections, Proxies, and Settings. Instance editing is only exposed through the sidebar picker.
- The desktop topbar is a page toolbar. Connections and Proxies mount their search and actions into it; the instance picker stays in the sidebar on desktop and appears in the topbar only on mobile.
- The API client covers the zashboard-style Clash REST surface, connection actions, proxy/provider/rule operations, DNS/cache/Geo/config actions, maintenance/storage/smart/honk helpers, and traffic/memory/connections/logs WebSockets.

## Latest UI behavior

- The default sidebar order is Overview, Proxies, Connections, Logs, Settings. Desktop navigation items are draggable and persist in `localStorage`; mobile navigation follows the same order.
- Logs filters and clearing are mounted in the compact top toolbar. Connections filters, status tabs, sorting, and close-all are also in the top toolbar.
- Connections default to descending download traffic, then Chains and Host as deterministic tie-breakers. The dense table has a stable horizontal scroll width, sortable headers, neutral X close actions, and a centered floating detail card whose body scrolls independently.
- Connection Raw details render the complete JSON payload with lightweight syntax colors for keys, strings, numbers, booleans, and null values.
- Proxy nodes use fixed rectangular two-row cards in a single-column group list. Every node card owns an independent loading key, so one test never disables selection or another node's test. Individual, group, all-group, and provider health checks preserve latency results in page state, animate values from zero only when a test result arrives, show existing values immediately after page navigation, keep the test button available, omit the redundant `ms` suffix, and color the latency number by health.
- Overview renders the three telemetry charts and the compact rule-flow board in the main content area; there is no separate Rules page in navigation. The charts use a fixed latest-sample window, progressively reveal from the newest point at the right, expose readable Y-axis labels, hover crosshairs, and value tooltips; traffic and memory use filled areas. Runtime status, instance/API details, theme, mode, and global-group selection remain in the sidebar quick-settings/telemetry area.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duakc/duakboard](https://github.com/duakc/duakboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
