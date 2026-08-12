---
trigger: always_on
description: The unit tests can be run with `uv run python -m unittest`.
---

# Primal-Dual Lagrangian Interior Point Algorithm (LIPA)

## Running the unit tests

The unit tests can be run with `uv run python -m unittest`.
This handles the environment set up properly and runs all tests.

It's important to check the solver logs in the unit tests, and not look only
at the exit status. This is especially true if you modify passing criteria (which you should avoid).

If `uv` is not installed, install it via `curl -LsSf https://astral.sh/uv/install.sh | sh`.
Make sure to request the user for permission to install it.
If this doesn't work, refer to the `Installation` section in https://github.com/astral-sh/uv.

## Bootstrapping the project venv

If `uv run python -c "import jax"` fails with `ModuleNotFoundError` — this happens when
the container is restarted and the project's `.venv/` is wiped — rebuild from scratch:

```bash
rm -rf .venv
uv pip install --python .venv/bin/python hatchling editables   # required, see "Pitfall" below
uv sync --python 3.13 --all-extras --all-groups
```

The `--all-extras --all-groups` flags pull in everything: the `test` / `lint` / `mpc-examples`
extras AND the `comparisons` dependency-group (CasADi, mim-solvers, sip-python, trajax, etc.)
that the LIPA-vs-SOTA benchmark in `tests/comparison/` needs. Without them, most adapters
import-error out at runtime.

**Pitfall:** if the environment has `UV_NO_BUILD_ISOLATION=1` set (check with
`echo $UV_NO_BUILD_ISOLATION`), `uv sync` skips the per-build isolated env and instead
expects build backends to already be installed in the project venv. Since `primal-dual-lipa`
is an editable install with `[build-system] requires = ["hatchling"]`, the first `uv sync`
against a fresh empty venv fails with `ModuleNotFoundError: No module named 'hatchling'`.
Install `hatchling` (and its peer dep `editables`) into the venv first, then `uv sync`
proceeds. Always pass `--python .venv/bin/python` to `uv pip install` so the install
targets the project venv — without it, uv may pick up a different Python (e.g. a system
or shared conda env, which is typically read-only and the wrong target).

Verify the bootstrap worked:

```bash
LD_LIBRARY_PATH=${LD_LIBRARY_PATH:-}:$PWD/.venv/lib/python3.13/site-packages/cmeel.prefix/lib \
  uv run --no-sync python -c "import jax; import casadi; import mim_solvers; print('ok')"
```

(`--no-sync` is important during normal use too — it skips uv's automatic sync check and
shaves seconds off every invocation, which adds up over thousands of solver subprocess calls
in the benchmark runner.)

**Runtime: `mim_solvers` / CSQP / `libpinocchio_parsers.so.3`.** CSQP (one of the
LIPA-vs-SOTA comparison solvers) links against cmeel-packaged pinocchio. Two gotchas:

1. The cmeel package only ships full-version `.so.X.Y.Z` files and unversioned `.so`,
   but `libcrocoddyl.so` (CSQP's intermediate dep) links against major-version SONAMEs
   like `libpinocchio_parsers.so.3`. Need to manually create the major-version symlinks
   inside `.venv/lib/python3.13/site-packages/cmeel.prefix/lib/`:
   ```bash
   cd .venv/lib/python3.13/site-packages/cmeel.prefix/lib/
   for f in *.so.*; do
     if [[ "$f" =~ \.so\.([0-9]+)\.[0-9]+\.[0-9]+$ ]]; then
       major="${BASH_REMATCH[1]}"
       base="${f%.${BASH_REMATCH[1]}.*.*}"
       target="${base}.${major}"
       [ ! -e "$target" ] && ln -s "$f" "$target"
     fi
   done
   ```
2. Even with symlinks, the loader can't *find* them because `libcrocoddyl.so` has no
   RUNPATH (RUNPATH on `mim_solvers_pywrap.so` covers direct deps only, not transitive).
   Every `uv run`/`.venv/bin/python` invocation that touches CSQP needs the cmeel lib
   directory on `LD_LIBRARY_PATH`:
   ```bash
   export LD_LIBRARY_PATH=${LD_LIBRARY_PATH:-}:$PWD/.venv/lib/python3.13/site-packages/cmeel.prefix/lib
   ```
   Without this, `import mim_solvers` fails with
   `ImportError: libpinocchio_parsers.so.3: cannot open shared object file`.

**Runtime: `acados` / `acados_template`.** The acados SOTA adapter needs both
the prebuilt C library (`libacados.so` + Tera renderer, see
`tests/comparison/acados_install.md` for the one-time CMake build) AND its
Python bindings installed editable into the project venv:

```bash
UV_NO_BUILD_ISOLATION=0 VIRTUAL_ENV=$PWD/.venv \
  uv pip install -e ~/github/acados/interfaces/acados_template \
  --python .venv/bin/python
```

Plus, every invocation that touches acados needs the source dir and lib
dir on the environment:

```bash
export ACADOS_SOURCE_DIR=$HOME/github/acados
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH:-}:$HOME/github/acados/lib
```

Without `ACADOS_SOURCE_DIR`, `acados_template` can't find the Tera renderer
binary or the install headers; without `acados/lib` on `LD_LIBRARY_PATH`,
the generated solver `.so` can't resolve `libacados.so`.

**Pitfall: `LD_LIBRARY_PATH` ordering matters.** Always APPEND
`cmeel.prefix/lib` (and `acados/lib`) to the existing `LD_LIBRARY_PATH`
(`LD_LIBRARY_PATH=${LD_LIBRARY_PATH:-}:.../cmeel.prefix/lib`), never
PREPEND. Prepending can shadow the system's `libstdc++`/CUDA libraries,
which on this host causes `jaxlib` to segfault at the first array
operation (`jnp.linspace` etc.). `tests/comparison/run_all.sh` follows
this convention.

**Pitfall: `acados_template` install upgrades Cython to 3.2.x.** This is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joaospinto/primal-dual-lipa](https://github.com/joaospinto/primal-dual-lipa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
