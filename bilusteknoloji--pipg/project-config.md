---
trigger: always_on
description: **pipg** is a Python package installer written in Go. It is a **drop-in
---

# CLAUDE.md — pipg

## Project Summary

**pipg** is a Python package installer written in Go. It is a **drop-in
replacement for `pip install`** — nothing more. It does NOT manage projects,
virtual environments, lock files, or pyproject.toml. It simply installs
packages, just like `pip install`, but downloads them **concurrently** using
goroutines.

Think of it as: `pip install` but faster, thanks to parallel downloads. That's
the entire value proposition.

**pipg is NOT:**

- A project manager (unlike uv, poetry, pdm)
- A virtual environment manager
- A build tool
- It does NOT create any config files (no toml, no lock files, no yaml, nothing)

**pipg IS:**

- A fast package installer that works exactly like `pip install`
- You point it at a Python environment (venv or system) and it installs packages there
- Same mental model as pip: `pipg install requests` → done

## Usage

```bash
pipg install requests
pipg install "flask>=3.0" "sqlalchemy<2.0"
pipg install -r requirements.txt
```

The user should be able to use `pipg install` anywhere they'd use `pip
install`. No setup, no config files, no ceremony.

## Architecture

### Modules

```
pipg/
├── cmd/
│   └── pipg/
│       └── main.go            # CLI entry point (cobra or bare flags)
├── internal/
│   ├── pypi/
│   │   ├── client.go          # PyPI JSON API client (GET https://pypi.org/pypi/{pkg}/json)
│   │   └── models.go          # PyPI API response structs
│   ├── resolver/
│   │   ├── resolver.go        # Dependency resolution (BFS/DFS, version conflict detection)
│   │   └── version.go         # PEP 440 version parsing & comparison
│   ├── downloader/
│   │   ├── downloader.go      # Concurrent download manager (errgroup + semaphore)
│   │   └── wheel.go           # Platform-compatible wheel selection (PEP 425 tags)
│   ├── installer/
│   │   ├── installer.go       # Wheel extract → site-packages
│   │   └── record.go          # RECORD, METADATA, INSTALLER file management
│   └── python/
│       └── env.go             # Detect active Python environment (sys.prefix, site-packages path, platform tag)
├── go.mod
├── go.sum
├── CLAUDE.md
└── README.md
```

### Flow

```
CLI parse args
    → Detect Python environment (venv / system)
    → Fetch metadata from PyPI API for each package
    → Build dependency tree (resolver)
    → Select compatible wheel for each package
    → Concurrent download (goroutines, default: GOMAXPROCS workers)
    → Install wheels sequentially or in parallel (unzip → site-packages)
    → Print result summary
```

## Technical Requirements & Rules

### PyPI API

- Endpoint: `GET https://pypi.org/pypi/{package_name}/json`
- Specific version: `GET https://pypi.org/pypi/{package_name}/{version}/json`
- From response: `info.requires_dist` → dependency list (PEP 508 format)
- From response: `urls[]` → downloadable files (wheel, sdist)
- Respect rate limits: max 8 concurrent requests, retry with backoff

### PEP 440 — Version Parsing

- Versions: `1.0`, `1.0.post1`, `1.0a1`, `1.0b2`, `1.0rc1`, `1.0.dev1`
- Specifiers: `>=1.0,<2.0`, `==1.5.*`, `~=1.4.2`, `!=1.3`
- Comparison: compare epoch, release, pre, post, dev segments separately
- Use an existing library like `github.com/aquasecurity/go-pep440-version` if available, otherwise write your own parser

### PEP 508 — Dependency Specifiers

- Format: `package_name[extra1,extra2] (>=1.0,<2.0) ; python_version >= "3.8"`
- Parse extras and environment markers
- Marker evaluation: get variables like `python_version`, `sys_platform`, `os_name` from the active Python
- Skip dependencies whose markers don't match the current environment

### PEP 425 — Wheel Compatibility Tags

- Wheel filename format: `{name}-{ver}-{python}-{abi}-{platform}.whl`
- Example: `requests-2.31.0-py3-none-any.whl`, `numpy-1.26.0-cp312-cp312-manylinux_2_17_x86_64.whl`
- Priority order: exact match > compatible > pure python (`py3-none-any`)
- Get compatible tag list from active Python: `python -c "import packaging.tags; ..."`
- If no wheel is found, do NOT fall back to sdist — raise an error (sdist build is complex, out of scope)

### Dependency Resolution

- A simple iterative resolver is sufficient (do NOT build a full SAT solver like pip's backtracking resolver)
- Algorithm:
  1. Start from root packages
  2. Fetch requires_dist for each package
  3. Walk the entire dependency tree using BFS
  4. If the same package is requested with multiple specifiers, find the intersection
  5. If intersection is empty → raise a conflict error and exit
  6. Select the highest compatible version for each package
- Check for circular dependencies
- SKIP extras support in v1 (can be added later)

### Concurrent Download

- Use `golang.org/x/sync/errgroup`
- Max concurrency: defaults to `runtime.GOMAXPROCS(0)` (i.e., number of available CPUs).
  User can override via -`-jobs N` flag.
- Each goroutine: HTTP GET → write to temp file → verify hash (PyPI sha256)
- If file hash doesn't match `digests.sha256` from PyPI response → error
- Retry: max 3 attempts, exponential backoff
- Progress display: print `downloading...` / `done ✓` line for each package
- All downloads over HTTPS. Do NOT disable TLS certificate verification. 
  Go's net/http handles this by default.

### Wheel Installation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bilusteknoloji/pipg](https://github.com/bilusteknoloji/pipg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
