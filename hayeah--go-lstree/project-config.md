---
trigger: always_on
description: Go library and CLIs for listing directory trees with glob filtering, sorting, pagination, and gitignore support. Use when you need to walk/query/serve filesystem trees over fs.FS or HTTP.
---


# go-lstree

Go library and CLIs for listing directory trees with glob filtering, sorting, pagination, and gitignore support. Works with any `fs.FS` implementation.

Three layers:

- **`lstree` library** — reusable tree walker with callback, iterator, and query interfaces
- **`lstree` CLI** — command-line directory listing
- **`lstree-serve` CLI** — HTTP server with static files + `__tree__` metadata endpoint

## Install

```bash
# Install CLIs via gobin (editable, rebuilds from source)
gobin install github.com/hayeah/go-lstree/cli/lstree
gobin install github.com/hayeah/go-lstree/cli/lstree-serve

# Or via go install
go install github.com/hayeah/go-lstree/cli/lstree@latest
go install github.com/hayeah/go-lstree/cli/lstree-serve@latest
```

## `lstree` CLI

```
lstree [flags] [root]

Flags:
  -depth int      Max recursion depth (1 = flat, 0 = unlimited)
  -glob value     Glob filter (repeatable, ! prefix to exclude)
  -sort string    Sort by: path (default), modified, size, name
  -desc           Sort descending
  -limit int      Max entries (0 = unlimited)
  -offset int     Skip first N entries
  -json           Output as JSON
  -long           Long format: size, modified, path
  -no-ignore      Disable ignore rules
```

### Examples

```bash
# List all Go files recursively
lstree --glob '**/*.go' ~/myproject

# All EPUBs, newest first, with metadata
lstree --glob '**/*.epub' --sort modified --desc --long ~/Dropbox/epubs

# Flat listing (like ls)
lstree --depth 1 ~/Dropbox/epubs

# EPUBs excluding drafts, as JSON
lstree --glob '**/*.epub' --glob '!**/draft*' --json ~/Dropbox/epubs

# Everything, ignoring .gitignore rules
lstree --no-ignore ~/myproject
```

### Output Formats

**Default** — one path per line:
```
alice.epub
sci-fi/dune.epub
sci-fi/neuromancer.epub
```

**Long** (`--long`) — size, modified, path:
```
  245760  2026-03-15T10:30:00Z  alice.epub
  512000  2026-02-01T08:00:00Z  sci-fi/dune.epub
```

**JSON** (`--json`):
```json
{
  "entries": [
    {"path": "alice.epub", "isDir": false, "size": 245760, "modified": "2026-03-15T10:30:00Z"}
  ],
  "total": 1,
  "offset": 0,
  "limit": 0
}
```

## `lstree-serve` CLI

HTTP server: static file serving + `__tree__` metadata endpoint.

```bash
lstree-serve ~/Dropbox/epubs --bind 0.0.0.0:8080
```

### Static Files

Normal paths serve files directly:

```
GET /alice.epub          → serves <root>/alice.epub
GET /sci-fi/dune.epub    → serves <root>/sci-fi/dune.epub
```

### `__tree__` Endpoint

Path prefix before `__tree__` scopes to a subdirectory.

```
GET /__tree__
GET /sci-fi/__tree__
```

Query params:

| Param | Type | Default | Description |
|---|---|---|---|
| `depth` | int | unlimited | Max recursion depth. `1` = flat listing |
| `glob` | string | — | Glob pattern. Repeatable. `!` prefix to exclude |
| `sort` | string | `path` | Sort: `path`, `modified`, `size`, `name` |
| `order` | string | `asc` | Direction: `asc`, `desc` |
| `limit` | int | unlimited | Max entries returned |
| `offset` | int | `0` | Skip first N entries |
| `stat` | bool | `false` | Include size/modified (`true` or `1`) |
| `no_ignore` | bool | `false` | Disable ignore rules (`true` or `1`) |

```bash
# All EPUBs, newest first, with metadata
curl 'http://localhost:8080/__tree__?glob=**/*.epub&sort=modified&order=desc&stat=true'

# Flat listing
curl 'http://localhost:8080/__tree__?depth=1'

# Scoped to subdirectory, paginated
curl 'http://localhost:8080/sci-fi/__tree__?glob=**/*.epub&limit=10&offset=0'
```

Response:

```json
{
  "base": "",
  "entries": [
    {"path": "alice.epub", "isDir": false, "size": 245760, "modified": "2026-03-15T10:30:00Z"}
  ],
  "total": 3,
  "offset": 0,
  "limit": 0
}
```

## Library Usage

```go
import "github.com/hayeah/go-lstree"
```

### List — Full Query Pipeline

```go
fsys := os.DirFS("/path/to/files")
ignorer, _ := lstree.NewIgnorer("/path/to/files")

result, err := lstree.List(fsys, lstree.Query{
    Globs:      []string{"**/*.epub"},
    Sort:       "modified",
    Descending: true,
    Stat:       true,
    Limit:      50,
}, ignorer)

for _, e := range result.Entries {
    fmt.Printf("%s (%d bytes)\n", e.Path, e.Size)
}
```

### Walk — Callback Interface

```go
// Low-level walk with per-entry callback
// Applies depth, glob, ignore filters during traversal
lstree.Walk(fsys, lstree.Query{
    MaxDepth: 2,
    Globs:    []string{"**/*.go"},
}, ignorer, func(e lstree.Entry) error {
    fmt.Println(e.Path)
    return nil
})
```

### Iter — Iterator Interface

```go
for entry, err := range lstree.Iter(fsys, lstree.Query{}, ignorer) {
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(entry.Path)
}
```

### HTTP Handler — Mount on Existing Server

```go
// Standalone
http.ListenAndServe(":8080", lstree.NewHTTPHandler(os.DirFS(".")))

// Mounted at a prefix
mux := http.NewServeMux()
mux.Handle("/files/", http.StripPrefix("/files", lstree.NewHTTPHandler(os.DirFS("."))))

// With fstest.MapFS for tests
handler := lstree.NewHTTPHandler(fstest.MapFS{
    "alice.epub": &fstest.MapFile{Data: []byte("..."), ModTime: time.Now()},
})
```

## Glob Patterns

Standard glob syntax extended with `**`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hayeah/go-lstree](https://github.com/hayeah/go-lstree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
