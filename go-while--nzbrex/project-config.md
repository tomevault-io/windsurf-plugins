---
trigger: always_on
description: NZBreX is a Go command-line tool for restoring missing Usenet articles by downloading them from providers where they are still available and re-uploading them to others. The application requires building the rapidyenc C library first, then the Go application with CGO enabled.
---

# NZBreX - NZB Refresh X

NZBreX is a Go command-line tool for restoring missing Usenet articles by downloading them from providers where they are still available and re-uploading them to others. The application requires building the rapidyenc C library first, then the Go application with CGO enabled.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Dependencies
- Install required system packages:
  ```bash
  apt-get install build-essential cmake ca-certificates curl git dpkg-dev wget
  # For cross-compilation (optional):
  apt-get install gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu mingw-w64 gcc-mingw-w64-x86-64 g++-mingw-w64-x86-64 g++-aarch64-linux-gnu
  ```
- Go 1.24.3 or later is required (as specified in go.mod)
- CMake 3.31+ is required for building rapidyenc

### Bootstrap, Build, and Test the Repository
1. **Build rapidyenc C library first** (REQUIRED):
   ```bash
   cd rapidyenc && ./build_rapidyenc_linux-amd64.sh
   ```
   - Takes ~15 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
   - Creates `rapidyenc/rapidyenc/build/rapidyenc_static/librapidyenc.a`
   - Also builds CLI tools: `rapidyenc_cli` and `rapidyenc_bench`

2. **Build Go application**:
   ```bash
   ./local_build_linux-amd64.sh
   ```
   - OR manually: `go build -o NZBreX -tags other .`
   - Takes ~15-30 seconds total. NEVER CANCEL. Set timeout to 120+ seconds.
   - First build downloads dependencies (~25 seconds)
   - Subsequent builds are faster (~5 seconds)

3. **Run tests**:
   ```bash
   go test ./rapidyenc/
   ```
   - Takes ~5 seconds. Set timeout to 60+ seconds.

4. **Run tests with race detector**:
   ```bash
   go test -race ./rapidyenc/
   ```
   - Takes ~5 seconds. Set timeout to 60+ seconds.

5. **Run code validation**:
   ```bash
   ./vet.sh
   ```
   - Takes ~7 seconds. Set timeout to 60+ seconds.

6. **Test rapidyenc integration**:
   ```bash
   ./NZBreX -testrapidyenc
   ```
   - Tests yenc encoding/decoding functionality
   - Takes ~1 second. Should always work.

## Validation

### Essential Validation Steps
- **ALWAYS run through the complete build sequence** after making changes:
  1. Clean build: `rm -rf rapidyenc/rapidyenc/build && rm -f NZBreX`
  2. Build rapidyenc: `cd rapidyenc && ./build_rapidyenc_linux-amd64.sh`
  3. Build application: `./local_build_linux-amd64.sh`
  4. Test rapidyenc: `./NZBreX -testrapidyenc`
  5. Run validation: `./vet.sh`

- **Manual validation scenarios**:
  - Test help: `./NZBreX -help` (should show usage)
  - Test version: `./NZBreX -version` (shows build version)
  - Validate rapidyenc: `./NZBreX -testrapidyenc` (tests yenc codec)

- **CRITICAL**: Application functionality cannot be fully tested without Usenet provider access. The provider.ygg.json requires yggdrasil network access which may not be available in all environments.

- **Before committing**: Always run `./vet.sh` and ensure the application builds and basic commands work.

## Running the Application

### Basic Usage
```bash
# Check application help
./NZBreX -help

# Test with sample NZB (check-only mode, safe)
./NZBreX -checkonly -nzb nzbs/debian-11.6.0-amd64-netinst.iso.nzb.gz -provider provider.sample.json

# Full operation (requires valid Usenet provider configuration)
./NZBreX -cd /path/to/cache -checkfirst -nzb nzbs/example.nzb.gz -provider provider.json
```

### Key Command-Line Options
- `-checkonly`: Safe mode - only check article availability, no downloads/uploads
- `-cd string`: Cache directory path (required for most operations)
- `-nzb string`: Path to NZB file (default: nzbs/ubuntu-24.04-live-server-amd64.iso.nzb.gz)
- `-provider string`: Provider configuration file (default: provider.json)
- `-verbose`: Enable detailed output (default: true)
- `-testrapidyenc`: Test rapidyenc functionality and exit

## Common Tasks

### Repository Structure
```
/home/runner/work/NZBreX/NZBreX/
├── README.md                    # Main documentation
├── main.go                      # Application entry point
├── *.go                         # Go source files (Config.go, Workers.go, etc.)
├── go.mod, go.sum              # Go module files
├── rapidyenc/                   # C library for yenc encoding/decoding
│   ├── rapidyenc/              # Actual rapidyenc source code
│   ├── build_rapidyenc_*.sh    # Build scripts for different platforms
│   └── required.deb-packages.txt # Required system packages
├── nzbs/                       # Sample NZB files for testing
├── yenc/                       # Test data for yenc functionality
├── provider.sample.json        # Sample provider configuration
├── provider.ygg.json          # Yggdrasil network test providers
├── cleanHeaders.txt            # Headers to clean during processing
├── local_build_linux-amd64.sh # Main build script
├── vet.sh                      # Code validation script
├── race_linux.sh              # Race detector test script
└── .github/workflows/          # CI/CD workflows
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-while/NZBreX](https://github.com/go-while/NZBreX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
