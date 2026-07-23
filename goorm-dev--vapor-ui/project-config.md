---
trigger: always_on
description: This coding convention applies to files under `packages/core/**`.
---

# CLAUDE.md

This coding convention applies to files under `packages/core/**`.
It is an English translation and adaptation of [`.gemini/styleguide.md`](../../.gemini/styleguide.md), updated to match the current implementation in `packages/core`.

# 0. Review Response Style

- Provide code review responses in Korean.
- If repository-level AI tooling is configured to use a different review language, follow that tool configuration for the final output language.

# 1. Development Environment

---

- **Package Manager**: PNPM v10.5.1+
- **Node.js**: v20.19+
- **Linting & Formatting**: Use ESLint v9+ and Prettier v3.3+. IDE integration and Git Hook setup are recommended.

# 2. Git & Version Control

---

## 2.1. Branch Strategy

- Use semantic branch names that clearly describe the purpose of the change.
- Prefer branch names that map directly to the work being done, for example `add-coderabbit-settings`, `fix-dialog-focus`, or `docs-update-radio-group`.
- Do not assume Git Flow prefixes such as `feature/*`, `release/*`, or `hotfix/*` are required unless the repository workflow explicitly calls for them.
- Delete working branches after the PR is merged.

## 2.2. Commit Messages (Conventional Commits & SemVer)

Follow the [Conventional Commits specification](https://www.conventionalcommits.org/). This supports [SemVer](https://semver.org/) based versioning and automated changelog generation.

**Format:**

```bash
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

### 2.2.1. Primary Commit Types & SemVer Guidance

- **Important**: In this repository, package version bumps are decided by **Changesets** in `.changeset/*.md`, not inferred automatically from commit types alone.
- Commit types should communicate change intent clearly.
- When a package release is needed, the changeset level (`patch`, `minor`, `major`) must match the actual impact of the change.

- **`feat`**: Add a new feature, such as a component or prop. This will often pair with a **`minor` changeset** when it introduces new user-facing capability.

    Example:

    ```bash
    feat(Avatar): add new Avatar component
    feat(Button): implement iconPosition prop for icon placement
    ```

- **`fix`**: Fix a bug. This will often pair with a **`patch` changeset** when it corrects behavior without introducing a breaking API change.

    Example:

    ```bash
    fix(Input): correct placeholder color in disabled state
    fix(Modal): resolve issue where modal content is not scrollable
    ```

- **`BREAKING CHANGE`**: Introduce an API-breaking change. Add `!` after the type or specify `BREAKING CHANGE:` in the footer. This should pair with a **`major` changeset**.

    Example:

    ```text
    feat(Button)!: change 'kind' prop to 'variant' for clarity

    BREAKING CHANGE: Button component's `kind` prop has been renamed to `variant`.
    Migrate `kind="primary"` to `variant="solid"`.
    Migrate `kind="secondary"` to `variant="outline"`.
    ```

### 2.2.2. Other Commit Types

- `docs`: Documentation changes.
- `style`: Code style changes without logic changes.
- `refactor`: Refactoring without behavioral changes.
- `test`: Add or update tests.
- `perf`: Performance improvements.
- `chore`: Maintenance work such as build or package configuration.

- These commit types do **not** imply "no release". If the change affects published package behavior or API, add an appropriate changeset even when the commit type is `chore`, `refactor`, or another non-`feat`/`fix` type.

### 2.2.3. Scope (Optional)

- Specify the scope of the change, usually the component name.

### 2.2.4. Subject

- Use present tense and imperative mood.
- Start with a lowercase letter.
- Do not end with a period.
- Keep it within 50 characters when possible.

## 2.3. Pull Request (PR)

- Fill out the PR template thoroughly, including change summary, tests, and related issues.
- Include screenshots for UI changes when relevant.
- Add tests, documentation updates, and changesets when the change requires them.
- Request review from the appropriate maintainers or code owners according to the repository workflow.

# 3. Coding Style

---

## 3.1. Modules (Import & Export)

- **Use absolute paths**: Prefer `tsconfig.json` path aliases such as `~/*` over deep relative imports.

    ```tsx
    import { createContext } from '~/libs/create-context';
    ```

- **Alias primitive dependencies clearly**: Current components are primarily built on **Base UI**, and imports commonly use explicit aliases such as:

    ```tsx
    import { Button as BaseButton } from '@base-ui/react/button';
    import { Dialog as BaseDialog } from '@base-ui/react/dialog';
    ```

- **Prefer named exports** over `default export`.

- **Export prop types following the local module pattern**:
    - Many components expose namespace-scoped types such as `Button.Props`, `DialogRoot.Props`, and `RadioGroupRoot.Props`.
    - Some modules also export direct type names such as `PaginationRootProps`.
    - Preserve the existing public API style of the module instead of forcing a single naming convention across all components.

## 3.2. Naming Conventions

- **`camelCase`**: variables and functions
- **`PascalCase`**: components, interfaces, types, and enums
- **`CONSTANT_CASE`**: constants

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goorm-dev/vapor-ui](https://github.com/goorm-dev/vapor-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
