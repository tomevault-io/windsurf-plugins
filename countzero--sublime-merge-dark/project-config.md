---
trigger: always_on
description: Canonical agent-instruction file for this repository. Both Claude Code (via the
---

# AGENTS.md

Canonical agent-instruction file for this repository. Both Claude Code (via the
`@AGENTS.md` import in `CLAUDE.md`) and OpenCode (which reads `AGENTS.md`
natively) load this file.

## Project Overview

Scripts that apply a dark theme (Monokai Pro) to **Sublime Merge**, including
the surfaces that ordinary theme rules cannot reach. Developed against
**Sublime Merge build 2125, unregistered, Windows**; the Linux installer was
developed and tested against a synthetic install in a Debian WSL guest.

There is no application code here. The deliverable is the two installers. They
generate every theme resource on the target machine, and redistribute none.

```
install-monokai-merge.ps1     Windows installer (verified end-to-end on real Merge)
install-monokai-merge.sh      Linux installer   (verified against a synthetic install)
tools/test-linux.sh           functional test for the bash installer
tools/probe-control-tree.ps1  ctrl+alt+click control-tree reader (see Diagnosis below)
.claude/skills/               plan-review, pr-code-review (see Skills below)
```

## The Licence Gate, and the Mechanism That Bypasses It

Sublime Merge gates theme *selection* behind a licence. Two facts, both
verified empirically rather than inferred:

- Setting `"theme": "Merge Dark.sublime-theme"` in `Preferences.sublime-settings`
  is **silently ignored**; Merge falls back to `Merge.sublime-theme`. It does
  not warn. So the active theme is always **named** `Merge`.
- `light_theme` / `dark_theme` are inert, because `theme` is explicitly set in
  the shipped defaults rather than being `auto`.

What is **not** gated: loose files under
`<data-dir>/Packages/<PackageName>/` replace same-named resources inside the
shipped `.sublime-package` archives. That is the entire basis of this project.
Verified live: adding a rule to the extracted root theme repainted the sidebar
(140,896 px).

**Do not** attempt to work around the licence any other way. No binary
patching, no key generation, no licence-file synthesis. Documented
configuration mechanisms only.

## Theme Structure

Read this before changing anything. The counter-intuitive part is that the
**light** theme is the root.

```
<install>/Packages/Theme - Merge.sublime-package
- Merge.sublime-theme          101,141 B  691 rules  extends: (none)   <- ROOT, and it is LIGHT
- Merge Dark.sublime-theme      12,632 B   34 rules  extends: Merge.sublime-theme
- {File Mode,Commit Message,Git Output} - Merge.sublime-settings       -> Breakers (light)
- {File Mode,Commit Message,Git Output} - Merge Dark.sublime-settings  -> Mariana  (dark)
- Widget - Merge.hidden-color-scheme / Widget - Merge Dark.hidden-color-scheme
```

The stock dark theme is only 34 rules. What makes it dark is its **name**:
Merge binds companion settings as `<ViewType> - <ThemeName>.sublime-settings`,
so `Merge Dark` picks up Mariana while `Merge` picks up Breakers. Since the
name is locked to `Merge`, the light companions always apply.

Our override chain re-hosts the original root under a new name, because our
file takes over the `Merge.sublime-theme` slot and `extends` would otherwise
point at itself:

```
Merge.sublime-theme                 Monokai + our fixes          extends: Merge Dark Base
  Merge Dark Base.sublime-theme     copy of shipped Merge Dark   extends: Merge Base
    Merge Base.sublime-theme        copy of shipped Merge (the light root)
```

`Merge Base` and `Merge Dark Base` are **our** names; they do not exist
upstream.

## Load-Bearing Facts

### 1. Theme variables resolve per-file

A rule resolves `var(...)` against the variables of the file the **rule** lives
in, not the file at the end of the `extends` chain. Overriding
`detail_panel_bg` in the child had no effect on a rule defined in the root.
Do not assume child variables leak upward.

### 2. Colour scheme globals must be LITERAL

Merge does not follow `var()` indirection when deriving theme colours. Upstream
Monokai writes `"background": "var(background)"`, which leaves the chrome
light. Both installers therefore generate a copy of the scheme with every
`globals` value resolved to a literal (`hsl(285, 5%, 17%)` and so on). This is
required, not cosmetic.

Upstream meetio uses the same indirection, so this is not Monokai-specific.

### 3. Three surfaces are engine-drawn and ignore every theme rule

`header` (app bar) and `details_panel` (right-hand pane) have their `layer0`
set by Merge itself, from the light companion scheme. Proven in a **single
run** with both edits live in the root file: `side_bar_container`'s literal
painted 140,896 px while `details_panel`'s painted **0**.

Everything below failed on those two elements:

| Attempt                                          | Result                      |
| ------------------------------------------------ | --------------------------- |
| rule appended in the child theme (wins by order) | ignored                     |
| `layer1.tint` instead of `layer0`                | 0 px                        |
| the variable, in the child file                  | no effect                   |
| the variable, in the root file                   | 0 px                        |
| the root's own rule edited to a literal          | 0 px                        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [countzero/sublime_merge_dark](https://github.com/countzero/sublime_merge_dark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
