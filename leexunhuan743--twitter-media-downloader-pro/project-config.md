---
trigger: always_on
description: - Run `lat search` to find sections relevant to your task. Read them to understand the design intent before writing code.
---

# Before starting work

- Run `lat search` to find sections relevant to your task. Read them to understand the design intent before writing code.
- Run `lat expand` on user prompts to expand any `[[refs]]` — this resolves section names to file locations and provides context.

# Post-task checklist (REQUIRED — do not skip)

After EVERY task, before responding to the user:

- [ ] Update `lat.md/` if you added or changed any functionality, architecture, tests, or behavior
- [ ] Run `lat check` — all wiki links and code refs must pass
- [ ] Do not skip these steps. Do not consider your task done until both are complete.

---

# What is lat.md?

This project uses [lat.md](https://www.npmjs.com/package/lat.md) to maintain a structured knowledge graph of its architecture, design decisions, and test specs in the `lat.md/` directory. It is a set of cross-linked markdown files that describe **what** this project does and **why** — the domain concepts, key design decisions, business logic, and test specifications. Use it to ground your work in the actual architecture rather than guessing.

# Commands

```bash
lat init                     # initialize lat.md/ in a new project
lat locate "Section Name"    # find a section by name (exact, fuzzy)
lat section "file#Section"   # show a section with content, outgoing refs, and incoming refs
lat refs "file#Section"      # find references; add --scope=md|code|md+code to narrow
lat search "natural language" # semantic search; use --limit N to tune result count
lat search                   # build/update the embedding index without querying
lat expand "user prompt text" # expand [[refs]] to resolved locations; use --stdin for pipes
lat reindex                  # rebuild embeddings; use --local or --remote to switch backend
lat config                   # show the lat config file path
lat mcp                      # start MCP server (stdio) for AI agent tool access
lat check                    # validate all; subcommands: md, code-refs, index, sections
```

Run `lat --help` for all options, `lat <command> --help` per command.

### Command behavior notes

- Prefer the installed `lat --help`, upstream `lat.md/cli.md`, and source code over generated templates when they conflict; templates can lag current implementation.
- `lat check` without a subcommand runs `md`, `code-refs`, `index`, and `sections`.
- `lat refs --scope=md` checks markdown wiki links, `--scope=code` checks `@lat` comments, and `--scope=md+code` checks both.
- `lat locate` and `lat expand` are exploratory and may use fuzzy matching. Do not treat a fuzzy result as a valid link until `lat check` accepts it.

### MCP (Model Context Protocol)

`lat mcp` starts an MCP server (stdio transport) with 6 tools: `lat_locate`, `lat_section`, `lat_search`, `lat_expand`, `lat_check`, `lat_refs`. Configure it in your agent's MCP settings to avoid manual CLI fallback. See the [upstream CLI docs](https://github.com/1st1/lat.md/blob/main/lat.md/cli.md).

### Semantic search and embeddings

According to the upstream `lat.md/cli.md` design notes and current source, `lat search` works offline by default using a bundled local embedding model. Do not tell the user that an API key is required just because semantic search is involved.

Hosted embeddings are optional. For higher-quality remote embeddings, configure an OpenAI (`sk-...`) or Vercel AI Gateway (`vck_...`) key via `LAT_LLM_KEY`, `LAT_LLM_KEY_FILE`, `LAT_LLM_KEY_HELPER`, or the config file shown by `lat config`. Use `lat reindex --local` to force the offline model or `lat reindex --remote` to use the hosted backend.

Normal `lat search` creates or updates the generated index at `lat.md/.cache/vectors.db`; `lat reindex` is the explicit full rebuild and backend-switch command. Once an index records a model, that model is authoritative: if the current environment cannot serve it, fix the key or run `lat reindex --local` / `lat reindex --remote` instead of silently changing backends.

If `lat search` fails, report the actual error and fall back to `lat locate`, `lat section`, and direct file reads rather than guessing.

---

# Quickstart: Add a new section

1. Create or edit a file in `lat.md/` (e.g. `lat.md/feature-x.md`)
2. Add a `# Title` and a one-paragraph description (≤250 chars)
3. Link to it from the index `lat.md/lat.md` with `- [[feature-x]] — description`
4. Cross-link from related sections with `[[feature-x#SectionName]]`
5. Run `lat check` — fix any broken links

# CI Integration

`lat check` should be part of CI to prevent drift. Current `.github/workflows/go.yml` only builds release binaries — add a `lat check` step if you enable CI for PRs.

# Syntax primer

- **Section ids**: `lat.md/path/to/file#Heading#SubHeading` — full form uses project-root-relative path (e.g. `lat.md/tests/search#RAG Replay Tests`). Short form uses bare file name when unique (e.g. `search#RAG Replay Tests`, `cli#search#Indexing`).
- **Wiki links**: `[[target]]` or `[[target|alias]]` — cross-references between sections. `[[foo]]` links to file `foo.md`; it does not search headings. `[[foo#Bar#Baz]]` must include the exact heading chain. Local-only heading links such as `[[#Bar]]` are invalid.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Leexunhuan743/twitter_media_downloader_pro](https://github.com/Leexunhuan743/twitter_media_downloader_pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
