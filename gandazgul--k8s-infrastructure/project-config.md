---
trigger: always_on
description: This file provides comprehensive guidelines for AI coding agents operating within this GitOps repository. This
---

# Agent Instructions: k8s-infrastructure

This file provides comprehensive guidelines for AI coding agents operating within this GitOps repository. This
repository utilizes Flux v2 for Kubernetes deployment alongside Node.js-based container tooling. Agents must read and
adhere to these rules when analyzing, modifying, or creating code in this project.

## 1. Build, Lint, and Test Commands

### 1.1 Linting

The codebase relies on `pre-commit` hooks for validation. Agent tasks should ensure code passes these pre-commit checks
before completing operations.

- **Run all pre-commit hooks:**
  ```bash
  pre-commit run --all-files
  ```
- **Lint a specific file:**
  ```bash
  pre-commit run --files path/to/file.yaml
  ```
- **YAML specific linting (recommended prior to committing):**
  ```bash
  yamllint path/to/file.yaml
  ```

If formatting errors occur, rely on `end-of-file-fixer` and `trailing-whitespace` automatic corrections by running the
hook again.

### 1.2 Container Builds

Custom Docker containers reside under the `containers/` directory and use Node.js scripts for orchestration. Always run
`npm install` in the project root if packages are missing.

- **Build a specific image:**
  ```bash
  node containers/container-build.js --image=<directory-name>
  ```
- **Run a specific image locally:**
  ```bash
  node containers/container-run.js --image=<directory-name>
  ```

### 1.3 Testing and Validation

Currently, the codebase does not utilize an automated programmatic test suite (like Jest or Go tests). Testing is
achieved by statically validating Kubernetes YAML manifests and ensuring Bash/JS scripts execute cleanly.

- **Dry-run a single Kubernetes manifest:**
  ```bash
  kubectl apply --dry-run=client -f path/to/manifest.yaml
  ```
- **Validate Flux Kustomizations:**
  ```bash
  kustomize build path/to/directory
  ```
- **Testing Container Scripts:** Run the build/run scripts on sample directories and ensure they return an exit code
  `0`. Implement `process.exit(1)` on errors for any new utility scripts to ensure pipelines fail securely.

## 2. Code Style Guidelines

### 2.1 Kubernetes & Flux Manifests (YAML)

- **Formatting:** Strictly 2 spaces for indentation. Never use tabs. Arrays should be aligned with their parent keys.
- **Document Separation:** Always use `---` to separate multiple Kubernetes objects within the same `.yaml` file.
- **Naming Conventions:**
    - Standard Kubernetes resources (deployments, services, PVCs) and namespaces should use `lower-kebab-case`.
    - App definition files inside the `apps/` directory currently use `PascalCase.yaml` (e.g., `HomeAssistant.yaml`,
      `Paperless.yaml`). Adhere to this convention for new app manifests to maintain consistency.
    - Core infrastructure files generally use `PascalCase`, although some folders are using kebab-case. New files should
      follow `PascalCase`.
- **Flux Architecture Conventions:**
    - **Applications:** Deployments go in the `apps/` directory and should primarily utilize the apps/generic deployment
      configured as an overlay. If an official helm chart exists then use a `HelmRelease` kind. Group `HelmRepository`
      definitions at the top of the file if they don't already exist globally.
    - **Infrastructure:** Core infrastructure belongs in `infrastructure/` and its subdirectories (like `kube-system/`,
      `monitoring/`, etc.).
    - **Storage:** Avoid using `emptyDir` for application data that requires persistence. This cluster heavily utilizes
      hostpath provisions mapping to the local "YASR" volume structure. Reference
      `infrastructure/storage/hostpath-provisioner/README.md`.

### 2.2 JavaScript (Container Build Tools)

- **Environment:** Node.js CommonJS format (use `const module = require('module')`, do not use ES6 `import`).
- **Formatting:**
    - Use 4 spaces for indentation inside `.js` files.
    - add `else`, `catch`, and `finally` blocks on new lines for better readability.
    - Add a new line before return statements and before control flow statements (`if`, `for`, `while`).
- **Naming:**
    - Use `camelCase` for variables and function names (`imageName`, `fullImageName`).
    - Use `UPPER_SNAKE_CASE` for global constants or environment toggles.
- **Error Handling:** Use `try...catch` blocks for system or shell operations (`execSync`, `process.chdir`). Ensure
  error logs are well-prefixed with the context. Example:
  ```javascript
  try {
      process.chdir(`containers/${imageName}`);
  }
  catch (err) {
      console.error('CONTAINER:BUILD', 'Chdir failed', err);
      process.exit(1);
  }
  ```
- **Dependencies:** Keep external dependencies to an absolute minimum. Use native `child_process` and `fs` modules where
  possible. Currently, `minimist` is the only argument parser.

### 2.3 Shell Scripts (.sh)

- **Shebang:** Always start with `#!/bin/bash`.
- **Variables:** Use curly braces for variables to prevent expansion errors (e.g., `${VARIABLE}`).
- **Safety:** Use `set -e` at the top of scripts to ensure they fail fast on error.
- **Paths:** Keep shell script footprints minimal; use exact paths or rely on `process.cwd()` consistency when scripts
  are invoked from the root.

## 3. Security and Secrets Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gandazgul/k8s-infrastructure](https://github.com/gandazgul/k8s-infrastructure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
