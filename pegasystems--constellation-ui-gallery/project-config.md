---
trigger: always_on
description: Instructions for coding agents (GitHub Copilot coding agent, Cursor, and similar) working in this repository.
---

# AGENTS.md — Constellation UI Gallery

Instructions for coding agents (GitHub Copilot coding agent, Cursor, and similar) working in this repository.

This gallery ships **Constellation DX components** that must work on **Pega Platform and Launchpad** unless a component is explicitly marked unsupported. Prefer existing gallery patterns over inventing new ones.

---

## Read these docs first

| Doc                                                                                                                                                                  | When to read                                                                                                             |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| [Component_Build_Guide.md](./Component_Build_Guide.md)                                                                                                               | Scaffolding a component, folder layout, `config.json`, Storybook/tests, Launchpad support column                         |
| [best practices.md](./best%20practices.md)                                                                                                                           | General Constellation DX design and development rules                                                                    |
| [LAUNCHPAD_VS_PLATFORM.md](./LAUNCHPAD_VS_PLATFORM.md)                                                                                                               | **Required** before any change that touches data APIs, case IDs, property keys, rule names, navigation, or local actions |
| [README.md](./README.md)                                                                                                                                             | Install, Storybook, lint, test, and publish overview                                                                     |
| [.github/skills/pega-dx-component-builder/SKILL.md](./.github/skills/pega-dx-component-builder/SKILL.md)                                                             | End-to-end procedure for creating or extending a component                                                               |
| [.github/skills/pega-dx-component-builder/references/repository-guidance.md](./.github/skills/pega-dx-component-builder/references/repository-guidance.md)           | Repo-specific conventions                                                                                                |
| [.github/skills/pega-dx-component-builder/references/official-pega-guidance.md](./.github/skills/pega-dx-component-builder/references/official-pega-guidance.md)     | PCore / PConnect public API summary                                                                                      |
| [.github/skills/pega-dx-component-builder/assets/component-delivery-checklist.md](./.github/skills/pega-dx-component-builder/assets/component-delivery-checklist.md) | Pre-merge delivery checklist                                                                                             |
| [.github/agents/pega-dx-component-builder.agent.md](./.github/agents/pega-dx-component-builder.agent.md)                                                             | GitHub agent persona for DX component work                                                                               |

Closest existing component under `src/components/Pega_Extensions_<Name>/` is also a primary source of truth. Reuse its structure before introducing a new pattern.

---

## Non-negotiable rules

1. **Do not invent Pega APIs.** Verify against repo usage or official docs.
2. **Prefer Cosmos.** Use `@pega/cosmos-react-core` and existing dependencies; do not add third-party UI libraries.
3. **Source only by default.** Edit `src/` (and docs). Do not hand-edit generated `Pega_Extensions/` or `store/` artifacts unless explicitly asked.
4. **Launchpad-safe by default.** Components that use data, actions, or case identity must follow [LAUNCHPAD_VS_PLATFORM.md](./LAUNCHPAD_VS_PLATFORM.md):
   - Use `getMappedKey` from `src/components/shared/utils.ts` for property names, data-page names, and local-action / flow-type names.
   - Do **not** hard-code `pyID`, `pzInsKey`, `pxObjClass`, `pyStatusWork`, `pyLabel`, etc. as runtime object keys.
   - Read case context via `PCore.getConstants().CASE_INFO.*`, not hard-coded paths like `caseInfo.businessID`.
   - Navigate with `getActionsApi()` and `PCore.getSemanticUrlUtils()` — never hand-build URLs.
   - Gate optional Platform-only REST DX APIs with `PCore.getRestClient().doesRestApiExist('…')`, or mark the component unsupported on Launchpad.
   - Do **not** use `if (isLaunchpad)` when key mapping or capability detection can solve the difference.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pegasystems/constellation-ui-gallery](https://github.com/pegasystems/constellation-ui-gallery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
