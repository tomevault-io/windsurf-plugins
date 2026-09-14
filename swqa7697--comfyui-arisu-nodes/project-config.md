---
trigger: always_on
description: ComfyUI custom node pack on the V3 API (`comfy_entrypoint` + `io.Schema`). Python >=3.10 (developed on 3.13), uv-managed, `src` layout. ComfyUI imports the repo-root `__init__.py`. Nodes are grouped by family in `src/arisu_nodes/<family>/`: `core.py` needs only the stdlib and Pillow, `nodes.py` needs ComfyUI's source tree and torch. `README.md` is the human-facing guide; this file is for agents.
---

# CLAUDE.md — ComfyUI-Arisu-Nodes

ComfyUI custom node pack on the V3 API (`comfy_entrypoint` + `io.Schema`). Python >=3.10 (developed on 3.13), uv-managed, `src` layout. ComfyUI imports the repo-root `__init__.py`. Nodes are grouped by family in `src/arisu_nodes/<family>/`: `core.py` needs only the stdlib and Pillow, `nodes.py` needs ComfyUI's source tree and torch. `README.md` is the human-facing guide; this file is for agents.

## Agent integration

Claude Code reads this file directly; Codex reads the root `AGENTS.md` symlink to it.
Keep that symlink and edit this file for shared project rules. Codex discovers
`.agents/skills/release-pr`, a relative symlink to `.claude/skills/release-pr`;
edit the shared skill there. Invoke it as `/release-pr` in Claude Code or
`$release-pr` in Codex. Both agents use the same ignored `.claude/comfyui-env.md`
and its tracked template; do not create a second set of machine facts for Codex.
Run development commands from this checkout. Personal Codex settings in `.codex/`
are ignored; this project needs no model, credential, or permission overrides.
Agent configuration and skills are excluded from the Registry archive.

## Hard boundary: the ComfyUI install at `$COMFYUI_PATH`

`$COMFYUI_PATH` names a live ComfyUI install on the developer's machine, the one tree this repo must never write to. `make comfyui-path` prints the resolved path; this file never spells it out, and an unset `COMFYUI_PATH` means the live install, never "somewhere safe". Machine facts live in the untracked `.claude/comfyui-env.md` (template `.claude/comfyui-env.example.md`); read it if present, otherwise ask rather than guess. The install is an always-on user systemd unit on the only GPU (restarting it is the owner's test loop) and a uv project alive only because `uv run` syncs inexactly: one `uv sync` there removes every package, and with no `pip` in the `.venv` and a CUDA torch wheel from an index `requirements.txt` omits, a rebuild costs tens of minutes.

Forbidden for the agent, without exception:
- Any uv project command with the working directory under `$COMFYUI_PATH` (`uv sync`, `uv add`, `uv remove`, `uv lock`, `uv run` without `--no-project`); `--active` on any uv command; `UV_PROJECT_ENVIRONMENT` or `VIRTUAL_ENV` pointing at `$COMFYUI_PATH/.venv` (shell, `.env`, script).
- `pip install` or any `uv pip install|uninstall|sync` targeting `$COMFYUI_PATH/.venv/bin/python`.
- Creating, editing, deleting, or `chmod` on anything under `$COMFYUI_PATH` (`.venv/` and `custom_nodes/` included); `git` writes to that checkout; `systemctl --user start|stop|restart`.

The one carve-out is CI: `.github/workflows/comfyui-lane.yml` clones ComfyUI into the runner's temp dir and points `COMFYUI_PATH` there, where `git clone`, `uv venv`, `uv pip install`, and editing `requirements.txt` are fine. Never point that workflow, or `COMFYUI_PATH`, at a developer's install; a local disposable clone unlocks nothing either.

Allowed reads, all leaving `$COMFYUI_PATH` byte-identical: `uv run --no-project --python "$COMFYUI_PATH/.venv/bin/python" --with <pkg> ...` with `PYTHONDONTWRITEBYTECODE=1`, as `scripts/test-comfyui.sh` does; `uv pip freeze -p "$COMFYUI_PATH/.venv/bin/python"`; reading and grepping the source tree; `systemctl --user status`, `journalctl --user -u`, and `curl -s <api_endpoint>/object_info` for the unit and endpoint in `.claude/comfyui-env.md`.

Human-only steps, which the agent may print but never runs: manual E2E (clone a pushed commit into `$COMFYUI_PATH/custom_nodes/`, restart, test in the browser, remove the clone, restart; never symlink the working copy); additive `uv pip install --python "$COMFYUI_PATH/.venv/bin/python" <pkg>`, preceded by a snapshot `uv pip freeze -p <that python> > "$COMFYUI_PATH/venv-snapshot-$(date +%F).txt"` (none exists yet); recovery via `uv pip install --python <that python> -r <snapshot>` with the torch family from `https://download.pytorch.org/whl/<cuXXX>` for the recorded build; and the optional `chmod -R a-w "$COMFYUI_PATH/.venv"` lock, which ComfyUI upgrades and ComfyUI-Manager need lifted.

## Layout

- `__init__.py` — the module ComfyUI imports: `ComfyExtension` subclass + `comfy_entrypoint`.
- `src/arisu_nodes/<family>/core.py` — logic on the stdlib and Pillow (the pack's one declared dependency, which ComfyUI already installs), no ComfyUI or torch. `nodes.py` — `io.ComfyNode` classes importing `comfy_api` and torch; ends with `NODES: List[Type[io.ComfyNode]]`, which the root `__init__.py` concatenates. `routes.py` (only `common` has one) — aiohttp handlers behind frontend buttons and the browse dialog, registered on `PromptServer.instance.routes` from the extension's `on_load`; validation stays in `core.py`. Families: `minimax_h3` and `common` (live), `anima` (reserved). Every subpackage needs an `__init__.py`: `find_packages` silently drops a directory without one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swqa7697/ComfyUI-Arisu-Nodes](https://github.com/swqa7697/ComfyUI-Arisu-Nodes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
