---
trigger: always_on
description: `mmdbconvert` is a Go command-line tool that merges multiple MaxMind MMDB
---

# mmdbconvert - AI Assistant Context

## Project Overview

`mmdbconvert` is a Go command-line tool that merges multiple MaxMind MMDB
(MaxMind Database) files and exports the merged data to CSV or Parquet format.
The primary use case is merging GeoIP databases (e.g., GeoIP2-Enterprise +
GeoIP2-Anonymous-IP) while maintaining non-overlapping network blocks and
allowing flexible column mapping.

**Key Features:**

- Merge data from multiple MMDB databases
- Output to CSV or Parquet format
- Flexible column mapping via TOML configuration
- Query-optimized Parquet output with integer columns for efficient IP lookups
- Streaming architecture for memory-efficient processing
- Optional type hints for Parquet native types (int64, float64, bool, etc.)

## Project Structure

```
mmdbconvert/
├── cmd/
│   └── mmdbconvert/
│       └── main.go              # CLI entry point
├── internal/
│   ├── config/                  # TOML configuration parsing & validation
│   ├── mmdb/                    # MMDB database reading & data extraction
│   ├── network/                 # IP/CIDR utilities
│   └── writer/                  # CSV and Parquet writers
├── examples/                    # Example configuration files
├── testdata/                    # Test MMDB files
├── docs/
│   ├── config.md                # Configuration file reference
│   └── parquet-queries.md       # Parquet query optimization guide
├── plan.md                      # Detailed implementation plan
├── README.md
├── LICENSE
├── .gitignore
├── .precious.toml               # Precious (linter runner) configuration
├── .prettierrc.json             # Prettier configuration for markdown
├── go.mod
└── go.sum
```

## Dependencies

- `github.com/pelletier/go-toml/v2` - TOML configuration parsing
- `github.com/oschwald/maxminddb-golang/v2` - MMDB database reading
- `github.com/parquet-go/parquet-go` - Parquet file writing

## Building

```bash
# Build the binary
go build -o mmdbconvert ./cmd/mmdbconvert

# Install to $GOPATH/bin
go install ./cmd/mmdbconvert
```

## Running

```bash
# Run with config file
./mmdbconvert config.toml

# Or with explicit flag
./mmdbconvert --config config.toml

# Suppress progress output
./mmdbconvert --config config.toml --quiet
```

## Testing

```bash
# Run all tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Run tests with race detection
go test -race ./...

# Run tests verbosely
go test -v ./...

# Run benchmarks
go test -bench ./...
```

## Code Quality & Formatting

### Linting with golangci-lint

```bash
# Run all linters
golangci-lint run

# Run with auto-fix where possible
golangci-lint run --fix
```

**Note:** This project uses `golangci-lint` for Go code quality checks. Ensure
it's installed:

```bash
# Install golangci-lint
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

### Formatting

```bash
# Format Go code (standard)
gofmt -w .

# Or use goimports for import organization
goimports -w .
```

**Note:** There is no `golangci-lint fmt` command. Use `gofmt` or `goimports`
for formatting.

### Using Precious (Unified Linter Runner)

This project uses [Precious](https://github.com/houseabsolute/precious) to run
linters consistently:

```bash
# Run all configured linters (includes prettier for markdown)
precious tidy

# Check without making changes (CI mode)
precious lint

# Format specific files
precious tidy docs/config.md plan.md
```

**Precious Configuration:** See `.precious.toml` for configured linters:

- `prettier-markdown` - Formats markdown files with 80-character line wrap

### Pre-commit Checklist

Before committing changes:

1. **Format code:**

   ```bash
   gofmt -w .
   ```

2. **Run linters:**

   ```bash
   golangci-lint run
   ```

3. **Format markdown:**

   ```bash
   precious tidy
   ```

4. **Run tests:**
   ```bash
   go test ./...
   ```

## Development Workflow

### Adding a New Feature

1. Read the implementation plan in `plan.md`
2. Implement changes following the project structure
3. Add tests for new functionality
4. Run linters and tests
5. Update documentation if needed
6. Format code and markdown

### Key Implementation Details

**Streaming Architecture:**

- Never collect all networks in memory
- Use streaming accumulator pattern
- Write rows immediately when data changes
- Ensures O(1) memory regardless of database size

**Network Iteration:**

- Uses `maxminddb.Networks()` and `NetworksWithin()`
- Nested iteration pattern for multiple databases
- Always uses smallest overlapping network
- Adjacent networks with identical data are merged

**Error Handling:**

- Fail-fast approach for data extraction and type conversion errors
- Exit immediately on JSON encoding failures or type mismatches
- Ensures data quality over partial completion

**Type System:**

- CSV: All values as strings
- Parquet: Optional type hints (string, int64, float64, bool, binary)
- Type mismatches cause immediate failure

## Configuration

Configuration is via TOML file. See:

- `docs/config.md` - Full configuration reference
- `examples/` - Example configuration files

## Documentation

- `README.md` - User-facing documentation
- `docs/config.md` - Configuration file reference
- `docs/parquet-queries.md` - How to query generated Parquet files efficiently

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxmind/mmdbconvert](https://github.com/maxmind/mmdbconvert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
