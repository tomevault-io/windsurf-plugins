---
trigger: always_on
description: Maintainer guide for okf-gem — `okf` on RubyGems. The gem reads, validates,
---

# AGENTS.md

Maintainer guide for okf-gem — `okf` on RubyGems. The gem reads, validates,
lints, and serves Open Knowledge Format (OKF) v0.1 bundles: directories of
Markdown + YAML frontmatter that humans and agents both read. The bundled skill
(`lib/okf/skill/`) documents the format itself; this file documents how to
change the code without breaking its contracts.

## Map

```
lib/okf/
  path.rb                 pure   path normalization + root-escape guard
  markdown/               pure   format layer: frontmatter (§4), links (§5), citations (§8)
  concept.rb  bundle.rb   pure   the in-memory model (no disk, no stdio)
  bundle/graph.rb         pure   nodes/edges + type/tag indexes
  bundle/validator*.rb    pure   spec §9 conformance (hard errors + soft warnings)
  bundle/linter*.rb       pure   curation-quality report (never rejects)
  concept/file.rb         shell  one-file on-disk handle
  bundle/reader|writer|folder.rb  shell  directory <-> Bundle (writer is atomic, validates before publish)
  server/app.rb           shell  Rack app: / (page), /node, /node/meta, /catalog, /tags, /types, /index, /log
  server/graph.rb + graph/template.html.erb  the whole UI in one self-contained ERB file
  server/runner.rb        shell  built-in WEBrick <-> Rack bridge (replaces any rackup need)
  skill.rb + skill/       shell  the companion agent skill + its installer
  cli.rb                  shell  the only layer that parses argv, prints, and exits
```

The core/shell split is _enforced_: `test/unit/boundary_test.rb` fails if a pure
file names a shell class or touches `File`/`Dir`/`FileUtils`/stdio. Put new I/O
in the shell; put new logic in the core, pure.

Outside `lib/`, `plugin/` and `.claude-plugin/` are the Claude Code plugin and
its marketplace manifest (the repo doubles as the marketplace): one thin command
that routes to the skill's playbooks (`lib/okf/skill/playbooks/`) or to the
skill itself, a PostToolUse curation hook (`plugin/hooks/scripts/curate.rb`,
plain Ruby on the stdlib, same 2.4 floor), and a generated copy of the skill.
Neither ships in the gem (gemspec reject).

`require "okf"` loads the library only — the model, the analyzers, and the
on-disk handles. The two argv-facing shells, `cli.rb` (and its `optparse`) and
`skill.rb`, load on demand: `exe/okf` requires them, and so must any test that
drives them. An embedding app never pays for the command-line machinery.
`test/unit/loading_test.rb` guards this in a clean subprocess; keep it green when
you touch what `require "okf"` pulls in.

## Hard constraints

1. **Ruby >= 2.4** (rack's own floor — the point is running on the Ruby an OS
   already ships). RuboCop parses at 2.4 and catches syntax, but **not APIs**.
   Do not introduce: `delete_prefix`/`delete_suffix`, `transform_keys`,
   `Dir.children`, `Dir.glob(base:)`, `Struct.new(keyword_init:)`,
   `yield_self` (2.5); `to_h { }`, `then`, `rescue`/`ensure` directly inside a
   `do…end` block, endless string slices `str[i..]`, `YAML.safe_load` keyword
   args outside the Frontmatter shim (2.6); `filter_map`, `tally`, numbered
   block params (2.7); endless methods, hash shorthand (3.x).
   The truth test: `docker run --rm -v "$PWD":/app -w /app ruby:2.4 bash -c
"bundle install && bundle exec rake test"`.
2. **Runtime dependencies are exactly `rack` and `webrick`.** No ActiveSupport —
   `OKF.blank?` and `Markdown::Frontmatter.stringify_keys` exist precisely so it
   is not needed. A new runtime dependency is a design decision, not a
   convenience; challenge it.
3. **YAML only through `Markdown::Frontmatter`** — `safe_load`, `Date`/`Time`
   permitted, no aliases. The Psych <3.1 positional-argument shim lives there;
   do not call `YAML.safe_load`/`YAML.load` anywhere else.
4. **`validate` and `lint` stay separate.** §9 forbids the validator from
   rejecting broken cross-links or missing optional fields (warnings only);
   lint owns curation findings and never emits conformance errors. New checks go
   to the right side, and exit codes keep the contract: 0 ok, 1 failing bundle,
   2 usage error.
5. **The server page stays self-contained**: one ERB template, inline CSS/JS,
   only Cytoscape, marked, and DOMPurify from a CDN at boot (Mermaid, Panzoom,
   and the extra layout engines lazy-load from the same CDN on first use),
   bodies pulled on demand with `fetch()`. No htmx, no bundler, no build step. Two XSS defenses hold the
   line: inlined data goes through `json_for_script` (escapes `<` so it cannot
   break out of its `<script>`), and every fetched body is run through
   `DOMPurify.sanitize(marked.parse(...))` before it reaches `innerHTML`. Keep
   both — a new render path that skips the sanitizer reopens the hole.
6. **The skill ships only from `lib/okf/skill/**`** — that tree is the single
canonical copy (`okf skill <dest>`installs from it), so edit it there and
nowhere else. Local installs (e.g.`.agents/`, `.claude/`) are gitignored.
`plugin/skills/okf`is a *generated* copy for the Claude Code plugin, so
never edit it there: run`bundle exec rake plugin:sync`after touching the
skill or bumping the version (the task also stamps`plugin/.claude-plugin/plugin.json`), and `test/plugin/sync_test.rb`fails on any drift (file lists and SHA-256 checksums). Signature guidance

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serradura/okf-gem](https://github.com/serradura/okf-gem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
