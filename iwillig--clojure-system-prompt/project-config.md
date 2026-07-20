---
trigger: always_on
description: This document provides instructions for maintaining consistency between the system prompt (`SYSTEM.md`) and the Anthropic/pi skill (`clojure-repl-dev/SKILL.md`).
---

# Agent Instructions: Maintaining SYSTEM.md and SKILL.md

This document provides instructions for maintaining consistency between the system prompt (`SYSTEM.md`) and the Anthropic/pi skill (`clojure-repl-dev/SKILL.md`).

## File Relationship

| File | Purpose | Format |
|------|---------|--------|
| `SYSTEM.md` | System prompt for pi agent | XML-style tags with priorities |
| `clojure-repl-dev/SKILL.md` | Anthropic/pi skill - core workflow | Markdown with frontmatter |
| `clojure-repl-dev/references/*.md` | Detailed reference material | Markdown (loaded on demand) |

**SKILL.md contains essential workflow** (~170 lines) while **references/** contains detailed documentation (~400 lines) following the [progressive disclosure principle](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).

Both `SYSTEM.md` and the skill files share the same Clojure development guidance, but formatted for their respective systems.

## Validating Clojure Code Examples

All Clojure code examples in SYSTEM.md and SKILL.md must be syntactically valid and runnable. Use `clj-nrepl-eval` to verify code before committing changes.

### Basic Validation Workflow

```bash
# 1. Start an nREPL server if not running
#    bb nrepl, lein repl :headless, or clj -M:mcp/nrepl

# 2. Test simple expressions
clj-nrepl-eval -p 7889 "(+ 1 2 3)"
# => 6

# 3. Test function definitions from your edits
clj-nrepl-eval -p 7889 "(defn greet [name] (str \"Hello, \" name))"
# => #'user/greet

clj-nrepl-eval -p 7889 "(greet \"World\")"
# => "Hello, World"

# 4. Test threading macro examples from the skills
clj-nrepl-eval -p 7889 "(-> {:count 0} (update :count inc) (assoc :active true))"
# => {:count 1, :active true}

clj-nrepl-eval -p 7889 "(->> [1 2 3 4] (filter even?) (map #(* % 2)))"
# => (4 8)

# 5. Test destructuring examples
clj-nrepl-eval -p 7889 "(let [{:keys [a b]} {:a 1 :b 2}] (+ a b))"
# => 3
```

### Validation Checklist for Code Examples

Before committing changes to Clojure code examples:

- [ ] Start nREPL: `bb nrepl` or equivalent
- [ ] Test each new function definition with `clj-nrepl-eval`
- [ ] Test each threading macro example (`->`, `->>`, `some->`, `cond->`)
- [ ] Verify destructuring syntax works
- [ ] Check that namespace examples use correct require/import syntax
- [ ] Ensure regex literals work: `#"pattern"`

### Common Issues to Catch

```bash
# Test edge cases shown in examples
clj-nrepl-eval -p 7889 "(sum-evens [])"  # empty collection handling
clj-nrepl-eval -p 7889 "(sum-evens nil)" # nil handling

# Verify metadata/accessor syntax
clj-nrepl-eval -p 7889 "(:arglists (meta #'map))"

# Test cond/case examples
clj-nrepl-eval -p 7889 "(cond (< -1 0) :negative (= 0 0) :zero :else :positive)"
```

### When clj-paren-repair is Needed

If you edit Clojure files and encounter delimiter errors:

```bash
# Fix unbalanced parentheses/brackets/braces
clj-paren-repair src/example.clj

# Or via stdin for quick testing
echo '(defn broken [x] (+ x 1)' | clj-paren-repair
```

**Never commit code that hasn't been validated in the REPL.**

## Synchronization Rules

### Rule 1: Update Both Files Together

When modifying Clojure development guidance, **ALWAYS update both files** unless the change is format-specific.

**Examples of shared content (update both):**
- REPL-first development workflow
- Threading macro patterns
- Naming conventions (kebab-case, `?` suffix, no `!` suffix)
- Tool usage instructions (`clj-nrepl-eval`, `clj-paren-repair`)
- Code quality standards (docstrings, line length, indentation)
- Validation checklists
- Research references (compiler feedback paper)

**Examples of format-specific content (update only one):**
- SYSTEM.md: XML tag structure, `priority="critical"` attributes
- SKILL.md: YAML frontmatter, relative path references

### Rule 2: Content Parity Checklist

Before considering any edit complete, verify:

- [ ] Core mandates match (REPL-first, no ! suffix, threading macros)
- [ ] Tool documentation is identical in substance
- [ ] Naming conventions are consistent
- [ ] Code quality standards align
- [ ] Research references match (arXiv paper)
- [ ] Version numbers are synchronized

### Rule 3: Version Synchronization

Both files must carry the same version number:

- `SYSTEM.md`: `<prompt-version>v1.X.X</prompt-version>`
- `SKILL.md`: `**Version:** v1.X.X` at end of file
- `README.md`: `Current version: v1.X.X`
- `CHANGELOG.md`: Version history for both files

**When bumping version:**
1. Update `CHANGELOG.md` with changes to both files
2. Update version in `SYSTEM.md`
3. Update version in `SKILL.md`
4. Update version in `README.md`

### Rule 4: Structural Mapping

| SYSTEM.md Section | SKILL.md | references/ |
|-------------------|----------|-------------|
| `<core-mandate>` | ## Core Workflow | — |
| `<clj-nrepl-eval-tool>` | ## Tools (brief) | tool-guide.md (detailed) |
| `<clj-paren-repair-tool>` | ## Tools (brief) | tool-guide.md (detailed) |
| `<idiomatic-clojure>` | ## Essential Patterns (examples) | idioms.md (complete reference) |
| `<code-quality>` | ## Naming Rules, Docstrings | idioms.md |
| `<testing>` | ## Validation Checklist | idioms.md |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iwillig/clojure-system-prompt](https://github.com/iwillig/clojure-system-prompt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
