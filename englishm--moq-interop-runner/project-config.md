---
trigger: always_on
description: Guidelines for AI assistants helping with this repository.
---

# MoQ Interop Runner - Agent Guidelines

Guidelines for AI assistants helping with this repository.

## Repository Purpose

This is a framework for testing interoperability between MoQT (Media over QUIC Transport) implementations. It orchestrates tests between different MoQT relays and clients, producing machine-readable results and HTML reports.

## Key Files and Directories

### Core Files

| File | Purpose |
|------|---------|
| `implementations.json` | Registry of all MoQT implementations with endpoints and Docker images |
| `implementations.schema.json` | JSON Schema for validating implementations.json |
| `Makefile` | Primary command interface (`make help` for all targets) |
| `run-interop-tests.sh` | Main test orchestration script |
| `docker-compose.test.yml` | Docker Compose configuration for local testing |
| `generate-report.sh` | Generates HTML interop matrix from test results |
| `generate-certs.sh` | Generates self-signed TLS certificates for testing |
| `lib/tap-parser.sh` | TAP v14 output parser for result aggregation |

### Directories

| Directory | Purpose |
|-----------|---------|
| `docs/` | All documentation (getting started, test specs, guides) |
| `builds/` | Source build configurations for compiling implementations |
| `adapters/` | Thin Docker wrappers for upstream images |
| `.github/workflows/` | CI: daily interop runs, GHCR image publishing |
| `results/` | Test output (gitignored, published to gh-pages) |

## Documentation Map

| Task | Start Here |
|------|------------|
| Understand the project | `README.md` |
| Get started as a contributor | `docs/GETTING-STARTED.md` |
| See test case definitions | `docs/tests/TEST-CASES.md` |
| Build a test client | `docs/IMPLEMENTING-A-TEST-CLIENT.md` |
| Test client interface contract | `docs/TEST-CLIENT-INTERFACE.md` |
| Docker testing workflows | `docs/DOCKER-TESTING.md` |
| Version matching logic | `docs/VERSION-MATCHING.md` |
| Adapter conventions | `adapters/README.md` |
| Source build framework | `builds/README.md` |
| Architecture decisions | `docs/decisions/` |

## Common Contributor Paths

### Registering or Updating an Implementation

Edit `implementations.json` to add or modify entries. Each implementation needs:

- `name`: Display name
- `roles`: Object with `relay` and/or `client` capabilities
- `draft_versions`: Array of supported drafts (e.g., `["draft-16"]`)
- Optionally: `organization`, `repository`, `notes`

**For remote endpoints** (public relays), add under `roles.relay.remote`:

```json
{
  "url": "https://relay.example.com:4443/moq",
  "transport": "webtransport",
  "tls_disable_verify": false
}
```

**For Docker images**, there are three strategies:

1. **Pre-built on GHCR** (preferred): Set `roles.<role>.docker.image` to the GHCR image name
2. **Adapter**: Wrap an upstream image with convention mapping (see `adapters/README.md`)
3. **Source build**: Compile from source (see `builds/README.md`)

Full walkthrough: `docs/GETTING-STARTED.md`

### Building a Test Client

Test clients must follow the interface contract in `docs/TEST-CLIENT-INTERFACE.md`:

- **Output**: TAP version 14 on stdout
- **Exit codes**: 0 = all passed, 1 = failure, 127 = unsupported test
- **Environment variables**: `RELAY_URL`, `TESTCASE`, `TLS_DISABLE_VERIFY`, `VERBOSE`

Reference implementations to use as templates:

- `builds/moq-rs/` - Rust client (from upstream moq-rs)
- `builds/moq-dev-rs/` - Rust client (self-contained)
- `builds/moq-dev-js/` - TypeScript/Bun client (self-contained)

Full guide: `docs/IMPLEMENTING-A-TEST-CLIENT.md`

### Adding or Modifying Test Cases

Test cases are defined in `docs/tests/TEST-CASES.md`. Current tests:

| Identifier | Category | Description |
|------------|----------|-------------|
| `setup-only` | Session | Basic CLIENT_SETUP/SERVER_SETUP exchange |
| `announce-only` | Namespace | PUBLISH_NAMESPACE flow |
| `publish-namespace-done` | Namespace | Unpublish namespace |
| `subscribe-error` | Subscription | Error for non-existent track |
| `announce-subscribe` | Subscription | Publisher announces, subscriber subscribes |
| `subscribe-before-announce` | Subscription | Out-of-order subscribe/announce |

When adding tests, follow the existing format (identifier, protocol refs, procedure, success criteria). Both the spec doc and test client implementations need updates.

### Maintaining the Framework

- **CI workflows**: `.github/workflows/interop-report.yml` (daily test runs), `build-images.yml` (GHCR publishing)
- **Reports**: `generate-report.sh` parses TAP output via `lib/tap-parser.sh`, publishes to gh-pages
- **Version matching**: `run-interop-tests.sh` pairs clients with relays, classifies results relative to `current_target` in `implementations.json`

## implementations.json Structure

```json
{
  "version": "1.1",
  "current_target": "draft-16",
  "implementations": {
    "example-impl": {
      "name": "Example Implementation",
      "organization": "Example Org",
      "repository": "https://github.com/example/moqt",
      "draft_versions": ["draft-16", "draft-14"],
      "roles": {
        "relay": {
          "docker": { "image": "ghcr.io/example/relay:latest" },
          "remote": [{ "url": "https://...", "transport": "webtransport" }]
        },
        "client": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [englishm/moq-interop-runner](https://github.com/englishm/moq-interop-runner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
