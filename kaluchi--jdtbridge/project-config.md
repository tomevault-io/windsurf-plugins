---
trigger: always_on
description: Required tools (verified by `jdt setup --check`):
---

# JDT Bridge — Agent Instructions

## Prerequisites

Required tools (verified by `jdt setup --check`):
- **Node.js** >= 20, **npm** — CLI runtime
- **Java** >= 21, **Maven** >= 3.9 — plugin build (Tycho)
- **Eclipse IDE** — running, with JDT Bridge plugin installed
- **git**, **gh** (GitHub CLI) — version control, PRs, releases

### Eclipse source bundles (recommended)

The `@source` axis needs source bundles to read Eclipse Platform/JDT API
source and javadoc. Without them,
`jdt q '"org.eclipse.core.runtime.CoreException" | @source'` returns
"Source not available". Install all Eclipse source bundles once
(Eclipse must NOT be running):

```bash
# List available source bundles, filter out tests, install all
BUNDLES=$(D:/eclipse/eclipsec.exe -nosplash \
  -application org.eclipse.equinox.p2.director \
  -repository "https://download.eclipse.org/eclipse/updates/4.39/" \
  -list 2>&1 | grep "^org\.eclipse\..*\.source=" | grep -v "tests\.\|examples\." \
  | sed 's/=.*//' | tr '\n' ',' | sed 's/,$//')

D:/eclipse/eclipsec.exe -nosplash \
  -application org.eclipse.equinox.p2.director \
  -repository "https://download.eclipse.org/eclipse/updates/4.39/" \
  -installIU "$BUNDLES" \
  -destination "D:/eclipse" \
  -profile "epp.package.jee"
```

Adjust the update site URL to match your Eclipse version (e.g. `4.39`)
and paths to your Eclipse installation.

## Environment check (REQUIRED)

**Run at the start of every conversation, before any other work.
Subagents (Explore, Plan, etc.) skip this section — jdt is already
verified by the main agent.**

1. `jdt setup --check` — verifies CLI, Node, Java, Maven, Eclipse, bridge.
   - `jdt: command not found` → `cd cli && npm install && npm link`
   - Plugin not installed → `jdt setup --eclipse <path>`
   - Eclipse not running → start it, re-run check

2. `jdt status` — **workspace dashboard**. Shows git repos, open editors,
   compilation errors, running launches, test results, and project list
   in one call. This is your orientation command — run it first to
   understand what the developer is working on.
   - If `jdtbridge-verify` or `jdtbridge-package` are missing from
     launch configs, import them:
     ```bash
     jdt launch config --import launches/jdtbridge-verify.launch
     jdt launch config --import launches/jdtbridge-package.launch
     ```

3. Verify `jdtbridge.target` exists in repo root (gitignored, per-developer).
   Without it, Tycho builds fail. If missing, create it pointing to the
   local Eclipse directory:
   ```xml
   <?xml version="1.0" encoding="UTF-8" standalone="no"?>
   <?pde version="3.8"?>
   <target name="eclipse-local" sequenceNumber="1">
       <locations>
           <location path="/path/to/eclipse" type="Directory"/>
       </locations>
   </target>
   ```

## Use `jdt` for Java analysis

**Prefer `jdt` over grep/glob for Java-specific queries.** Grep returns
string matches; `jdt` returns semantic results from Eclipse's compiler index.

`jdt status` shows the full workspace state — including a `help` section
with the full command reference. `jdt help <command>` for detailed usage
of any command.

### `jdt q` — graph query language

`jdt q '<qlang-pipeline>'` evaluates a qlang pipeline against the
Eclipse semantic graph. The graph is navigated through operand axes
that consume a node-Map (skeleton or detail) OR a fqn string
from `pipeValue`. Output is qlang-literal; fqn strings in the result
are ready subjects for the next step.

Full axis catalog, sugar conduits, and modifier syntax live in
`jdt help q` (runtime truth) and `docs/jdt-query-spec.md`.

### FQN — member form

Every node carries one identifier field, `:fqn`. Per kind:

```
pkg.Class                     type
pkg.Class#fieldName           field or unambiguous method by name
pkg.Class#method()            zero-arg method
pkg.Class#method(String)      method with erased parameter signature
pkg.Class.method(String)      Eclipse Copy Qualified Name — also accepted
```

A bare `pkg.Class#method` without parens is ambiguous when the
type has multiple overloads; the error descriptor carries
`:candidates` so the caller can pick and retry. Types can be
simple (`String`) or qualified (`java.lang.String`); generics
are stripped — `List<String>` matches `List`.

### Pipeline composability

All of the below return a single qlang-literal value — ready for
pipe-chaining with standard shell tools or further `jdt q` calls.

```bash
jdt q '"io.github.kaluchi.jdtbridge.SearchHandler" | @members * /name'
jdt q '"io.github.kaluchi.jdtbridge.JdtUtils#findMethod" | @callers | count'
jdt q '"my-server" | @problems * /message'
jdt q '"org.springframework.jdbc.core.JdbcTemplate#query" | @source' | grep -n throw
jdt q '"io.github.kaluchi.jdtbridge.HttpServer" | @methods | filter(@untested) * /fqn'
```

### Subagents (Explore, Plan)

Custom jdt-aware agents are installed in `.claude/agents/` by
`jdt setup --claude`. They have `jdt` commands in their system prompt
and auto-allow hooks — no setup needed.

**When to use subagents:**
- `Explore` — finding types, tracing references, understanding
  hierarchies, reading source. Fast (haiku model). Use for research
  that doesn't require writing code.
- `Plan` — designing implementation strategy with code understanding.
  Inherits parent model. Read-only.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaluchi/jdtbridge](https://github.com/kaluchi/jdtbridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
