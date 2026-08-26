---
trigger: always_on
description: This project uses [release-please](https://github.com/googleapis/release-please) for automated releases. Follow these commit message conventions to ensure proper versioning and changelog generation.
---

# Cursor Rules for Zigbee2MQTT Exporter

## Commit Message Guidelines

This project uses [release-please](https://github.com/googleapis/release-please) for automated releases. Follow these commit message conventions to ensure proper versioning and changelog generation.

### Commit Message Format

Use conventional commit format with the following structure:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Types

- **feat**: A new feature (triggers minor version bump)
- **fix**: A bug fix (triggers patch version bump)
- **docs**: Documentation only changes
- **style**: Changes that do not affect the meaning of the code (white-space, formatting, etc)
- **refactor**: A code change that neither fixes a bug nor adds a feature
- **perf**: A code change that improves performance
- **test**: Adding missing tests or correcting existing tests
- **build**: Changes that affect the build system or external dependencies
- **ci**: Changes to CI configuration files and scripts
- **chore**: Other changes that don't modify src or test files
- **revert**: Reverts a previous commit

### Scopes (Optional)

Use scopes to indicate which part of the codebase is affected:

- **config**: Configuration changes
- **metrics**: Metrics collection or export changes
- **websocket**: WebSocket connection or message handling changes
- **collectors**: Data collection changes
- **server**: HTTP server changes
- **logging**: Logging system changes
- **docker**: Docker-related changes
- **docs**: Documentation changes
- **ci**: CI/CD changes
- **zigbee**: Zigbee device handling changes
- **mqtt**: MQTT integration changes

### Examples

✅ **Good commit messages:**

```
feat(metrics): add device type classification metrics

feat: add support for custom device type labels

fix(websocket): handle connection timeouts gracefully

fix: resolve memory leak in device collection

docs: update configuration examples in README

refactor(server): improve HTTP handler error handling

perf: optimize device data processing algorithm

test: add unit tests for WebSocket connection handling

build: update Go version to 1.24

ci: add automated security scanning

chore: update dependencies to latest versions
```

❌ **Bad commit messages:**

```
update code
fix stuff
add feature
bug fix
```

❌ **Bad commit practices:**

- Committing multiple unrelated changes together
- Large commits that are hard to review
- Not reviewing changes before committing
- Committing without understanding what changed

### Breaking Changes

For breaking changes, add `!` after the type/scope and include `BREAKING CHANGE:` in the footer:

```
feat!: remove deprecated config option

BREAKING CHANGE: The `old_config_option` has been removed. Use `new_config_option` instead.
```

### Commit Message Best Practices

1. **Use imperative mood**: "add" not "added" or "adds"
2. **Don't capitalize the first letter**: "add feature" not "Add feature"
3. **No period at the end**: "add feature" not "add feature."
4. **Keep it under 72 characters** for the first line
5. **Be specific and descriptive**
6. **Reference issues when relevant**: "fix: resolve memory leak (#123)"
7. **Inspect before committing**: Always review `git diff --staged` before committing
8. **Keep commits focused**: One logical change per commit
9. **Prefer smaller commits**: Easier to review, revert, and understand

### Release-Please Integration

- **feat** commits trigger minor version bumps
- **fix** commits trigger patch version bumps
- **BREAKING CHANGE** commits trigger major version bumps
- **docs**, **style**, **refactor**, **perf**, **test**, **build**, **ci**, **chore** don't trigger version bumps

### Development Workflow

**Always follow this workflow when making changes:**

1. ✅ **Make your changes**
2. ✅ **Run tests**: `make test` - Ensure all tests pass
3. ✅ **Run linting**: `make lint` - Fix any linting issues
4. ✅ **Format code**: `make fmt` - Ensure consistent formatting
5. ✅ **Verify changes work** - Test your changes manually if needed
6. ✅ **Inspect your changes**: Review what you're about to commit
7. ✅ **Choose appropriate commit message**: Use conventional commit format
8. ✅ **Commit when it works**: Only commit working, tested code
9. ✅ **Push immediately**: `git push` - Don't let changes sit locally

### Development Workflow

**Always follow this workflow when making changes:**

1. ✅ **Check git status**: `git status` - Ensure you're up to date and on the right branch
2. ✅ **Make your changes**
3. ✅ **Run tests**: `make test` - Ensure all tests pass
4. ✅ **Run linting**: `make lint` - Fix any linting issues
5. ✅ **Format code**: `make fmt` - Ensure consistent formatting
6. ✅ **Verify changes work** - Test your changes manually if needed
7. ✅ **Inspect your changes**: Review what you're about to commit
8. ✅ **Choose appropriate commit message**: Use conventional commit format
9. ✅ **Commit when it works**: Only commit working, tested code
10. ✅ **Push immediately**: `git push` - Don't let changes sit locally

### Git Status Check

**Always check git status before starting work:**

```bash
git status
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [d0ugal/zigbee2mqtt-exporter](https://github.com/d0ugal/zigbee2mqtt-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
