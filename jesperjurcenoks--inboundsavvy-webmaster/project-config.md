---
trigger: always_on
description: AI designer for InboundSavvy websites. Use this skill whenever the user wants to edit, create, or redesign website content through their InboundSavvy CMS — including editing pages, adding sections, updating the hero, changing fonts or colors, managing the design system, uploading images, or creating new pages. Also use it when the user says things like "update my site", "change my homepage", "add a testimonials section", "make it look more modern", or any request involving their website content,
---


# Skill: inboundsavvy-webmaster

**Trigger:** `/inboundsavvy-webmaster`
**Description:** AI designer for InboundSavvy websites — explore, create, and improve pages using schema-valid JSON via the InboundSavvy MCP server.

---

## Philosophy

InboundSavvy is an **opinionated site builder**. The content schema *is* the design system — every page is composed from a fixed library of layouts, components, and block-level elements. You cannot invent components, write arbitrary HTML, or bypass the schema. Creative work happens **within** these constraints, not around them. When something feels impossible, the answer is almost always "use a different component from the loaded schema," not "extend the schema."

Pages are responsive by default and must remain so. Layout is expressed almost entirely through **declarative `options` keys** — `size`, `padding`, `margin`, `display`, `flex`, `grid`, `gridColumns`, `imagesPerRow`, `maxColumns`, `position`, `transform`, `background`, etc. These cover the overwhelming majority of cases. The raw `options.css` field is available for rare situations where a CSS property isn't exposed as a named key, but it should not be the default reach — prefer the declarative keys whenever they can express what you want.

The breakpoint model is **inheritance, not duplication**:

- `options: { ... }` (base) is the desktop default — every property cascades to every viewport unless overridden.
- `options.tablet: { ... }` inherits from base and overrides only the fields it redeclares.
- `options.phone: { ... }` inherits from base and overrides only the fields it redeclares.
- `options.computer: { ... }` exists for the rare desktop-only override that shouldn't cascade.

So you write your desktop layout once at the base, then add a `phone` (and sometimes `tablet`) block that surgically overrides whatever would break on a narrow viewport — typically `flex.flexDirection: "column"`, reduced `padding`, smaller `gridColumns` / `imagesPerRow` / `maxColumns`, removed fixed `size.width`. You do **not** repeat unchanged fields.

One caveat: anything written in `options.css` **cannot be overridden by `phone` / `tablet` breakpoints** — the breakpoint system only operates on the declarative keys. That's another reason to prefer the named keys when both are available.

If a change can't survive a 375px-wide viewport, it doesn't ship.

---

## 1. Setup

1. **Get an MCP token** — CMS → **... More → MCP Tokens** → create token (starts with `is_mcp_`).
2. **Configure MCP server** — run `./install.sh` (writes `.mcp.json` + adds to `.gitignore`), or add manually:
   ```json
   {
     "mcpServers": {
       "inboundsavvy": {
         "type": "http",
         "url": "https://beta.inboundsavvy.com/mcp",
         "headers": { "Authorization": "Bearer is_mcp_your_token_here" }
       }
     }
   }
   ```
3. **Install this skill** — `./install.sh` places it at `~/.claude/skills/inboundsavvy-webmaster/SKILL.md`.

**Never commit `.mcp.json`** — it contains your token. Add `is_mcp_*` to `.gitignore`.

---

## 2. Session Start Protocol

**Run automatically when `/inboundsavvy-webmaster` is invoked. No user questions needed.**

If the user typed `/inboundsavvy-webmaster refresh-schema`, delete `~/.claude/skills/inboundsavvy-webmaster/schema-cache.md` before continuing, then run steps 1–5 normally.

Execute in this order:

1. **Load schema rules** — version-check the local cache; fetch only when stale.

   Cache path: `~/.claude/skills/inboundsavvy-webmaster/schema-cache.md`

   a. Call `get_schema_version()` → `current_version`
   b. If cache exists: run `grep -m1 "schema-version" ~/.claude/skills/inboundsavvy-webmaster/schema-cache.md` → extract `cached_version`
   c. If no cache **or** `cached_version ≠ current_version`: call `get_content_schema_reference()` → write full response to cache path
   d. Find Section 7 start line: `grep -n "^## 7\." ~/.claude/skills/inboundsavvy-webmaster/schema-cache.md` → `line_N`
   e. `Read(cache_path, offset=line_N, limit=250)` → hold Section 7 rules in active context. Do **not** load the full document.

2. **Load site identity** — call `get_content_file("globalsitesettings", "globalsitesettings")`. Extract:
   - `siteName` — the site's display name
   - `seo.url` — the canonical domain (e.g. `https://acmestudio.com`)
   - Derive beta preview base URL: `https://beta.{domain}/` (strip `https://` from `seo.url`)

3. **Load design system** — call `get_content_file("globaldesignsettings", "globaldesignsettings")`. Extract:
   - `colorDesign` — the hex values the user has set for each CSS variable in their design system; use these when updating `globaldesignsettings`. In page/content JSON always reference them as `var(--...)` instead.
   - `fontSelection` — heading and body fonts (use when describing the current design system to the user)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JesperJurcenoks/Inboundsavvy-webmaster](https://github.com/JesperJurcenoks/Inboundsavvy-webmaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
