---
trigger: always_on
description: Operating instructions for any agent (or person) modifying Horizun PBI MCP.
---

# Repository rules for agents

Operating instructions for any agent (or person) modifying Horizun PBI MCP.
They take priority over any general convention.

---

## 1. Before touching anything

```bash
python -m pytest -q                    # must pass green
python scripts/doctor.py               # must exit with code 0
```

If the baseline is already broken, **fix it or report it first**. Don't build on red.

---

## 2. The MCP contract is untouchable

The **34** baseline tools are frozen in `tests/golden/tools_v1.json`.

**Forbidden without explicit approval:**
- removing a tool
- renaming a tool
- removing a parameter
- adding a **required** parameter
- changing a parameter's type
- changing a default value
- changing the response shape

**Allowed:**
- adding new tools
- adding **optional parameters with a default**
- adding new fields to the response dict
- improving descriptions

**Adding a tool also means classifying it** in `src/tools/risk.py`. The suite
fails if you don't: an unclassified tool is announced to the client as
destructive — which is the safe default, but the table has to say so out loud.
Never declare as `read_only` anything that calls `guard_mutation` or writes a
file; `tests/test_tool_annotations.py` checks that against the code, not
against the docs.

Check at any time:

```bash
python -m tests.contract_utils
```

Returns 0 if there are no breaks, 1 if there are, with a report stating **what** changed and **whether it breaks compatibility** — not a dump of two JSONs.

After a deliberate, approved change:

```bash
python -m tests.contract_utils --write
```

---

## 3. Invariants no phase can break

1. **stdout is the JSON-RPC channel.** All logging goes to stderr or a file. A debug `print()` breaks the client connection.
2. **Never overwrite JSON that doesn't parse.** If it can't be read, abort.
3. **Every write to the user's project:** backup before, re-read after.
4. **No write path leaves the active project.** Use `ensure_within_base()`.
5. **No fields are invented** that don't exist in the model. If a field doesn't exist, report it; don't guess.
6. **Destructive tools require `confirm=true`.**
7. **Prefer cloning a real template** over hand-building visual JSON.

---

## 4. Real data: never enters git

| Never version | Do version |
|---|---|
| Real `.pbix`, `.pbip`, `.Report/`, `.SemanticModel/` | `tests/fixtures/synthetic/**` |
| `libs/` (DLLs) | `scripts/fetch_libs.py` |
| `outputs/`, `backups/`, `*.log` | `*.example.*` templates |
| `.env`, `.mcp.json`, credentials | `.env.example`, `.mcp.json.example` |
| `tests/fixtures/local/` | `docs/`, `tests/` |

Before any commit:

```bash
git status --short --ignored
```

Synthetic fixtures **contain no** commercial names, data or information from any real project. If you need real PBIR structure, use the ignored local fixture (`scripts/setup_local_fixture.py`) and **never promote it to `synthetic/` without anonymizing it and without review**.

---

## 5. Tests

| Level | Where | Rule |
|---|---|---|
| Unit | `tests/test_*.py` | No real I/O outside `tmp_path` |
| Synthetic fixtures | `tests/fixtures/synthetic/` | Use `materialize(tmp_path)`. **Never** write over the versioned fixture |
| MCP contract | `tests/test_tool_contract.py` | Must always pass |
| Live | marked `@pytest.mark.skip` or `live` | Not run on their own. Never destructive against a real model |
| Local fixture | marked `local_fixture` | Read-only. Skipped if the folder doesn't exist |

**Scripts that drive a real Power BI Desktop** (ad-hoc checks, not part of the
suite) must **isolate the persisted session before doing anything else**:

```bash
HORIZUN_PBI_MCP_OUTPUTS_DIR=/some/scratch/outputs python your_live_script.py
```

Opening a project persists it (`project_locator.open_project()` writes
`outputs/session.json`), so without this a check leaves the user's session
pointing at a temporary fixture. The variable is read once, when settings are
first resolved, so it has to be set **before** the process imports anything
that touches them. Same rule for the windows themselves: identify every
process and file before acting, close only the instances the script opened,
and never point one at a real project. `tests/test_correcciones_de_auditoria.py`
(group 22) pins the isolation route.

**Path traversal tests:** the "outside" must be created **inside pytest's `tmp_path`** (`synthetic.outside_marker_dir()`). Never point at a real machine path, not even to demonstrate a failure.

---

## 6. Git and contributions

This is the **public** repository. Contributions go through **branches and pull requests**.

- **Never `force-push` to `main`.** Rewriting published history breaks any clone and any reference to a commit.
- One branch per change, with a name that says what it does.
- **Before opening a PR**, all three green:

  ```bash
  python -m pytest -q
  python scripts/doctor.py
  python -m tests.contract_utils
  ```

  CI repeats them on `windows-latest` with Python 3.10 and 3.13. A red PR is not reviewed.

- **The MCP contract is frozen.** See section 2: adding is free, changing or removing is not.
- **Real data is never versioned**: no one's `.pbix`, no one's `.pbip`, no DLLs, no `outputs/`, no `backups/`, no `.env`, no `.mcp.json`. See section 4.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HorizunGroup/horizun-pbi-mcp](https://github.com/HorizunGroup/horizun-pbi-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
