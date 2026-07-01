---
trigger: always_on
description: XSSMaze is a Crystal-based web application designed to be vulnerable to XSS (Cross-Site Scripting) attacks. It serves as a testing platform to measure and enhance the performance of security testing tools. The application uses the Kemal web framework and provides various XSS vulnerability scenarios.
---

# XSSMaze Development Instructions

XSSMaze is a Crystal-based web application designed to be vulnerable to XSS (Cross-Site Scripting) attacks. It serves as a testing platform to measure and enhance the performance of security testing tools. The application uses the Kemal web framework and provides various XSS vulnerability scenarios.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Setup
- Install Crystal programming language (version 1.8.2 as specified in shard.yml)
- Ensure network connectivity for dependency installation
- Docker (optional, for containerized builds)

### Bootstrap, Build, and Test the Repository
```bash
# Install Crystal dependencies
shards install
# NEVER CANCEL: Dependency installation can take 5-10 minutes depending on network speed. Set timeout to 15+ minutes.

# Development build
shards build
# NEVER CANCEL: Build takes 2-5 minutes depending on system. Set timeout to 10+ minutes.

# Production build (recommended for deployment)
shards build --release --no-debug --production
# NEVER CANCEL: Production build takes 3-7 minutes with optimizations. Set timeout to 15+ minutes.

# Run tests
crystal spec
# NEVER CANCEL: Test suite takes 1-2 minutes. Set timeout to 5+ minutes.
```

### Running the Application
```bash
# Run XSSMaze (after successful build)
./bin/xssmaze

# Alternative: Run with custom configuration
./bin/xssmaze -b 0.0.0.0 -p 8080

# Default runs on http://127.0.0.1:3000 (loopback only).
# Pass `-b 0.0.0.0` to expose the lab on the network.
```

### Command Line Options
- `-b HOST, --bind HOST`: Host to bind (defaults to 127.0.0.1; use `0.0.0.0` to expose to the network)
- `-p PORT, --port PORT`: Port to listen for connections (defaults to 3000)
- `-s, --ssl`: Enables SSL
- `--ssl-key-file FILE`: SSL key file
- `--ssl-cert-file FILE`: SSL certificate file
- `-h, --help`: Shows help

### Docker Build and Run
```bash
# Build Docker image
docker build -t xssmaze .
# NEVER CANCEL: Docker build takes 10-15 minutes. Set timeout to 30+ minutes.

# Run using pre-built image
docker run -p 3000:3000 ghcr.io/hahwul/xssmaze:main

# Multi-arch build (single Dockerfile supports amd64 + arm64)
docker buildx build --platform linux/amd64,linux/arm64 -t xssmaze .
```

## Validation

### Manual Testing Scenarios
After building and running XSSMaze, ALWAYS validate functionality by:

1. **Basic Connectivity Test:**
   ```bash
   curl http://localhost:3000/
   # Should return HTML with XSSMaze title and endpoint list
   ```

2. **API Endpoint Tests:**
   ```bash
   # Test map endpoints
   curl http://localhost:3000/map/text
   curl http://localhost:3000/map/json
   # Should return list of vulnerable endpoints
   ```

3. **XSS Vulnerability Tests:**
   ```bash
   # Test basic XSS endpoint
   curl "http://localhost:3000/basic/level1/?query=<script>alert(1)</script>"
   # Should return the script tag (demonstrates vulnerability)

   # Test escaped XSS endpoint
   curl "http://localhost:3000/basic/level2/?query=<script>alert(1)</script>"
   # Should return escaped content
   ```

4. **Complete User Scenario:**
   - Access the main page at http://localhost:3000
   - Verify all XSS test cases are listed
   - Test at least 3 different vulnerability levels
   - Check that /map/json returns valid JSON with endpoint list

### Linting and Code Quality
```bash
# Install Crystal Ameba linter (if not installed)
# The CI uses crystal-ameba/github-action@v0.8.0

# Run linting (via GitHub Actions workflow)
# Manual linting requires Ameba to be installed separately
```

## Repository Structure and Navigation

### Key Directories
```
/home/runner/work/xssmaze/xssmaze/
├── src/                    # Main source code
│   ├── xssmaze.cr         # Application entry point
│   ├── maze.cr            # Maze class definition
│   ├── banner.cr          # Application banner
│   ├── mazes/             # XSS vulnerability implementations
│   │   ├── basic.cr       # Basic XSS scenarios
│   │   ├── dom.cr         # DOM-based XSS
│   │   ├── header.cr      # Header injection XSS
│   │   ├── post.cr        # POST-based XSS
│   │   └── [others].cr    # Additional XSS types
│   └── protections/       # XSS protection modules
│       └── protections.cr # Protection escape functions
├── spec/                  # Test files
│   ├── spec_helper.cr     # Test configuration
│   └── xssmaze_spec.cr    # Main test suite (minimal)
├── .github/workflows/     # CI/CD configuration
├── shard.yml             # Dependencies (like package.json)
├── shard.lock           # Lock file with exact versions
├── Dockerfile           # Docker build configuration
└── README.md           # Project documentation
```

### Important Files to Review When Making Changes
- Always check `src/xssmaze.cr` after modifying route definitions
- Review `shard.yml` when adding new dependencies
- Update `README.md` if adding new XSS vulnerability types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hahwul/xssmaze](https://github.com/hahwul/xssmaze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
