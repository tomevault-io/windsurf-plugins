---
trigger: always_on
description: Teaches a model how to write BareScript — language syntax, the built-in library, the include library, MarkdownUp client-rendered applications, and unit tests. Use this skill whenever the user asks you to write, read, modify, or review BareScript (`.bare`) code, a MarkdownUp Markdown application, a `markdown-script` code block, or any code that uses BareScript built-in functions (`array*`, `object*`, `string*`, `data*`, `markdownPrint`, `elementModelRender`, `drawNew`, etc.). Also use it when ask
---


# BareScript Skill

This file teaches a language model how to write correct, idiomatic
[BareScript](https://craigahobbs.github.io/bare-script/language/). It is plain
Markdown with concrete examples — any frontier model (Claude, GPT, Gemini,
Llama, …) can read it.

## When to use this skill

Use it whenever you write, modify, review, or debug BareScript — `.bare` files,
`markdown-script` fenced blocks, MarkdownUp apps, or their unit tests. Recognize
BareScript even when it isn't named: the `.bare` extension, `markdown-script`
code fences, function names like `arrayLength` / `objectGet` / `markdownPrint` /
`systemFetch`, or the `function:` … `endfunction` / `if:` … `endif` /
`for … in …:` … `endfor` block syntax.

## What BareScript is (one paragraph)

BareScript is a small, embeddable scripting language with Python-flavored
syntax. It runs in two implementations (JavaScript and Python) that share
100% of their test suite. There are no classes, no modules, no exceptions, no
imports — just functions, a literal-only data model (number, string, boolean,
null, array, object, datetime, regex, function), and ~200 library functions.
Long-running scripts can be executed in a synchronous interpreter; scripts
that do I/O (`systemFetch`, includes that fetch URLs) run in the async
interpreter and use `async function` declarations.

---

## 1. Language syntax (must-know)

### Statements end at newline

There are no semicolons. To continue a long statement onto the next line, end
the previous line with a backslash `\`:

```barescript
colors = [ \
    'red', \
    'green', \
    'blue' \
]
```

### Comments

`#` to end of line. **There are no block comments.**

```barescript
# This is a comment
x = 1  # Trailing comments are also fine
```

### Values

- `null`, `true`, `false` (special variables; cannot be reassigned)
- Numbers: integer, float, scientific (`1.5e10`), hex (`0xFF`)
- Strings: `'single'` or `"double"` — concatenate with `+`
- Arrays: `[1, 2, 3]`
- Objects: `{'name': 'Alice', 'age': 30}`
- Datetimes: produced by `datetimeNew(...)`, `datetimeISOParse(...)`, etc.
- Regex: produced by `regexNew('[0-9]+')`
- Functions are first-class values

### Variables and assignment

```barescript
x = 5
name = 'Alice'
items = [1, 2, 3]
```

Top-level assignments create globals. Assignments inside a `function` create
locals. Variable names with non-alphanumeric characters must be wrapped in
brackets: `[Height (ft)]`.

### **Critical: there is no bracket access for arrays/objects**

`obj['key']` and `arr[0]` are **NOT valid BareScript**. (Brackets are only used
for array *literals* and for variable names with special characters.) Always
use the library functions:

| You want…                 | Use this                          |
| ------------------------- | --------------------------------- |
| `arr[i]`                  | `arrayGet(arr, i)`                |
| `arr[i] = v`              | `arraySet(arr, i, v)`             |
| `obj['k']`                | `objectGet(obj, 'k')`             |
| `obj['k'] = v`            | `objectSet(obj, 'k', v)`          |
| `obj['k']` with default   | `objectGet(obj, 'k', defaultVal)` |
| check key existence       | `objectHas(obj, 'k')`             |
| array length              | `arrayLength(arr)`                |
| object keys               | `objectKeys(obj)`                 |

This is the single most common mistake when models write BareScript. **If you
catch yourself typing `[`, stop and check whether you mean a literal.**

### Control flow

Every block-opening statement ends with `:` and every block has an explicit
`endif` / `endwhile` / `endfor` / `endfunction`:

```barescript
if x < 0:
    y = -1
elif x > 0:
    y = 1
else:
    y = 0
endif

i = 0
while i < 10:
    i = i + 1
endwhile

for value in items:
    systemLog(value)
endfor

# With index
for value, ixValue in items:
    systemLog(ixValue + ': ' + value)
endfor

break       # exits the nearest while/for
continue    # next iteration of the nearest while/for
```

`jump label` / `jumpif (expr) label` / `label:` exist for hot loops but are
rarely used outside the standard library — prefer `while`/`for`.

### Functions

```barescript
function double(n):
    return n * 2
endfunction

# Async required if the body calls any async function (e.g., systemFetch,
# included libraries that fetch URLs). The closer is always `endfunction` —
# there is no separate `endasync` keyword.
async function fetchJson(url):
    return jsonParse(systemFetch(url))
endfunction
```

A function that calls **any** async function must itself be declared `async`.
Sync code cannot call async code.

`return` without an expression returns `null`. A function with no `return`
also returns `null`.

### The built-in `if(test, a, b)` expression function

This is special: only the chosen branch is evaluated.

```barescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craigahobbs/bare-script-py](https://github.com/craigahobbs/bare-script-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
