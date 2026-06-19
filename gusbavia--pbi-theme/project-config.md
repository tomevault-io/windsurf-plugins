---
trigger: always_on
description: Generate complete Power BI Desktop theme files from brand colors. Creates production-ready JSON themes with branded canvas backgrounds, CSS design tokens, DAX HTML visual examples, and A4 PDF design system documents. Runs standalone or chained with pbi-lifecycle (auto-saves into the client's Global Theme/pbi-theme/ folder when invoked inside a lifecycle-managed project). Use when user asks to create a Power BI theme, PBI theme, dashboard theme, report branding, or Power BI color palette. Trigger
---


# Power BI Theme Generator

Generate a complete Power BI Desktop theme JSON file (~5800 lines) from the client's brand colors. The output is a production-ready `.json` theme importable via View > Themes > Browse in Power BI Desktop, plus a design system PDF, CSS tokens, and DAX HTML visual examples.

Input: **$ARGUMENTS**

## 0. Mode triage — ALWAYS THE FIRST QUESTION

This skill runs in one of three modes. Ask the user up front, before the colour/layout questionnaire. Do not skip — the mode decides where the output files land.

```
This theme is for:
  1. Standalone — I'll save the output where you tell me (default: current working directory)
  2. pbi-lifecycle project — output goes to <Client>/Global Theme/pbi-theme/
  3. pbi-project-hub engagement — output goes to Clients/{name}/02_Design_System/

Type 1, 2, or 3.
```

If the user's invocation already implies the mode (e.g. they pasted a path that contains `Global Theme/`, or they said "use the lifecycle theme path"), the agent may infer the mode and skip the prompt.

### Mode 1 · Standalone

- Ask the user where to save the files (default: current working directory).
- Use the brand name they provide for file naming.
- Output goes flat into the chosen folder.

### Mode 2 · pbi-lifecycle integration

- Ask: *"Paste any path inside the lifecycle-managed client (the project folder, the Power BI Repository folder, or the client root itself). I'll find `Global Theme/` automatically."*
- Resolve the **client root** by walking up from the path the user provided until you find a sibling folder named `Global Theme` (or `ClientTheme` for older lifecycle versions). The client root is the folder that contains both `Global Theme/` and `Power BI Repository/`.
- If `Global Theme/` is not found within 5 parent levels of the provided path, stop and ask the user to confirm the client root manually. Do not guess.
- Inside `Global Theme/`, the lifecycle skill creates three subfolders: `logo/`, `brand-guideline/`, `pbi-theme/`. The output of THIS skill goes into `Global Theme/pbi-theme/`, organized in four subfolders by file category:

  ```
  Global Theme/pbi-theme/
    theme/                  ← {brand}-pbi-theme.json + {brand}-canvas-background.svg
                              (the importable artifacts — View > Themes > Browse)
    design-system/          ← {brand}-design-system.pdf
                              (shareable A4 documentation)
    design-tokens/          ← {brand}-tokens.css
                              (CSS custom properties for HTML visuals)
    html-visual-style/      ← {brand}-html-visual-example.dax
                              (DAX example using the tokens)
  ```

- If any of the four category subfolders does not exist yet (older lifecycle version or manual client setup), create the missing ones before writing. Never delete or modify anything that already exists.
- Read-only on everything else under the client root. Never touch `Power BI Repository/`, `Global Info/`, `Global Theme/logo/`, `Global Theme/brand-guideline/`, or any pre-existing file in the four category subfolders. The skill only writes net-new files; if a `{brand}-pbi-theme.json` already exists in `theme/`, ask the operator before overwriting (offer to suffix with a version number, e.g. `{brand}-pbi-theme-v2.json`).

### Mode 3 · pbi-project-hub engagement (legacy companion)

- The user will typically already have a client folder at `Clients/{name}/`.
- Generate outputs directly into `Clients/{name}/02_Design_System/`.
- Use the client name from the project-hub context to name the files.
- Cross-link: the project-hub's `brand-guide.md` should reference the theme outputs.
- Detect by checking for a `Clients/{name}/02_Design_System/` path in $ARGUMENTS or nearby working files. If present, use that path.

### Output file list (same in every mode)

Whatever the mode, this skill always writes the same five files (and only these five):

1. `{brand}-pbi-theme.json`
2. `{brand}-canvas-background.svg`
3. `{brand}-design-system.pdf`
4. `{brand}-tokens.css`
5. `{brand}-html-visual-example.dax`

**Where each file lands:**

| Mode | File | Destination |
|---|---|---|
| 1 (standalone) | all 5 | flat in the chosen folder |
| 2 (lifecycle) | `{brand}-pbi-theme.json` | `Global Theme/pbi-theme/theme/` |
| 2 (lifecycle) | `{brand}-canvas-background.svg` | `Global Theme/pbi-theme/theme/` |
| 2 (lifecycle) | `{brand}-design-system.pdf` | `Global Theme/pbi-theme/design-system/` |
| 2 (lifecycle) | `{brand}-tokens.css` | `Global Theme/pbi-theme/design-tokens/` |
| 2 (lifecycle) | `{brand}-html-visual-example.dax` | `Global Theme/pbi-theme/html-visual-style/` |
| 3 (project-hub) | all 5 | flat in `Clients/{name}/02_Design_System/` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gusbavia/pbi-theme](https://github.com/gusbavia/pbi-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
