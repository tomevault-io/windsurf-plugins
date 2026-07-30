---
trigger: always_on
description: 这个文件规范 agent 在 `docs/sphinx/` 下撰写和修改文档内容的行为。仓库顶层
---

# docs/sphinx Agent Guide

这个文件规范 agent 在 `docs/sphinx/` 下撰写和修改文档内容的行为。仓库顶层
`AGENTS.md` / `CLAUDE.md` 仍然优先适用于代码、配置、测试和 PR 流程；本文件只补充
Sphinx 文档写作规则。

## Ground Truth

- 文档基础设施、构建和部署：`docs/sphinx/README.md`
- 本地完整构建并发布到 UniLab-doc：`docs/sphinx/README.md#本地发布到-unilab-doc`
- Sphinx 配置：`docs/sphinx/source/conf.py`
- 架构标准：`docs/sphinx/source/zh_CN/4-developer_guide/0-index.md`
- ADR：`docs/sphinx/source/adr/ADR-0000-index.md`
- 术语表：`docs/sphinx/source/glossary.md`
- 文档检查：`tests/scripts/doc_checks.py`、`tests/scripts/test_check_docs.py`

## Source Structure

```text
source/
├── index.md                 # root redirect → en/0-index.html (no language picker)
├── en/                      # English tree (site root, shown in sidebar nav)
├── zh_CN/                   # Chinese tree (hidden from sidebar, via switcher only)
├── adr/                     # shared ADR, one set
├── api_reference/           # shared autodoc output from src/ docstrings
├── glossary.md
├── changelog.md
├── _static/
└── _templates/
    ├── sidebar/
    │   └── lang_switcher.html   # language dropdown in sidebar
    └── autosummary/
```

`source/en/` is the **default site root**. Visitors land on `/en/0-index.html`
directly (root `index.md` is a redirect). The sidebar navigation tree only
shows the **active language root subtree** — when viewing an English page the
sidebar contains only `en/...` pages; when viewing a Chinese page it contains
only `zh_CN/...`. Shared resources (`adr/`, `api_reference/`, `glossary.md`,
`changelog.md`) **do not appear directly in the sidebar** because they live
outside both language roots.

`source/en/` and `source/zh_CN/` are parallel language roots, but they are
**not currently a strict 1:1 path mirror**. Both roots use numbered section
directories and numbered Markdown files, while some sections exist only in one
language. The language switcher uses an explicit path map in `conf.py`
(`_LANGUAGE_PATH_FORWARD`) to handle the mismatch — **when adding a new English
page, add a corresponding entry to that map** so the switcher lands somewhere
sensible (or omit the entry to fall back to the zh_CN index).

Do not add per-page language button blocks or hand-written cross-language
navigation. The sidebar language switcher handles language changes globally.

### Section indexes and shared resources

Every multi-page section under a language root has a section index page that
introduces the section and contains a hidden toctree of its sibling pages.
Existing unnumbered sections use `index.md` recursively:
`en/3-deployment/0-index.md`, `en/3-deployment/1-sim_to_real/0-index.md`,
`en/4-developer_guide/1-architecture/0-index.md`, `en/2-user_guide/7-tooling/0-index.md`,
etc. Ordered sections may use numbered filenames, such as
`en/1-getting_started/0-index.md`, `1-quick_demo.md`, and `2-installation.md`.
When a section uses numbered filenames, keep the file numbers and toctree order
aligned. Adding a new section means adding its section index page and including
the section in the parent index's toctree.

Shared resources are reached through **language-local wrapper pages**, not by
including the shared docs directly in a language toctree. Example: instead of

```markdown
<!-- BAD: would pull /glossary into the sidebar tree -->
```{toctree}
/glossary
```
```

the English reference section uses

```markdown
<!-- GOOD: links via {doc} without inserting the shared page into the
     sidebar subtree -->
- {doc}`Shared glossary </glossary>`
```

See `en/5-reference/4-adr.md`, `en/5-reference/2-glossary.md`, `en/5-reference/3-changelog.md`
for the established pattern. The wrapper page is what shows up in the
language sidebar; the actual shared content remains accessible via the
language-independent absolute path.

## Core Principles

1. **Evidence only**: only document facts that can be verified in `src/`,
   `conf/`, `tests/`, `scripts/`, ADRs, or generated support data.
2. **Code is the source of truth**: names, signatures, defaults, Hydra keys, and
   commands follow the repository, not memory.
3. **Owner layer first**: scripts assemble; contracts live in backend, env,
   registry/config, runner/IPC, or algorithm owner layers.
4. **Link, do not duplicate**: English developer pages should summarize and link
   to ADRs or the development standard instead of copying the full Chinese
   standard.
5. **Config first**: backend/task/reward behavior belongs in Hydra owner YAMLs
   and registries where possible.
6. **API reference is autodoc**: `source/api_reference/` pages should contain
   autodoc/autosummary directives. Improve API prose in `src/unilab/**/*.py`
   docstrings.
7. **Use canonical commands**: user-facing examples use the top-level CLI:
   `uv run train --algo <algo> --task <task> --sim <backend>`,
   `uv run eval ...`, or `uv run demo`. Script paths such as
   `scripts/train_rsl_rl.py` may be named as implementation evidence, but they
   are not the primary command shape for docs readers.

## Before Writing

1. Decide the language root: `source/en/` or `source/zh_CN/`.
2. Locate the topic in `user_guide`, `developer_guide`, `transfer`, or `agents`.
3. Search first with `rg` / `rg --files`; update an existing page instead of
   creating a duplicate.
4. Gather evidence near the claim:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unilabsim/UniLab](https://github.com/unilabsim/UniLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
