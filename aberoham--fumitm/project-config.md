---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

fumitm (Fix Up My Interception of TLS, Man) is a Python script that automatically fixes TLS certificate trust issues caused by MITM proxies. It supports multiple providers — currently Cloudflare WARP and Netskope — and configures various development tools to trust the proxy's CA certificate.

## Key Commands

### Running the Script

```bash
# Check current certificate status (no changes made, auto-detects provider)
./fumitm.py

# Actually install/update certificates (makes changes)
./fumitm.py --fix

# Explicitly select a provider instead of auto-detecting
./fumitm.py --provider netskope
./fumitm.py --provider warp --fix

# Run with detailed debug output for troubleshooting
./fumitm.py --debug
./fumitm.py --debug --fix  # Debug mode with fixes

# Show help
./fumitm.py --help

# List all available tools and their tags
./fumitm.py --list-tools

# Non-interactive mode (answer yes to all prompts, for curl-pipe one-liners)
./fumitm.py --fix --yes

# Check/fix specific tools only
./fumitm.py --tools node --tools python  # Check Node.js and Python only
./fumitm.py --fix --tools node-npm,gcloud  # Fix Node.js/npm and gcloud only
./fumitm.py --fix --tools java,db  # Fix Java and database tools using tags

# Headless/MDM mode (JAMF, Ansible, Puppet)
./fumitm.py --fix --yes --headless --provider netskope
./fumitm.py --fix --yes --headless --run-as-user $USER --log-dir /var/log/fumitm

# Disable colors (also respects NO_COLOR=1 env var)
./fumitm.py --no-color

# Log to file or directory
./fumitm.py --log-file /tmp/fumitm.log
./fumitm.py --log-dir /var/log/fumitm --json-log-dir /var/log/fumitm
```

### Testing

The project has a pytest-based test suite in `test_suite/`:

```bash
# Run all tests
cd test_suite
uvx pytest test_fumitm_integration.py test_netskope_provider.py test_suspicious_bundles.py test_headless_mdm.py -v

# Run specific test files or classes
uvx pytest test_fumitm_integration.py::TestStatusFunctionContracts -v
uvx pytest test_fumitm_integration.py::TestCodeQuality -v
uvx pytest test_netskope_provider.py -v
```

Key test categories in `test_fumitm_integration.py`:
- **TestCertificateManagement**: Certificate download and validation
- **TestBrewCacerts**: Homebrew ca-certificates setup and status checking
- **TestToolSetup**: Tool-specific certificate setup workflows
- **TestStatusFunctionContracts**: Ensures all `check_*_status()` functions return booleans
- **TestCodeQuality**: Static analysis tests that enforce code standards:
  - No unsafe certificate appends (use `safe_append_certificate()`)
  - No unused global variables
  - Consistent messaging ("Configuring" not "Setting up")
  - No bare `except:` clauses (use `except Exception:`)
- **TestBundleCreation**: Tests for `create_bundle_with_system_certs()` helper
- **TestCertificateAppending**: Tests for safe PEM file handling (issue #13 fix)
- **TestPerformance**: Ensures subprocess call limits aren't exceeded
- **TestCertificateContentMatching**: Tests for pure-Python certificate matching
- **TestUpdateCheck**: Tests for the auto-update check functionality
- **TestGcloudVerification**: Tests for gcloud connectivity verification
- **TestOwnershipProtection**: Tests for sudo detection and file ownership correction

Key test categories in `test_netskope_provider.py`:
- **TestProviderDetection**: WARP and Netskope detection (cert files, encrypted certs, STAgent process)
- **TestProviderResolution**: Auto-detect priority, explicit override, invalid provider handling
- **TestNetskopeProviderConfig / TestNetskopeWarpProviderConfig**: Config propagation (cert_path, bundle_dir, keytool_alias, container_cert_name)
- **TestNetskopeGetCert**: Certificate retrieval (file read, keychain fallback with root + intermediate)
- **TestProviderCLI**: `--provider` argument parsing
- **TestCheckProviderConnection**: Provider-specific connection status checking

Key test categories in `test_headless_mdm.py`:
- **TestColorControl**: No color when `--no-color`, `NO_COLOR` env, `--headless`, non-TTY stdout
- **TestHeadlessFlag**: `--headless` disables color and update check, does NOT imply `--yes`
- **TestNonInteractiveError**: Non-TTY without `--yes` raises `NonInteractiveError`, exit code 2
- **TestLogFile**: `--log-file` and `--log-dir` text logging with timestamps and symlinks
- **TestJsonLogFile**: JSON-lines logging with schema validation
- **TestToolResultWrapper**: `_run_setup()` wraps legacy functions, error counting, exception handling
- **TestChangesmadeAccuracy**: `changes_made` is null/true/false based on ToolResult statuses
- **TestExitCodes**: 0 success, 1 hard failure, 2 non-interactive, 3 partial, 130 interrupted
- **TestRunAsUser**: `--run-as-user` user targeting, auto detection, root requirement
- **TestUserScopeGating**: User-scoped tools skipped without user context
- **TestSudoHelperUpdates**: Updated sudo helpers use `_target_uid`

## Architecture Overview

The script follows a modular architecture with these key components:

1. **Mode System**: Two modes - "status" (default, read-only) and "install" (with `--fix` flag)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aberoham/fumitm](https://github.com/aberoham/fumitm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
