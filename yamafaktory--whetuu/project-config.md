---
trigger: always_on
description: An opinionated, zero-config, async status line and history picker
---

# whetū

An opinionated, zero-config, async status line and history picker
(fish/bash/zsh) in Zig 0.17.
The binary is installed as the ASCII command `whetuu` (whetū is Māori for "star").

The status line format and module set are hardcoded — there is intentionally no
config file. A single compiled binary renders the whole line, spawning its slowest
module as a task via `std.Io` (`Io.async` → `Future`, backed by `Io.Threaded`) and
running the rest alongside it. A thread costs more than most of these modules do,
so only the one worth overlapping gets one.

## Working approach

- Before writing any code, identify unclear or ambiguous requirements and ask about them. The goal is a complete picture of the task before implementation begins.
- When adding or changing code, look for opportunities to extract reusable helpers and avoid duplication. Shared logic belongs in a single place (e.g. escape-wrapping lives only in `style.zig`).
- When fixing a bug, add a test that would have caught it to prevent regression.

## Performance and security come first

These two outrank every other preference in this file. A status line runs before
every command you type, and it reads your repositories, your history and your
environment.

- **Do not allocate on a path that runs per render.** Prefer a buffer on the
  stack and a function that writes into it (`std.fmt.bufPrint`,
  `Io.Dir.readFile`) over an allocator and a function that hands back new memory
  (`std.fmt.allocPrint`, `readFileAlloc`). An arena that has to grow pays an
  `mmap` for it, and the usual answer a module gives is that it has nothing to
  show, which should cost nothing at all. Take an allocator where the size is
  genuinely unbounded, and let the signature say so.
- **Prove a performance claim before you make it.** `hyperfine` pinned to one
  core, several hundred runs, and the pair run in both orders, because the
  machine drifts under you. Add a control that isolates the change: a command
  that shares the startup but not the new work says whether the cost is where
  you think it is. Page faults and system time say the same thing more directly.
  A difference inside the noise is not a difference.
- **Every byte from outside goes through `sanitize`** before it reaches the
  terminal. A branch name, a directory, a command line, a changelog entry
  downloaded a second ago. Control bytes repaint screens and move cursors.
- **A network request is a decision, not a detail.** The README's security
  section lists every host whetuu can reach and what sends the request. A change
  that adds a host, or makes an existing request happen more often, updates that
  list in the same commit or it does not land.
- **Verify anything downloaded before it reaches the disk as a binary**, and put
  it in place with a rename rather than a write. A partial write is a whetuu the
  shell then runs.

## Zig Style

Follow Zig master's own conventions: the official Style Guide in the language
reference plus the practices observed in current `std`. Where the two conflict
with personal habit, Zig wins.

### Naming

- camelCase for functions and methods; TitleCase for types and for functions
  that return a `type`; snake_case for everything else (variables, parameters,
  constants, namespaces).
- Acronyms and initialisms follow the same casing rules as ordinary words
  (`readU32Be`, `XmlParser`), even two-letter ones.
- Avoid filler words in type names: `Value`, `Data`, `Context`, `Manager`,
  `State`, `utils`, `misc`. Everything is a value and all logic manages state —
  such words communicate nothing.
- Choose names based on the fully-qualified namespace and avoid redundant
  segments (`json.Value`, not `json.JsonValue`).
- No underscore prefixes. Prefer verbose names at outer scopes and abbreviated
  names at inner scopes.
- Method receivers are short names derived from the type (`w: *Writer`,
  `env: Env`, `list: *DoublyLinkedList`) — never `self`.
- File names: a file that is a struct with top-level fields is `TitleCase.zig`;
  a namespace file is `snake_case.zig`. Directories are snake_case.
- prefer `const foo: Type = .{ .field = value };` and decl literals
  (`.empty`, `.init`) over `const foo = Type{ … };`
- pass allocators explicitly; use `errdefer` for cleanup on error
- when an import property is referenced more than once in a file (e.g.
  `std.os.linux.errno`), introduce a file-scope or local `const` alias and use
  it throughout instead of repeating the dotted path
- use underscores as digit separators in integer literals with 4 or more digits
  (e.g. `1_000`, `2_000`)

### Control flow

- Use early return (or early `continue` inside loops) to guard against the non-primary case and keep the main path at the lowest nesting level. Prefer `if (!condition) return;` over `if (condition) { … }` when the body is the rest of the function or loop iteration. The same applies to `if/else`: when one branch is short and the other is the main path, put the short case first with an exit so the main body is un-nested. When `return`/`continue` are not available mid-function, use a Zig labeled block (`label: { if (guard) break :label; … }`).
- Expand long `if/else if` chains to block form rather than one-liners.

### Layout


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yamafaktory/whetuu](https://github.com/yamafaktory/whetuu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
