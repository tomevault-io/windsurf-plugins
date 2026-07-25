---
trigger: always_on
description: This repository is a **Solidity smart contract library**: reusable `abstract contract` and `library` implementations meant to be inherited and extended by downstream projects. It is not a deployed product. Most conventions exist because code here is a base for others: functions must be overridable, argument types must not constrain inheritors, and the public API must not force a particular call context on child contracts.
---

# openzeppelin-contracts — Claude project guide

This repository is a **Solidity smart contract library**: reusable `abstract contract` and `library` implementations meant to be inherited and extended by downstream projects. It is not a deployed product. Most conventions exist because code here is a base for others: functions must be overridable, argument types must not constrain inheritors, and the public API must not force a particular call context on child contracts.

Read [`GUIDELINES.md`](./GUIDELINES.md) first — it is the human contributor spec and authoritative. This file is for what is specific to Claude-assisted work, plus the conventions not fully documented elsewhere.

Before preparing a contribution, also read [`CONTRIBUTING.md`](./CONTRIBUTING.md) thoroughly. Non-trivial changes must be discussed in an issue before a PR is opened.

## Repo map

| Path                    | Purpose                                                                                                                                        |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `contracts/`            | Library source. Subdirectory layout (`token/ERC20/`, `access/`, `utils/`, …) mirrors the public API.                                           |
| `contracts/mocks/`      | Mocks used by tests. Mirrors `contracts/`. Manual mocks only — see the `testing` skill.                                                        |
| `contracts/interfaces/` | ERC interfaces in their pristine spec form.                                                                                                    |
| `test/`                 | Hardhat + Chai tests (`.test.js`), shared behaviors (`.behavior.js`), and Foundry tests (`.t.sol`).                                            |
| `contracts-exposed/`    | Auto-generated `$ContractName` wrappers from `hardhat-exposed`. Gitignored. Do not edit.                                                       |
| `fv/`                   | Certora specs (`fv/specs/*.{conf,spec}`), harnesses, and `make`-applied patches.                                                               |
| `scripts/generate/`     | Procedurally generated `.sol` files (Arrays variants, Checkpoints, EnumerableSet, …). Source of truth is the template, not the generated file. |
| `scripts/checks/`       | CI checks: `inheritance-ordering.js`, `pragma-validity.js`, generation diff, storage layout.                                                   |
| `docs/`                 | AsciiDoc sources used by the OZ documentation site. Per-module `README.adoc` files in `contracts/**/` are also rendered.                       |
| `.changeset/`           | Per-PR changelog entries consumed by the release workflow.                                                                                     |
| `audits/`               | Historical third-party audit reports, one per release. Reference only.                                                                         |
| `.claude/skills/`       | Skills specific to working in this repository, loaded by AI assistants per task. See `CONTRIBUTING.md`.                                        |

## Commands

| Task                                     | Command                                                                 |
| ---------------------------------------- | ----------------------------------------------------------------------- |
| Compile                                  | `npm run compile`                                                       |
| Hardhat tests                            | `npm test`                                                              |
| Foundry tests (includes Halmos symbolic) | `forge test -vvv`                                                       |
| Lint (JS + Sol)                          | `npm run lint` / `npm run lint:fix`                                     |
| Coverage                                 | `npm run coverage`                                                      |
| Inheritance order check                  | `npm run test:inheritance`                                              |
| Pragma validity check                    | `npm run test:pragma`                                                   |
| Generated-file freshness check           | `npm run test:generation`                                               |
| Regenerate procedural contracts          | `npm run generate`                                                      |
| Add a changelog entry                    | `npx changeset add`                                                     |
| Run a single Certora spec                | `node fv/run.js <SpecName>` (apply harnesses first: `make -C fv apply`) |

## What linting via solhint already enforces (don't re-state in code review)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenZeppelin/openzeppelin-contracts-upgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
