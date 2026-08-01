---
trigger: always_on
description: Welcome to the octoDNS organization workspace. This guide outlines the project structure, development workflow, and coding guidelines for developers and AI agents.
---

# Developer Agent Guide

Welcome to the octoDNS organization workspace. This guide outlines the project structure, development workflow, and coding guidelines for developers and AI agents.

---

## General Workflow & Guidelines (All Repositories)

This section applies to all repositories under the octoDNS organization (including the core framework and provider/add-on modules).

### Project Structure & Aim

octoDNS is a tool and framework that provides a set of tools and patterns to manage DNS records across multiple providers as code (Infrastructure as Code).

- **Core Repository**: [octodns](https://github.com/octodns/octodns) is the core repository containing the main framework and CLI tools.
- **Provider & Add-on Modules**: Subdirectories/repositories prefixed with `octodns-` (such as [octodns-cloudflare](https://github.com/octodns/octodns-cloudflare), [octodns-route53](https://github.com/octodns/octodns-route53), etc.) are individual modules for integrating different DNS providers or adding specific features.
- **Tooling & Scripts**: All octoDNS repositories follow the GitHub **"Scripts to Rule Them All"** pattern, housing all environment setup, testing, formatting, and other developer tools within the `./script/` directory of each repository.

---

### Development Workflow

Follow this step-by-step workflow when contributing changes:

#### 1. Create a Branch

Always start work by creating a new feature or bugfix branch:

```bash
git checkout -b <branch-name>
```

#### 2. Verify Compliance

Before committing, you must verify your changes comply with the repository standards by running the following scripts from the root of the repository you are modifying:

- **Test Suite**: Run the unit tests.

  ```bash
  ./script/test
  ```

- **Code Coverage**: Verify that code coverage is sufficient (typically 100%).

  ```bash
  ./script/coverage
  ```

  If code coverage is less than 100%, you can use the helper script `./script/coverage-report` to identify the exact files, line number ranges, and branches that are missing coverage:

  ```bash
  ./script/coverage-report
  ```

  > [!TIP]
  > When working in other `octodns-` repos, you can copy or run the `script/coverage-report` utility from the core repository against their generated `coverage.json` file. It outputs parsed information about coverage gaps that is much easier for AI agents and developers to digest and act on than raw JSON/XML reports.

- **Linting**: Ensure code conforms to Python style constraints.

  ```bash
  ./script/lint
  ```

- **Formatting**: Format the code automatically to match repository standards.

  ```bash
  ./script/format
  ```

#### 3. Create a Changelog Entry (First Commit)

The first commit on a branch must contain a changelog entry. Note that you should stage your changes prior to running the changelog command so that they are included in this first commit. Use the changelog tool to create one:

```bash
./script/changelog create --type <type> --commit "Brief description of changes"
```

##### Command Options & Arguments

- **`change-description`** (positional): A short, single-line description of the changes, suitable as an entry in `CHANGELOG.md`. Can include simple markdown formatting and links.
- **`-t, --type {none,patch,minor,major}`**: The scope of the change:
  - `patch`: This is a bug fix.
  - `minor`: Adds new functionality/changes in a fully backwards-compatible way.
  - `major`: Substantial new functionality and/or breaking changes.
  - `none`: This change does not need to be mentioned in the changelog.
- **`-a, --add`**: Run `git add` automatically on the newly created changelog entry.
- **`-c, --commit`**: Run `git commit` to stage and commit the entry (and other staged changes) using the same description.
- **`--continue`**: Continue a previously failed commit attempt.

*Example:*

```bash
./script/changelog create --type patch --add --commit "Fix DNS record parser bug"
```

#### 4. Subsequent Commits

For any subsequent commits on the same branch, use `git commit` normally:

```bash
git commit --message "Commit message"
```

#### 5. Push and Set Upstream

Push your branch to the remote repository and set the upstream branch:

```bash
git push --set-upstream origin <branch-name>
```

#### 6. Create a Pull Request

Use the GitHub CLI (`gh`) to create a pull request:

```bash
gh pr create --title "<title>" --body "<body>" --assignee "@me"
```

##### Common Parameters

- **`-t, --title <string>`**: Title of the pull request.
- **`-b, --body <string>`**: Body description of the pull request.
- **`-d, --draft`**: Mark the pull request as a draft (useful for work-in-progress).
- **`-f, --fill`**: Automatically use the commit title and description.
- **`-a, --assignee login`**: Assign the pull request to people by their login (use `"@me"` to self-assign).

*Example:*

```bash
gh pr create --title "Fix DNS record parser bug" --body "Fixes a bug in the record parser. /cc #123 Fix DNS record parser bug" --assignee "@me"
```

##### PR Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [octodns/octodns](https://github.com/octodns/octodns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
