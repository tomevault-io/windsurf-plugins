---
trigger: always_on
description: PencilPlaybook is the UI Skills / Taste-Skill for Pencil.dev + Claude Code — a design playbook that gives Claude real perceptual psychology and senior-level guardrails so it stops producing averaged-out AI slop.
---


# Pencil Pro

## Primary Resolution Rule

**This skill must be configured before first use.**

**If the user says "run the PencilPlaybook setup wizard" or asks to set up or configure PencilPlaybook:**
→ Read `.claude/skills/pencilplaybook/setup.md` and follow the instructions there exactly.

**If the user says "build my first screen with PencilPlaybook" or asks to start designing:**
→ Read `.claude/skills/pencilplaybook/onboarding.md` and follow the instructions there exactly.

> Configured resolution: *(run setup to set this)*

If a screen root in an existing `.pen` file doesn't match your configured resolution, note it before editing. Do not build on top of a wrongly-sized canvas without flagging the deviation.

**Common exception — marketing pages:** Design at 1440px regardless of your app resolution. Marketing pages are consumed at variable browser widths — design at 1440px, verify at 390px.

---

## Session Startup (Every Pencil Session)

```
1. get_editor_state()                           — check active document and selection
2. open_document("path/to/your-file.pen")       — open target file
3. get_guidelines(topic="web-app")              — load design guidelines
4. set_variables(<token map below>)             — inject brand tokens
5. get_variables()                              — confirm tokens stored
6. snapshot_layout()                            — verify canvas dimensions
```

**Your token map** — run `set_variables` with this object at the start of every session. Variables may not persist between Claude Code sessions.

```json
{
  "color-primary":        "YOUR_HEX",
  "color-primary-dark":   "YOUR_HEX",
  "color-accent":         "YOUR_HEX",
  "color-background":     "YOUR_HEX",
  "color-surface":        "YOUR_HEX",
  "color-border":         "YOUR_HEX",
  "color-text":           "YOUR_HEX",
  "color-text-muted":     "YOUR_HEX",
  "color-success":        "YOUR_HEX",
  "color-warning":        "YOUR_HEX",
  "color-error":          "YOUR_HEX",
  "font-sans":            "YOUR_FONT",
  "font-mono":            "YOUR_FONT",
  "font-serif":           "YOUR_FONT",
  "spacing-card":         "24",
  "spacing-gap-sm":       "8",
  "spacing-gap-md":       "16",
  "spacing-gap-lg":       "24",
  "spacing-gap-xl":       "32",
  "content-max-width":    "YOUR_VALUE",
  "sidebar-width":        "YOUR_VALUE"
}
```

After `set_variables`, call `get_variables()` to confirm all tokens are stored.

---

## Scaffold Scripts

Nine scaffold scripts covering the most common layout archetypes. Run the setup wizard to fill in the dimension and color values below before using.

---

### Scaffold A — App Dashboard
*Metrics, KPIs, activity feed, monitoring*

```
sidebar=I("root", {name:"Sidebar", width:SCAFFOLD_SIDEBAR_WIDTH, height:SCAFFOLD_CANVAS_HEIGHT, bg:"SIDEBAR_BG", display:"flex", flexDirection:"column"})
pageArea=I("root", {name:"PageArea", width:SCAFFOLD_PAGE_AREA_WIDTH, height:SCAFFOLD_CANVAS_HEIGHT, x:SCAFFOLD_SIDEBAR_WIDTH, bg:"PAGE_BG"})
pageHeader=I("pageArea", {name:"PageHeader", width:SCAFFOLD_PAGE_AREA_WIDTH, height:64, bg:"SURFACE_COLOR", borderBottom:"1px solid BORDER_COLOR", display:"flex", alignItems:"center", px:32})
statsBar=I("pageArea", {name:"StatsBar", y:64, width:SCAFFOLD_PAGE_AREA_WIDTH, height:80, bg:"STATS_BG", display:"flex", alignItems:"center", gap:24, px:32})
contentWrap=I("pageArea", {name:"ContentWrap", y:144, width:SCAFFOLD_PAGE_AREA_WIDTH, height:SCAFFOLD_CONTENT_HEIGHT_A, display:"flex", justifyContent:"center", p:32})
contentInner=I("contentWrap", {name:"ContentInner", width:SCAFFOLD_CONTENT_MAX, display:"flex", flexDirection:"column", gap:24})
```

After scaffold: `snapshot_layout()` — verify pageArea.width and contentInner.width are correct.

---

### Scaffold B — App List / Queue
*Tables, asset libraries, queues, user management*

```
sidebar=I("root", {name:"Sidebar", width:SCAFFOLD_SIDEBAR_WIDTH, height:SCAFFOLD_CANVAS_HEIGHT, bg:"SIDEBAR_BG"})
pageArea=I("root", {name:"PageArea", width:SCAFFOLD_PAGE_AREA_WIDTH, height:SCAFFOLD_CANVAS_HEIGHT, x:SCAFFOLD_SIDEBAR_WIDTH, bg:"PAGE_BG"})
pageHeader=I("pageArea", {name:"PageHeader", width:SCAFFOLD_PAGE_AREA_WIDTH, height:64, bg:"SURFACE_COLOR", borderBottom:"1px solid BORDER_COLOR", display:"flex", alignItems:"center", px:32})
commandStrip=I("pageArea", {name:"CommandStrip", y:64, width:SCAFFOLD_PAGE_AREA_WIDTH, height:52, bg:"SURFACE_COLOR", borderBottom:"1px solid BORDER_COLOR", display:"flex", alignItems:"center", gap:16, px:32})
tableWrap=I("pageArea", {name:"TableWrap", y:116, width:SCAFFOLD_PAGE_AREA_WIDTH, height:SCAFFOLD_CONTENT_HEIGHT_B, display:"flex", justifyContent:"center", px:32, pt:24})
tableInner=I("tableWrap", {name:"TableInner", width:SCAFFOLD_CONTENT_MAX, display:"flex", flexDirection:"column", gap:0})
tableHeader=I("tableInner", {name:"TableHeader", width:SCAFFOLD_CONTENT_MAX, height:40, bg:"TABLE_HEADER_BG", borderBottom:"1px solid BORDER_COLOR", display:"flex", alignItems:"center"})
```

---

### Scaffold C — App Detail / Review
*Two-panel detail views, review flows, side-by-side layouts*

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stevembarclay/pencilplaybook](https://github.com/stevembarclay/pencilplaybook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
