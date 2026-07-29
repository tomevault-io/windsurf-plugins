---
trigger: always_on
description: **Analysis Date:** 2026-05-07
---

# Coding Conventions

**Analysis Date:** 2026-05-07

## Naming Patterns

**Packages:**
- All lowercase single-word names, no underscores: `connect`, `cache`, `filter`, `source`, `content`, `multisource`, `localgit`, `namedlocks`, `shake256`, `cachetype`
- Sub-packages under `internal/providers/` group by provider type: `github`, `bitbucket`, `localgit`, `cache`, `filter`, `content`, `multisource`, `source`
- Config types live under `cmd/easyp/internal/config/` and `cmd/easyp/internal/config/cachetype/`

**Files:**
- One primary type/concern per file, named after the concern: `api.go`, `blobs.go`, `bynames.go`, `modulepins.go`, `repos.go`, `client.go`, `getfiles.go`, `getrepo.go`
- File names are all lowercase, no underscores (except `_test.go` suffix which is not currently used)
- A `filter.go` file in each provider package holds the `Repo` filtering struct and logic

**Functions:**
- Exported constructor functions use `New` or `NewMultiRepo` pattern: `connect.New()`, `multisource.New()`, `github.NewMultiRepo()`, `bitbucket.NewMultiRepo()`, `localgit.New()`, `namedlocks.New()`, `artifactory.New()`
- Unexported helper functions use camelCase: `getRepo`, `getFiles`, `getFile`, `splitRepoName`, `filterEntries`, `buildQuery`, `tmplBuild`, `tmplExec`
- Exported functions are PascalCase: `Find`, `Check`, `Get`, `Put`, `Hash`, `New`
- One-liner methods on a single line: `func (r sourceRepo) Name() string { return "github proxy" }` (see `internal/providers/github/repos.go:72-75`)

**Variables:**
- Private struct fields are camelCase: `log`, `repo`, `domain`, `basePath`, `byName`
- Sentinel errors use `Err` prefix: `ErrNotFound`, `ErrEmpty`, `ErrUnexpected`, `ErrInvalidType`
- Constants are PascalCase (exported) or camelCase (unexported): `ProtoSuffix`, `digestFormat`, `MaxRedirects`, `minNumberOfRepos`, `connectionTimeout`, `accessCheckPeriod`, `filesListUnlimited`

**Types:**
- Struct types are PascalCase: `Repo`, `Meta`, `File`, `Hash`, `Config`, `Cache`
- Unexported implementation structs are camelCase: `api`, `client`, `store`, `multiRepo`, `sourceRepo`, `namedLocks`, `artifactory`
- Type aliases for domain concepts: `type User string`, `type Password string`, `type Hash [64]byte`, `type Type string`
- Interface names are typically one-word or short: `Source`, `Provider`, `Cache`, `Git`, `Repositories`
- Unexported interfaces for internal decoupling: `provider` in `internal/connect/api.go`, `namedLocks` in `internal/providers/localgit/localgit.go`

## Code Style

**Formatting:**
- Tool: `gofmt` / `goimports` enforced via `golangci-lint`
- Import ordering enforced by `gci` linter (see `.golangci.yml:73-82`):
  1. Standard library (`standard`)
  2. Third-party / default imports (`default`)
  3. Project imports (`prefix(github.com/easyp-tech)`)
- Blank lines separate import groups (standard, third-party, project)

**Linting:**
- Tool: `golangci-lint` with `enable-all: true` (`.golangci.yml`)
- Disabled deprecated linters: `exhaustivestruct`, `ifshort`, `maligned`, `interfacer`, `deadcode`, `golint`, `varcheck`, `structcheck`, `nosnakecase`, `scopelint`, `varnamelen`
- Timeout: 5 minutes
- Tests included in linting (`tests: true`)
- Relaxed rules for test files (`.golangci.yml:59-70`): excludes `gocyclo`, `errcheck`, `dupl`, `gosec`, `gochecknoglobals`, `exhaustruct`, `ireturn`, `funlen`, `unparam`, `lll`
- `depguard` restricts imports: production code allows only `$gostd`, test code additionally allows `github.com/stretchr/testify`, cmd code allows only `$gostd`

**Key lint suppressions (nolint directives used deliberately):**
- `//nolint:exhaustruct` -- widely used for protobuf-generated types and structs with default-zero fields. See `internal/connect/api.go:38`, `internal/providers/localgit/localgit.go:126`, `internal/https/https.go:22`
- `//nolint:ireturn` -- used on functions returning interface types. See `cmd/easyp/main.go:260`, `internal/providers/multisource/repo.go:122`
- `//nolint:wrapcheck` -- used for transparent error passthrough from dependencies. See `internal/https/https.go:41`, `internal/providers/multisource/repo.go:59`
- `//nolint:gochecknoglobals` -- used for legitimate module-level vars (flags, template caches). See `cmd/easyp/main.go:27`, `internal/providers/bitbucket/client.go:61`
- `//nolint:musttag` -- used for JSON encode/decode of internal `[]content.File` types. See `internal/providers/cache/file.go:37`, `internal/providers/cache/artifactory/artifactory.go:84`
- `//nolint:gomnd` -- used for octal permission `0o750`. See `internal/providers/cache/file.go:51`
- `//nolint:lll` -- used for long interface method signatures. See `internal/providers/github/client.go:16`
- `//nolint:nilerr` -- used intentionally in `fs.WalkDir` callback to skip errors on directories. See `internal/providers/localgit/localgit.go:186`

## Import Organization

**Order (enforced by gci):**
1. Standard library: `"context"`, `"fmt"`, `"net/http"`, `"os"`
2. Third-party: `"connectrpc.com/connect"`, `"golang.org/x/exp/slog"`, `"github.com/google/go-github/v59/github"`
3. Project: `"github.com/easyp-tech/server/internal/..."`

**Example from `internal/connect/blobs.go`:**
```go
import (
    "bytes"
    "context"
    "fmt"

    "connectrpc.com/connect"

    module "github.com/easyp-tech/server/gen/proto/buf/alpha/module/v1alpha1"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [easyp-tech/server](https://github.com/easyp-tech/server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
