---
trigger: always_on
description: - Use `uv run SCRIPT.py` or `uv run python ARGS` to run python instead of Just plain `python`.
---

- Use `uv run SCRIPT.py` or `uv run python ARGS` to run python instead of Just plain `python`.
- To submit jobs, use `launch.py` scripts (e.g., `uv run python scripts/launch.py SUBCOMMAND`). Nearly every part of this package has a launch.py entrypoint. Don't try to run modules directly with `-m`.
- In `contrib/trait_discovery/`, use `uv run python scripts/launch.py SUBCOMMAND` (e.g., `cls::train`, `cls::eval`, `probe1d`). Note the `::` separator for subcommands.
- After making edits, run `uvx ruff format --preview .` to format the file, then run `uvx ruff check --fix .` to lint, then run `uvx ty check FILEPATH` to type check (`ty` is prerelease software, and typechecking often will have false positives). Only do this if you think you're finished, or if you can't figure out a bug. Maybe linting will make it obvious. Don't fix linting or typing errors in files you haven't modified.

# Gather Context

- Public docs for developers and users are in markdown in docs/src. Internal, messier design and implementation docs are in markdown in docs/research/issues. Both are valuable sources of context when getting started.
- You can use `gh` to access issues and PRs on GitHub to gather more context. We use GitHub issues a lot to share ideas and communicate about problems, so you should almost always check to see if there's a relevant GitHub issue for whatever you're working on.

# Code Style

- Don't hard-wrap comments. Only use linebreaks for new paragraphs. Let the editor soft wrap content.
- Don't hard-wrap string literals. Keep each log or user-facing message in a single source line and rely on soft wrapping when reading it.
- Prefer negative if statements in combination with early returns/continues. Rather than nesting multiple positive if statements, just check if a condition is False, then return/continue in a loop. This reduces indentation.
- This project uses Python 3.12. You can use `dict`, `list`, `tuple` instead of the imports from `typing`. You can use `| None` instead of `Optional`.
- File descriptors from `open()` are called `fd`.
- Use types where possible, including `jaxtyping` hints.
- Decorate functions with `beartype.beartype` unless they use a `jaxtyping` hint, in which case use `jaxtyped(typechecker=beartype.beartype)`.
- Variables referring to a absolute filepath should be suffixed with `_fpath`. Filenames are `_fname`. Directories are `_dpath`.
- Prefer `make` over `build` when naming functions that construct objects, and use `get` when constructing primitives (like string paths or config values).
- Only use `setup` for naming functions that don't return anything.
- submitit and jaxtyping don't work in the same file. See [this issue]. To solve this, all jaxtyped functions/classes need to be in a different file to the submitit launcher script.
- Never create a simple script to demonstrate functionality unless explicitly asked.
- Write single-line commit messages; never say you co-authored a commit.
- Before committing, run `git status` to check for already-staged files. If asked to commit only specific files, unstage everything first, then stage only the requested files, then after the commit, restage the already-staged files.
- Only use ascii characters. If you would use unicode to represent math, use pseudo-LaTeX instead in comments: 10⁶ should be 10^6, 3×10⁷ should be 3x10^7.
- Prefix variables with `n_` for totals and cardinalities, but ignore it for dimensions `..._per_...` and dimensions. Examples: `n_examples`, `n_models`, but `tokens_per_example`, `examples_per_shard`
- Try to keep code short. Shorter code is in principle easier to read. If variable names are really long, shorten based on conventions in this codebase (..._indices -> ..._i). Since you use `uvx ruff format --preview`, if you can make a small variable name change to fit everything on one line, that's a good idea. When variables are used once, simply inline it.
- If you make edits to a file and notice that I made edits to your edits, note the changes I make compared to your initial version and explicitly describe the style of changes. Keep these preferences in mind as you write the rest of the code.
- Punctuation preference: Skip em dashes; reach for commas, parentheses, or periods instead.
- Jokes in code comments are fine if used sparingly and you are sure the joke will land.
- Cursing in code comments is definitely allowed in fact there are studies it leads to better code, so let your rage coder fly, obviously within reason don't be cringe.

# Defensive Programming

- Consider the [style guidelines for TigerBeetle](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md) and adapt it to Python.
- Use asserts to validate assumptions frequently. For example, I didn't have an assert here at first because I assumed the shape couldn't change. It turns out it can! So now we have an assert to make it clear that we expect the input and output shapes are identical.
```py
def sp_csr_to_pt(csr: scipy.sparse.csr_matrix, *, device: str) -> Tensor:
    shape_sp = csr.shape
    pt = torch.sparse_csr_tensor(
        csr.indptr,
        csr.indices,
        csr.data,
        size=shape_sp,
        device=device,
    )
    # MISSING

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Imageomics/saev](https://github.com/Imageomics/saev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
