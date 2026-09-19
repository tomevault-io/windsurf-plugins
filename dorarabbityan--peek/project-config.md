---
trigger: always_on
description: Run the local server yourself and open the preview in the browser available to this environment. Do not give the user server-start instructions when you can run it.
---

# Prototype Instructions

Run the local server yourself and open the preview in the browser available to this environment. Do not give the user server-start instructions when you can run it.

Before making substantial visual changes, use the Product Design plugin's `get-context` skill when the visual source is unclear or no longer matches the current goal. When the user gives durable prototype-specific design feedback, preferences, or decisions, record them in `AGENTS.md`.

When implementing from a selected generated mock, treat that image as the source of truth for layout, component anatomy, density, spacing, color, typography, visible content, and hierarchy.

Build app UI in `src/`. Keep `.openai/hosting.json`, `worker/index.js`, `scripts/prepare-sites-build.mjs`, and `tests/sites-worker.test.mjs` intact so the same local prototype can be handed to Sites. Before a Sites handoff, run `npm run build` and `npm run test:sites`; the build must leave `dist/client/index.html`, `dist/server/index.js`, and `dist/.openai/hosting.json`.

## Product decisions

- Demo avatar decision (2026-09-17): use the user-provided illustrated portrait at `public/assets/peek-composer-avatar.png` for the reply composer in `src/App.jsx`, replacing the previous sample portrait. Keep its existing circular styling. The extension's composer continues to use the signed-in X account avatar.
- Branding decision (2026-09-16): use the product name `Peek`, following the earlier naming recommendation and the user's request to rename the plugin. The user selected the magenta rabbit/window design, then requested removal of the surrounding magenta background. The current avatar is `docs/branding/peek-avatar-v5-transparent.png`: retain the rabbit, dark window frame and enclosed magenta fill, with true transparent alpha outside the combined silhouette. This image is applied to `public/assets/tuzai-icon-source.png`, shared by the demo and extension build. Preserve earlier image versions as references. Keep internal tuzai/Tuzai identifiers, repository names and example post authors unchanged.
- This repository ships a Manifest V3 Chrome extension for `x.com` and `twitter.com` plus a Vite-based visual demo.
- The core interaction keeps the X timeline in place and opens a modal reader: original post on the left, replies on the right.
- On a `/status/...` detail page, do not intercept ordinary tweet or reply clicks because the native detail experience remains authoritative. A nested quoted-post card is the only exception: clicking its card opens that quoted post in the reader. In timeline and notification lists, a clicked quoted-post card must likewise win over its containing tweet instead of opening the outer tweet. Current X quote cards are often `div[role="link"]` nodes with no `href` or descendant status link, so detect the actually clicked semantic quote container first, then resolve its target from the containing tweet's GraphQL `quoted_status_result`; never depend on a quote-card `/status/` anchor being present.
- When a list or notification click targets a reply, the left pane must render its available ancestor chain from the conversation root to the immediate parent, followed by the clicked reply as the focal post. Preserve X-like vertical thread connectors and keep descendants in the right replies pane.
- When that reply thread first opens, scroll the left pane to the clicked focal reply while keeping its ancestors above and reachable by upward scrolling. This initial focus runs once only; subsequent interactions and rerenders must preserve the user's pane position.
- A standalone focal post, including one opened by resolving a nested quote, must start at the top of the left pane. Disable browser scroll anchoring for the no-context post pane and reset its initial scroll once so automatic translation or media hydration cannot push the first visible content down; context threads still use their focal-reply positioning behavior.
- The original-post action bar must preserve X's native five-item order and density: reply, repost, like, bookmark, share. Do not insert analytics into that row.
- The reader uses X's currently loaded internal GraphQL operation definitions and the current signed-in browser session. A small MAIN-world bridge captures only the request metadata required to call X; credentials and CSRF values must never be persisted, logged, or sent outside X.
- The left pane renders the focal post; the right pane renders reply sorting, a pure-text reply composer, nested replies, and cursor-based loading. Each pane scrolls independently.
- On desktop, keep the original-post pane slightly wider than the replies pane at approximately `52% : 48%`; the stacked mobile layout remains unchanged.
- The reader uses one compact global header with the rabbit icon, the exact title `Peek`, open-in-X, and close controls. Do not add pane titles, explanatory subtitles, independent-scroll pills, or a bottom status/footer bar. In the reply tools row, keep sort and plain-text reply count together on the left and `查看引用` aligned to the far right.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DoraRabbitYan/peek](https://github.com/DoraRabbitYan/peek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
