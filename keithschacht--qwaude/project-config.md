---
trigger: always_on
description: `qwaude` runs Claude Code against a local Qwen 3.8 27B on Apple Silicon. It is a
---

# qwaude — agent context

`qwaude` runs Claude Code against a local Qwen 3.8 27B on Apple Silicon. It is a
single self-contained bash script; the interesting engineering is in the choices
below, most of which were arrived at by measurement, not preference. Read this
before changing anything — several "obvious improvements" were tried and lost.

## Hard constraints (don't break these)

- **One file, stock `/bin/bash` 3.2.** A bootstrapper runs before its
  dependencies exist; a fresh Mac has no Python, Ruby, jq, uv. No associative
  arrays, no `mapfile`, no `${var,,}`, no `[[ -v ]]`; empty arrays under
  `set -u` need `${arr[@]+"${arr[@]}"}`. Test under `/bin/bash`, never
  Homebrew bash. After oMLX is installed, its bundled Python
  (`$(brew --prefix)/opt/omlx/libexec/bin/python`) may be used for JSON/Jinja.
  Perl is also stock on macOS. `timeout` is NOT (use background + poll + kill).
- **Prerequisites stop-and-instruct, never install:** Apple Silicon, Homebrew,
  Claude Code. Missing → print the official install command, exit 1.
- **Flags are exactly** `--speed 4bit|8bit`, `--effort low|medium|xhigh|off`,
  `--uninstall`, `-y`, `-h/--help`. Everything else passes through to
  `claude` (`-r`, `-c`, …).
- **`--uninstall` is provenance-aware.** Setup appends to a manifest
  (`~/.qwaude/installed`) only for components qwaude itself installed;
  uninstall offers those (default Y), skips ones that pre-existed ("already on
  your machine before qwaude"), and with no manifest treats provenance as
  unknown (defaults N for uv/oMLX/LiteLLM, Y for the models). Homebrew and
  Claude Code are never touched. It refuses to run while a qwaude server is
  up. Keep the manifest honest when adding install steps.
- **`--uninstall` has to leave nothing behind** — the README promises "remove
  all traces", so every new artefact needs an offer. Order: oMLX / LiteLLM /
  uv / both model dirs → `~/.omlx/{cache,logs}` → the rest of `~/.omlx` →
  `brew autoremove` → `~/.qwaude` + /tmp state → the script itself. Three
  things that are easy to get wrong here:
  - **Show the real size, not a constant.** `dir_size_h` (`du -sh`) goes in
    the cache/logs/`~/.omlx` prompts. The oMLX cache reaches tens of GB (57 GB
    on the owner's machine) and is invisible otherwise — the number is the
    whole argument for the prompt existing.
  - **Two prompts deliberately default N**: `brew autoremove` (brew-wide; it
    prints the exact formula list from `--dry-run` first, and is skipped
    entirely when that list is empty) and deleting the script. `-y` accepts
    defaults, so `-y` does neither. That is the intended reading of `-y`.
  - **`~/.omlx` wholesale** is offered only when `models/` is empty *and*
    `omlx_settings_only_ours` confirms `model_settings.json` names no model
    but ours (no Python to read it with ⇒ can't tell ⇒ don't offer). The
    script is found via `command -v`, verified by grepping `^QWAUDE_VERSION=`
    in it before any `rm` — never trust the name alone — and a symlink is
    removed as a symlink, never followed into the user's checkout.
- **One hidden knob only:** `QWAUDE_INITIAL_DRY_RUN=1` replays the full first
  run (fake timed downloads, prompts, install steps) writing nothing, and ends
  exactly where a real first run ends — "Installation is complete! Now run
  qwaude", exit 0, no servers, no session. The owner explicitly rejected any
  other env switches and `NO_COLOR`; plain output is automatic when stdout
  isn't a TTY. Don't add knobs.
- **Setup never rolls into a session.** `run_setup` does not return: it prints
  `✓ Setup complete.`, a blank line, then `Installation is complete! Now run
  qwaude` and exits 0. That applies to any run where `setup_needed` was true,
  including one where only the quiet steps (template, settings, proxy config,
  Tavily key) had work to do. Only a launch with nothing to set up reaches
  `start_server`. The self-check that used to run in `main` after setup now
  lives at the end of `run_setup`, before those lines.
- First run downloads **both** quants (4-bit 16.6 GB, 8-bit 29.3 GB) so
  `--speed` switches instantly later. Everything is idempotent, and the
  checklist reflects it: components already present are listed as ✓ under
  "Checking for prerequisites"; "We are about to set up" shows only real work.
- **The weights come down over `curl`, not oMLX's bundled `hf`.** `hf` only
  exists after `brew install omlx` has built 3.2 GB, so on a genuinely fresh
  Mac the download could not begin until minutes after "Proceed? y" and the
  progress bar under the Tavily prompt never appeared — it only ever worked in
  demo mode or on a machine that already had oMLX. Nothing in the download
  path may depend on an installed component again.
- The first-run UI wording (banner, "First run, initializing...", "Checking for
  prerequisites", "We are about to set up", the "Quick start" paragraph, the
  unindented "Download total" / "Proceed? [Y/n]" lines, the Tavily steps and
  paste prompt) is owner-specified verbatim. Don't reword it.

## Architecture

```
Claude Code ──ANTHROPIC_BASE_URL──▶ LiteLLM proxy :4002 ──▶ oMLX server :8092
              (effort router + Tavily          (paged KV, MTP k=3,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keithschacht/qwaude](https://github.com/keithschacht/qwaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
