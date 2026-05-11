---
trigger: always_on
description: Waybar module displaying Claude Code usage statistics. Zero external dependencies, single binary output.
---

# waybar-claude-code

Waybar module displaying Claude Code usage statistics. Zero external dependencies, single binary output.

## Architecture

**Single execution model**: Module runs once per invocation, outputs JSON, exits. Waybar handles polling via `interval` config.

**Why one-shot execution:**
- Simpler process lifecycle (no ticker management)
- Waybar already provides interval mechanism
- Lower memory footprint
- Easier debugging

**Waybar JSON Protocol:**

Custom modules with `"return-type": "json"` expect single-line JSON on stdout:

```json
{"text": "display", "tooltip": "hover text", "class": "css-class", "percentage": 50}
```

Protocol rules:
- Single line output (no pretty printing)
- stdout for JSON only
- stderr for errors/logs
- Exit immediately after output

## Project Structure

Core Go entrypoint lives in `cmd/waybar-claude-code`, while reusable logic is grouped under:
- `internal/ccusage` - Claude Code usage API client (API calls, caching, timeout handling)
- `internal/config` - Environment variable parsing
- `internal/format` - Tooltip rendering helpers with business rules
- `pkg/waybar` - Shared Waybar JSON protocol (reusable across modules)

Assets for docs and demos live in `assets/`, ready-to-copy Waybar + CSS samples in `examples/`. Keep binaries out of Git; `waybar-claude-code` at root is compiled output only.

## Development Workflow

**Build, test, run:**
```bash
make build                  # Produces static binary with version metadata
make install                # Drops binary into ~/.config/waybar/modules/
make run RUN_INTERVAL=15    # Local debugging with 15s refresh
make fmt                    # Runs go fmt
make lint                   # Runs go vet + staticcheck
make test                   # Runs go test -race -cover ./...
```

Run `make fmt`, `make lint`, `make test` before any PR.

**Build flags:**
- `CGO_ENABLED=0` - static binary
- `-ldflags="-s -w"` - strip debug symbols
- `-X main.version` - embed version from git

**File locations for testing:**

Personal system (live testing):
- `~/.config/waybar/modules/custom-claude-code.jsonc` - module config
- `~/.config/waybar/user-style.css` - CSS styles
- `~/.config/waybar/modules/waybar-claude-code` - compiled binary

Repository examples (update after testing):
- `examples/waybar-config.jsonc` - reference module config
- `examples/style.css` - reference CSS

Test workflow:
1. Edit personal system files
2. `make build && make install`
3. `systemctl --user restart hyde-Hyprland-bar.service` (HyDE) or `pkill -SIGUSR2 waybar`
4. Verify output
5. Copy to examples/

## Code Standards

**Style and naming:**
- Rely on `go fmt` defaults (tabs, single blank lines between top-level declarations)
- Exported APIs use UpperCamelCase
- Internal helpers stay lowerCamelCase
- CLI flags and env knobs follow `CLAUDE_*` pattern
- Waybar config keys mirror JSON schema in `examples/waybar-config.jsonc`
- Keep files small and cohesive per package

**Testing expectations:**
- Place `_test.go` beside code under test
- Use `TestXxx` naming for `go test ./...` discovery
- Prefer table-driven tests for formatters and JSON serializers
- Include integration coverage for Waybar payload
- Keep race detector clean (`-race` flag)
- Maintain current coverage level (`-cover` enforces regressions)

## Configuration

**Waybar config:**
```jsonc
{
  "custom/claude-code": {
    "return-type": "json",
    "exec": "~/.config/waybar/modules/waybar-claude-code",
    "format": "{text}",
    "interval": 300,
    "restart-interval": 30,
    "tooltip": true,
    "on-click": "kitty -e claude"
  }
}
```

**CSS styling:**

GTK CSS limitations:
- No `transform` property (no scale/rotate/translate)
- No `font-size` animation support
- Working properties: `color`, `background-color`, `box-shadow`, `opacity`
- Supported pseudo-classes: `:hover`, `:active`, `:focus` (must be last element in selector)

Hover effect:
```css
#custom-claude-code {
    padding: 0 10px;
    margin: 0 2px;
    color: inherit;
    transition: color 0.2s ease-in-out;
}

#custom-claude-code:hover {
    color: #ff8c00;
}
```

**HyDE integration:**

HyDE manages waybar via Python watcher and systemd service.

Key paths:
- Layouts: `~/.local/share/waybar/layouts/hyprdots/`
- Styles: `~/.local/share/waybar/styles/`
- State: `~/.local/state/hyde/staterc`
- Config: `~/.config/waybar/config.jsonc` (auto-generated, do not edit)

Reload after layout changes:
```bash
~/.local/lib/hyde/waybar.py --set hyprdots/04-claude-stats
```

**NEVER use `pkill waybar && waybar &`** - creates duplicate bars.

## Commit & Release

**Conventional commits:**

Commits follow Conventional Commit prefixes (`feat`, `fix`, `docs`, `chore`, `test`, `ci`) as seen in git log. Keep messages imperative and scoped to single concern.

Examples from history:
```
feat: add tooltip hover support
fix: include cache tokens in input count
docs: simplify README examples
chore: force CSS language detection
ci: fix Go version to 1.23
```

**Pull requests:**

PRs should describe user-facing impact, list test commands run, link tracked issues, include updated screenshots when UI output (icons, tooltip) changes. Mention new env vars or Waybar config knobs for downstream config updates.

**Releases:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hxreborn/waybar-claude-code](https://github.com/hxreborn/waybar-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
