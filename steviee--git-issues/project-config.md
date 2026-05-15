---
trigger: always_on
description: Build a CLI tool called `git-issues` (binary: `issues`) that provides a git-native issue tracker. Issues are stored as Markdown files inside the repository itself under `.issues/`, making them version-controlled alongside the code they describe.
---

# git-issues – Implementation Brief for Claude Code

## Project Goal

Build a CLI tool called `git-issues` (binary: `issues`) that provides a git-native issue tracker. Issues are stored as Markdown files inside the repository itself under `.issues/`, making them version-controlled alongside the code they describe.

---

## Tech Stack

- **Language:** Go 1.22+
- **Dependencies (minimal):**
  - `gopkg.in/yaml.v3` – Frontmatter parsing
  - `github.com/spf13/cobra` – CLI framework
  - No database, no network, no auth
- **Binary name:** `issues`
- **Install target:** `/usr/local/bin/issues`

---

## Repository Structure

```
git-issues/
  cmd/
    root.go
    new.go
    list.go
    show.go
    edit.go
    close.go
    reopen.go
    set.go
    relate.go
    unrelate.go
    labels.go
    graph.go
  internal/
    issue/
      issue.go       ← Issue struct + Frontmatter schema
      store.go       ← Read/write .issues/ directory
      relations.go   ← Bidirectional relation sync
      id.go          ← ID generation
      slug.go        ← Title → filename slug
    git/
      stage.go       ← git add wrapper
    config/
      config.go      ← .issues/.config.yml
  main.go
  go.mod
  go.sum
  README.md
  .issues/
    .agent.md        ← generated on `issues init`
    .config.yml      ← generated on `issues init`
```

---

## Data Model

### Issue Struct

```go
type Issue struct {
    // Frontmatter
    ID       int       `yaml:"id"`
    Title    string    `yaml:"title"`
    Status   string    `yaml:"status"`   // open | in-progress | closed | wontfix
    Priority string    `yaml:"priority"` // low | medium | high | critical
    Labels   []string  `yaml:"labels"`
    Relations Relations `yaml:"relations,omitempty"`
    Created  string    `yaml:"created"`  // ISO date YYYY-MM-DD
    Updated  string    `yaml:"updated"`
    Closed   string    `yaml:"closed,omitempty"`

    // Body (everything after frontmatter delimiter)
    Body string `yaml:"-"`

    // Derived (not stored)
    FilePath string `yaml:"-"`
}

type Relations struct {
    Blocks    []int `yaml:"blocks,omitempty"`
    DependsOn []int `yaml:"depends-on,omitempty"`
    RelatedTo []int `yaml:"related-to,omitempty"`
    Duplicates []int `yaml:"duplicates,omitempty"`
}
```

### File Format

```
---
id: 7
title: "Login schlägt bei leeren Passwörtern fehl"
status: open
priority: high
labels: [bug, auth]
relations:
  blocks: [12, 15]
  depends-on: [3]
created: 2026-03-04
updated: 2026-03-04
---

Freitext Markdown body hier.

## Notes
Weitere Notizen als freie Sections.
```

**Critical:** The frontmatter delimiter is exactly `---` on its own line. The file always starts with `---`, contains YAML, then `---`, then the body. If no body, a newline after the closing `---` is sufficient.

### Filename Convention

```
{id:04d}-{slug}.md
```

Slug generation:
1. Lowercase the title
2. Replace umlauts: ä→ae, ö→oe, ü→ue, ß→ss
3. Remove all characters except `[a-z0-9 -]`
4. Replace spaces with `-`
5. Truncate to 40 characters
6. Strip trailing `-`

Examples:
- `"Login schlägt bei leeren Passwörtern fehl"` → `0007-login-schlaegt-bei-leeren-passwoertern`
- `"Fix auth bug"` → `0001-fix-auth-bug`

**The filename never changes after creation**, even if the title is edited.

### ID Generation

Read all existing `.issues/*.md` files, parse their `id` frontmatter field, return `max(ids) + 1`. Start at 1 if no issues exist. IDs are never reused, never derived from filename.

---

## Store Layer (`internal/issue/store.go`)

```go
// Must implement:
func LoadAll(issuesDir string) ([]*Issue, error)
func LoadByID(issuesDir string, id int) (*Issue, error)
func Save(issuesDir string, issue *Issue) error   // writes file, updates `updated` field
func Delete(issuesDir string, id int) error
func NextID(issuesDir string) (int, error)
func IssuesDir() (string, error)  // finds .issues/ by walking up from cwd
```

`IssuesDir()` behavior: walk up from current working directory until finding `.issues/` or hitting filesystem root. Error if not found. This mirrors how `git` finds `.git/`.

`Save()` must:
1. Set `updated` to today's date
2. Marshal frontmatter as YAML
3. Write `---\n{yaml}---\n{body}\n`
4. If `auto_stage: true` in config, call `git add {filepath}`

---

## Config (`internal/config/config.go`)

File: `.issues/.config.yml`

```yaml
default_priority: medium
auto_stage: true
labels:
  - bug
  - feature
  - auth
  - security
  - docs
```

```go
type Config struct {
    DefaultPriority string   `yaml:"default_priority"`
    AutoStage       bool     `yaml:"auto_stage"`
    Labels          []string `yaml:"labels"`
}

func Load(issuesDir string) (*Config, error)
func Default() *Config  // returns sensible defaults if no config file exists
```

Default values: `default_priority: medium`, `auto_stage: true`.

---

## Git Integration (`internal/git/stage.go`)

```go
func Stage(filepath string) error {
    cmd := exec.Command("git", "add", filepath)
    cmd.Dir = filepath.Dir(filepath)
    return cmd.Run()
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [steviee/git-issues](https://github.com/steviee/git-issues) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
