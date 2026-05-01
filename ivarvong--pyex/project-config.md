---
trigger: always_on
description: - When the user gives feedback about how to work, update this file (AGENTS.md)
---

# Pyex Agent Guidelines

## Agent behavior
- When the user gives feedback about how to work, update this file (AGENTS.md)
  immediately so the guidance is remembered across sessions.
- Always match real Python behavior when choosing semantics.  If Pyex differs,
  fix Pyex instead of documenting or depending on the mismatch.
- Keep tests and implementation pure.  Do not introduce process messaging or
  process-coupled test probes to observe behavior that should be verified via
  returned state.
- Never worry about how hard a fix is.  Worry about making a correct and
  performant system.  If the right fix is large, do the large fix.
- Never work around Pyex limitations in fixture code or tests.  Fixtures
  exist to expose bugs.  If a fixture fails, fix Pyex.
- Known limitations go in TODO.txt, not AGENTS.md.  AGENTS.md is for how
  to build, not what's broken.

## Project
Pyex is a Python 3 interpreter written in Elixir, designed as a capabilities-based
sandbox for LLMs to safely run compute. It is the core of a PaaS where customers
write arbitrary Python -- it must be rock solid.

## Runtime
- Elixir ~> 1.19, OTP 28
- asdf for version management (`.tool-versions` in project root)
- LSP runs Elixir 1.17 and reports version mismatch errors on `mix.exs` -- these
  are false positives; ignore them.

## Style
- Write code in Jose Valim's style: clear, minimal, well-structured
- Prefer pattern matching and multi-clause functions over conditionals
- Small focused modules with clear responsibilities
- No one-line comments
- Use NimbleParsec for lexing/tokenization where it makes sense

## Architecture

### Core pipeline: Lexer -> Parser -> Interpreter

- `Pyex` -- public API: `compile/1`, `run/2`, `run!/2`, `output/1`
- `Pyex.Lexer` -- NimbleParsec-based tokenizer with indent/dedent/newline handling
- `Pyex.Parser` -- recursive descent parser producing `{node_type, meta, children}`
  AST nodes with `[line: n]` metadata
- `Pyex.Interpreter` -- tree-walking evaluator (~4600+ lines). Control flow via
  tagged tuples (`{:returned, val}`, `{:break}`, `{:continue}`, `{:exception, msg}`,
  `{:yielded, val, continuation}`). Never raise/rescue for Python semantics.

### Environment and context

- `Pyex.Env` -- scope-stack environment with global/nonlocal/put_at_source support
- `Pyex.Ctx` -- execution context: append-only event log for observability,
  filesystem handles, environ, compute timeout, `:noop` mode for compilation checks,
  custom modules, `imported_modules` cache, profile data, `generator_mode`
  (`:accumulate | :defer | nil`), `generator_acc`
- `Pyex.Error` -- structured error type with `kind` (`:syntax | :python | :timeout |
  :import | :io | :route_not_found | :internal`), `message`, `line`,
  `exception_type`. Auto-classifies from raw error strings.

### Web / Lambda

- `Pyex.Lambda` -- Lambda-style execution of FastAPI programs without a server.
  `boot/2` compiles routes, `handle/2` dispatches requests (stateful, threads ctx),
  `handle_stream/2` returns lazy `Stream` of chunks via continuations.
- `Pyex.Trace` -- custom OpenTelemetry exporter for span tree visualization

### Builtins and methods

- `Pyex.Builtins` -- built-in Python functions (len, range, print, str, int, float,
  type, abs, min, max, sum, sorted, reversed, enumerate, zip, map, filter, any, all,
  chr, ord, hex, oct, bin, pow, divmod, repr, callable, open, iter, next, getattr,
  setattr, hasattr, super, isinstance, issubclass, id, hash, vars, dir, etc.)
- `Pyex.Methods` -- method dispatch for string, list, dict, set, tuple, file_handle
  types. Resolves attribute access to bound method closures.

### Stdlib modules

- `Pyex.Stdlib` -- registry mapping Python module names to Elixir implementations.
  `module_names/0` returns all registered names.
- `Pyex.Stdlib.Module` -- behaviour that all stdlib modules implement via
  `module_value/0` returning an attribute map.
- `Pyex.Stdlib.Collections` -- Counter, defaultdict, OrderedDict
- `Pyex.Stdlib.Csv` -- reader, DictReader, writer, DictWriter
- `Pyex.Stdlib.Datetime` -- datetime.now(), date.today(), timedelta, fromisoformat
- `Pyex.Stdlib.FastAPI` -- route registration with decorators, HTMLResponse,
  JSONResponse, StreamingResponse. List-based, no ETS/Bandit.
- `Pyex.Stdlib.Html` -- html.escape(), html.unescape()
- `Pyex.Stdlib.Itertools` -- combinatoric iterators (eagerly materialized for safety)
- `Pyex.Stdlib.Jinja2` -- template engine with loops, conditionals, includes, extends
- `Pyex.Stdlib.Json` -- json.loads() / json.dumps() backed by Jason
- `Pyex.Stdlib.Markdown` -- Markdown to HTML via cmark NIF
- `Pyex.Stdlib.Math` -- trig, sqrt, pow, log, ceil, floor, pi, e via `:math`
- `Pyex.Stdlib.Random` -- randint, choice, shuffle, uniform, sample via `:rand`
- `Pyex.Stdlib.Re` -- match, search, findall, sub, split, compile via `Regex`
- `Pyex.Stdlib.Requests` -- requests.get() / requests.post() backed by Req
- `Pyex.Stdlib.Sql` -- parameterized sql.query() against PostgreSQL via Postgrex
- `Pyex.Stdlib.Time` -- time, sleep, monotonic, time_ns via `:os` / `:timer`
- `Pyex.Stdlib.Unittest` -- TestCase with assertion methods and main() discovery
- `Pyex.Stdlib.Uuid` -- uuid4() (random) and uuid7() (time-ordered)

### Filesystem backends


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivarvong/pyex](https://github.com/ivarvong/pyex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
