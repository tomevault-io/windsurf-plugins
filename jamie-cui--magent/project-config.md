---
trigger: always_on
description: Guidance for agentic coding tools working with Magent.
---

# AGENTS.md

Guidance for agentic coding tools working with Magent.

## Boundaries

- **Do not** implement Codex sandbox/seatbelt/bubblewrap/shell isolation. Preserve Emacs-native workflow (live buffers, `emacs_eval`, agent-shell, project-scoped sessions, gptel transport).
- **Keep** `gptel-request` for provider/HTTP/SSE plumbing. Do not rewrite gptel integration.
- **Child-agent tools**: `spawn_agent`, `send_agent_message`, `wait_agent`, `list_agents`, `close_agent` replaced the old `delegate` tool. Do not reintroduce a compatibility wrapper unless explicitly requested.
- **Interrupted work**: Update relevant docs or task notes before stopping so progress is recoverable from git.

See `docs/AGENT_JOBS.org` and `docs/AGENT_WORKFLOW.org` for child-agent lifecycle details.

## Build Commands

```bash
make compile    # Byte-compile all Elisp files
make lint       # Validate declarations and package metadata
make test-unit  # Run ERT unit tests in batch mode
make test       # Run unit tests plus deterministic live smoke tests
make coverage   # Run ERT under testcover and write coverage/testcover-summary.tsv
make clean      # Remove build files and Harbor benchmark containers/networks
make purge      # Also remove benchmark Docker images and Harbor task cache
```

The Makefile auto-detects dependency paths (`gptel`, `acp`, `shell-maker`, `agent-shell`, `cond-let`, `compat`, `yaml`, `llama`, `with-editor`, `package-lint`) by scanning `~/.emacs.d/elpa/`. Override any with e.g. `GPTEL_DIR=/path/to/gptel`.

Single-file compilation:
```bash
emacs -Q --batch -L lisp -L ~/path/to/gptel -f batch-byte-compile lisp/magent-foo.el
```

Run a single test by regexp:
```bash
emacs -Q --batch -L lisp -L $(find ~/.emacs.d/elpa -maxdepth 1 -name 'gptel-*' -type d | head -1) \
  -l ert -l test/magent-test.el --eval '(ert-run-tests-batch "test-name-regexp")'
```

## Testing

### Unit Tests

`test/magent-test.el` contains the main ERT suite. Tests mock `gptel-request` and frontend/runtime functions via `cl-letf`. Key patterns:
- Registry tests bind `magent-agent-registry--agents` to a fresh hash table
- Skills tests bind `magent-skills--registry` to nil
- Session tests call `magent-session-reset` to clear global state
- Loop/tool permission tests should use `magent-agent-loop.el` and `magent-tool-orchestrator.el`

### Coverage

`test/coverage.el` is the batch `testcover` runner used by `make coverage` and GitHub Actions. It instruments Magent sources, reloads built-in skill/capability directories from this checkout, runs `test/magent-test.el`, and writes `coverage/testcover-summary.tsv`.

### GitHub Actions

CI is defined under `.github/workflows/`:
- `test.yml`: tests Emacs 29.4 and 30.1 via Nix, installs package dependencies, runs `make compile`, `make lint`, `make test-unit`, and `make test-live-smoke` in a temporary daemon.
- `coverage.yml`: runs `make coverage` and uploads `coverage/testcover-summary.tsv`.
- `melpazoid.yml`: runs MELPA-style package checks. Its recipe explicitly includes `lisp/magent*.el`, `prompts/`, `skills/`, and `capabilities/`, because Magent depends on those production libraries and bundled data at runtime. Keep the explicit library and runtime-data entries aligned with the package source manifest.

Package metadata should stay centralized in `magent.el` and `magent-pkg.el`; non-main modules should not carry `Package-Requires` headers. Keep SPDX license identifiers in every Elisp source file so melpazoid can detect licensing consistently.

### Local melpazoid Reproduction

When reproducing the melpazoid job locally:

1. Use the exact recipe from `.github/workflows/melpazoid.yml` and set `LOCAL_REPO` to this checkout. Verify the staged `pkg/` contains the complete flattened production Elisp set, excludes ERT sources under `test/`, and includes the bundled runtime-data directories before trusting later checks.
2. Check for an existing `melpazoid:latest` image before rebuilding. It may be reused only when its installed dependency set matches the newly generated `_requirements.el`; never treat the package sources embedded in an old image as current.
3. To reuse a compatible image, bind-mount the newly staged `pkg/` over `/workspace/pkg` and separately mount the current `melpazoid.el`, because the package mount hides the checker bundled in the image. The staged package root must be writable by the container user because byte compilation creates `.elc` files.
4. Start each rerun from a fresh staged package, or remove checker-generated `.elc` and autoload files first. Otherwise a second run can lint artifacts that were not part of the MELPA recipe and produce misleading extra warnings.
5. Run the container with `--network=none`, matching melpazoid's normal test target. Success for a packaging regression requires a zero exit status and a clean packaged `#'load` check; source-tree compilation alone does not prove that the recipe copied every runtime file.

Keep `magent-test-melpazoid-recipe-packages-production-libraries` aligned with the workflow recipe so unit tests catch missing production libraries or runtime-data entries. When adding a production module, also keep `magent-test-source-manifest-covers-production-elisp` green.

### End-to-End Testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jamie-Cui/magent](https://github.com/Jamie-Cui/magent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
