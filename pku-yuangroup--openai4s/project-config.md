---
trigger: always_on
description: This file provides shared guidance to both Claude Code (`CLAUDE.md`) and Codex (`AGENTS.md`, symlinked to this file) when working with code in this repository.
---

# CLAUDE.md / AGENTS.md

This file provides shared guidance to both Claude Code (`CLAUDE.md`) and Codex (`AGENTS.md`, symlinked to this file) when working with code in this repository.

Keep future guidance platform-neutral unless a section explicitly calls out a Claude Code- or Codex-specific workflow.

## What this is

**OpenAI4S** is a pure-stdlib hybrid scientific research agent: provider-native JSON `Tool` calls form the orchestration/permission control plane, while persistent Python/R Code-as-Action kernels form the scientific execution plane. Python can call the in-kernel `host` singleton synchronously mid-cell; R is an independent analysis channel.

**Core is zero-dependency by design.** The engine, the LLM client (over `urllib`), and the web server (`http.server` + a hand-rolled WebSocket) use only the Python standard library. This is a hard constraint — see conventions below.

## Commands

```bash
./setup.sh                          # one-time: locked lightweight .venv + pre-commit hook
./setup.sh --with-kernel-envs       # also create comprehensive Python + R envs
./setup.sh --update-kernel-envs     # sync existing Python + R envs, without pruning
./start.sh                          # launch daemon + web UI at http://127.0.0.1:8760/
uv run pytest                       # full offline test suite (LLM is mocked — no network, no keys)
uv run pytest -n auto --maxprocesses=4 --dist loadfile   # the same suite, the way CI runs it (~7min vs ~20min)
uv run pytest tests/test_agent.py::test_max_turns_stop   # a single test
uv run pytest tests/test_kernel.py -k background         # tests matching a pattern
uv run mypy                         # strict agent/Host + orchestration contracts
uv run pre-commit run --all-files   # format + lint + mypy (black · isort --profile black · ruff)
uv run openai4s run "…" -v          # run one Code-as-Action task in-process, no daemon
openai4s setup --profile standard   # build Python + R from envs/*.yml
openai4s setup                      # build all 4 envs (--dry-run to preview)
```

CLI subcommands (`openai4s <cmd>`): `serve` · `status` · `stop` · `url` · `run` · `init` · `setup` · `doctor` · `diagnostics` · `verify-package` · `inspect-package` · `benchmark` · `env` (`plan`/`apply`/`list`/`rollback`/`recover`) · `jupyter` (`describe`/`export`/`install`) · `share` · `relay`. `start.sh` just runs `openai4s serve`.

**`uv run pytest` is not the whole gate.** CI runs each of the following as its own job, and each fails independently — a green pytest run says nothing about them:

```bash
uv run python scripts/check_directory_readmes.py            # bilingual per-directory README coverage
uv run python3 -m harness.cli run --tier pr --offline       # deterministic scenario contracts
uv run python scripts/capture_response_schemas.py --check   # frozen response shapes still match reality
uv run python scripts/capture_response_contract.py --check  # every routable route still has a contract
python scripts/source_secret_scan.py                        # no credential-shaped literal in release sources
bash scripts/container_smoke.sh                             # the image builds AND the daemon runs inside it
OPENAI4S_KERNEL_SANDBOX=enforce OPENAI4S_KERNEL_ALLOW_RAW_NETWORK=1 \
  uv run python -m harness.smoke.linux_bwrap_interrupt      # Linux+bwrap: Python/R persistent SIGINT
OPENAI4S_KERNEL_SANDBOX=enforce \
  uv run python -m harness.smoke.linux_sandbox              # Linux+bwrap: full filesystem/egress boundary
node tests/browser_smoke.mjs                                # workbench E2E, needs a daemon on :8760
node tests/browser_admission_fault.mjs                      # pinned-comment admission survives a lost response
node tests/browser_sandbox_preview.mjs                      # artifact preview executes on the sandbox origin (owns its daemon)
node tests/browser_matrix.mjs --browser=firefox             # cross-engine breadth (chromium/firefox/webkit)
OPENAI4S_STAGE0_SELF_TEST=1 node tests/browser_stage0_acceptance.mjs   # Stage 0 harness self-check, no browser/daemon
node tools/skills-installer/selftest.mjs                    # the npx Skill installer: extraction safety, install/uninstall
uv sync --locked --extra singlecell && \
  uv run python -m pytest tests/test_single_cell_rna_analysis_skill.py -m "not network"   # the locked single-cell stack contracts (py3.11+)
node tools/skills-installer/check_package.mjs               # `npm pack` still carries the CLI *and* the Skill tree
```

The hosted-Linux interrupt smoke deliberately allows raw worker networking so
its private-PID evidence is independent of network-namespace setup. It proves
the team info-fd/procfs/pidfd signal path and post-interrupt kernel reuse, not
the full Linux filesystem-and-egress boundary. That full boundary is a separate
independent job (`harness.smoke.linux_sandbox`) that refuses the raw-network
override; it is attested at the frozen SHA and stays out of the release
workflow's `platform-checks` matrix until multiple scheduled greens land.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PKU-YuanGroup/OpenAI4S](https://github.com/PKU-YuanGroup/OpenAI4S) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
