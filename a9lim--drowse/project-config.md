---
trigger: always_on
description: `saklas` is a Python library + dual-protocol HTTP server for activation steering
---

# AGENTS.md

## What this is

`saklas` is a Python library + dual-protocol HTTP server for activation steering
and trait monitoring on HuggingFace causal LMs. It runs OpenAI `/v1/*` and
Ollama `/api/*` on one port, plus a native `/saklas/v1/*` API and a Svelte
dashboard at `/`. Steering signal comes from representation engineering, unified
under a single artifact family — the **manifold**: labeled nodes placed on a
domain, fit to a per-layer subspace. A difference-of-means steering vector is
the 2-node flat case; `personas` is a 107-node flat fan; `emotions` is a 20-node
affect manifold over PAD. Every steering term — vectors, poles, `~`/`|`
projections, `!` ablations, and `%` manifold positions — lowers at generation
time to one unified per-layer injection (the along/onto subspace kernel,
`core/manifold.py::subspace_inject`). Per-call coefficients, no model mutation.
Two frontends over one engine: `SaklasSession` (programmatic) and `saklas serve`
(HTTP APIs plus the web dashboard).

Version lives in `saklas/__init__.py` as `__version__`. `pyproject.toml` reads
it via `version = {attr = "saklas.__version__"}`, so there is one place to bump.
Do not bump it as part of feature work — version bumps are user-owned.

Releases: merge a version bump to `main` → `.github/workflows/release.yml` tags
`v$VERSION`, builds, publishes via trusted publishing, and cuts a GitHub
release. A push without a bump is a no-op.

The cross-cutting design — how extraction, composition, injection, and reads
fit together — lives in the repo-root `ARCHITECTURE.md`. Read it before
touching the engine.

## Subtree docs

Deep internals live in subtree `AGENTS.md` files — Claude Code auto-loads each
when you work in that directory. Consult them only when editing that layer.

- `saklas/core/AGENTS.md` — model loading, the manifold/subspace fit +
  injection, monitor + instruments, session, generation loop, loom tree
- `saklas/io/AGENTS.md` — manifold format, HF distribution, GGUF, merge,
  alignment, paths/selectors, source registries
- `saklas/cli/AGENTS.md` — eight-verb dispatch, config loading, flags
- `saklas/server/AGENTS.md` — OpenAI / Ollama / native routes, WS protocol
- `saklas/web/AGENTS.md` — dashboard mount, wire protocol, Svelte source layout

## Commands

```bash
pip install -e ".[dev]"                         # editable + pytest + SAELens
pip install -e ".[gguf]"                        # llama.cpp GGUF I/O
pip install -e ".[cuda,flash]"                  # bitsandbytes + kernels + tested FlashAttention (Linux/CUDA)
saklas serve <model_id> [--no-web] [--steer/-S EXPR]
saklas manifold extract <concept>|<pos> <neg> [-m MODEL] [--sae RELEASE] [--role SLUG] [--kind abstract|concrete|custom] [--system TEMPLATE] [--namespace NS] [--no-dls] [-f]
saklas manifold generate <name> --concepts C... [--kind abstract|concrete|custom] [--system TEMPLATE] [--samples-per-prompt K] [--seed S]
saklas manifold from-template <template> [--name MANIFOLD] [--fit-mode auto|pca|spectral] [--max-dim N] [--var-threshold T] [--description TEXT] [-f]
saklas manifold fit <name>|<folder> [-m MODEL] [--sae REL] [--layers L1,L2|workspace|all] [--method pca|spectral|auto] [--max-dim N] [--min-dim N] [--var-threshold T] [--k-nn K] [--bandwidth SIGMA] [--max-subspace-dim R] [--smoothing auto|0|LAMBDA] [--persistence-frac F] [--no-dls]
saklas manifold bake <name> <expression> [-m]    # additive subset: "0.3 ns/a + 0.5 ns/b"
saklas manifold merge <name> <src...> [-f]           # union discover-mode node corpora
saklas manifold transfer <name> --from SRC --to TGT [-f]   # cross-model Procrustes
saklas manifold compare <concepts...> -m MODEL [--ridge-scale R]
saklas manifold why <concept> -m MODEL [-j]       # per-layer ||baked|| as a 16-bucket histogram
saklas pack ls [-v|-j] | show <name> [-j]            # list / inspect manifolds
saklas pack install <target> [-a NS/N] [-f]          # HF coord or current local manifold folder
saklas pack search <query> [-j|-v]                   # search HF hub for saklas-manifold repos
saklas pack push <name> [-a OWNER/N] [-m MODEL] [--variant raw|sae|all] [--private] [--dry-run]
saklas pack rm <name> -y                             # remove folder (bundled respawns); -y required
saklas pack clear <name> [-m MODEL] [--variant raw|sae|all]   # delete per-model fitted tensors
saklas pack refresh <name> [-m MODEL]                # re-pull (hf) / re-fit (-m scoped)
saklas pack export gguf <name> [-m MODEL] [-o PATH] [--model-hint HINT]   # fold a 2-node pca manifold to a control-vector GGUF
saklas experiment fan <model> "<prompt>" -g concept=0,0.5,1 [-S EXPR] [--max-tokens N]   # alpha grid as loom siblings
saklas experiment transcript run <path.yaml> [model] [--max-tokens N]     # replay a saved transcript
saklas experiment naturalness <model> "<prompt>" --manifold F -S EXPR [--max-tokens N]   # behavior-manifold eval
saklas template create <name> --slot TOKEN --values V... --contexts FILE [--description TEXT] [-f]
saklas template ls [-j] | show <name> [-j] | rm <name> [-y]
saklas template score <name> -m MODEL [-S EXPR] [--by sum|mean] [-j]   # restricted-choice value distribution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a9lim/drowse](https://github.com/a9lim/drowse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
