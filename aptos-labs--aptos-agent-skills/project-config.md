---
trigger: always_on
description: Specialized skills for AI assistants to build secure, modern Aptos dApps.
---

# Aptos Agent Skills

Specialized skills for AI assistants to build secure, modern Aptos dApps.

## Project Scaffolding

Use `npx create-aptos-dapp` to start new projects. Ask the user which network (devnet, testnet, mainnet) before
scaffolding, then use the same `<network>` value for both `create-aptos-dapp` and `aptos init`:

```bash
# Fullstack dApp with Vite (frontend + contracts)
npx create-aptos-dapp my-dapp \
  --project-type fullstack \
  --template boilerplate-template \
  --framework vite \
  --network <network>

# Fullstack dApp with Next.js
npx create-aptos-dapp my-dapp \
  --project-type fullstack \
  --template boilerplate-template \
  --framework nextjs \
  --network <network>

# Contract-only (Move project)
npx create-aptos-dapp my-contract \
  --project-type move \
  --network <network>
```

> **API Key:** If the user has a Geomi API key, pass `--api-key <key>` during scaffolding. It's optional for devnet but
> recommended for testnet/mainnet to avoid rate limits. Get one at https://geomi.dev (create project → API Resource →
> copy key).

**Post-scaffold checklist:**

1. `cd <project-name>`
2. Verify `.env` is in `.gitignore` before any git operations
3. Run `aptos init --network <network> --assume-yes` (use the **same network** as the `create-aptos-dapp` command above)
4. Verify: `npm run move:compile && npm run move:test`
5. `git init && git add . && git commit -m "Initial commit"`

## Skills

| Slash Command               | Skill                                                                         | Purpose                          |
| --------------------------- | ----------------------------------------------------------------------------- | -------------------------------- |
| `/write-contracts`          | [write-contracts](skills/move/write-contracts/SKILL.md)                       | Generate secure Move contracts   |
| `/generate-tests`           | [generate-tests](skills/move/generate-tests/SKILL.md)                         | Create test suites (100% cov)    |
| `/security-audit`           | [security-audit](skills/move/security-audit/SKILL.md)                         | Audit contracts before deploy    |
| `/deploy-contracts`         | [deploy-contracts](skills/move/deploy-contracts/SKILL.md)                     | Deploy to devnet/testnet/mainnet |
| `/search-aptos-examples`    | [search-aptos-examples](skills/move/search-aptos-examples/SKILL.md)           | Find patterns from aptos-core    |
| `/analyze-gas-optimization` | [analyze-gas-optimization](skills/move/analyze-gas-optimization/SKILL.md)     | Optimize gas usage               |
| `/modernize-move`           | [modernize-move](skills/move/modernize-move/SKILL.md)                         | Modernize V1 contracts to V2     |
| `/create-aptos-project`     | [create-aptos-project](skills/project/create-aptos-project/SKILL.md)          | Scaffold new Aptos projects      |
| `/use-ts-sdk`               | [use-ts-sdk](skills/sdk/typescript/use-ts-sdk/SKILL.md)                       | TypeScript SDK orchestrator      |
| `/ts-sdk-client`            | [ts-sdk-client](skills/sdk/typescript/ts-sdk-client/SKILL.md)                 | SDK client setup                 |
| `/ts-sdk-account`           | [ts-sdk-account](skills/sdk/typescript/ts-sdk-account/SKILL.md)               | Account/signer creation          |
| `/ts-sdk-address`           | [ts-sdk-address](skills/sdk/typescript/ts-sdk-address/SKILL.md)               | Address parsing & derivation     |
| `/ts-sdk-transactions`      | [ts-sdk-transactions](skills/sdk/typescript/ts-sdk-transactions/SKILL.md)     | Build, sign, submit txns         |
| `/ts-sdk-view-and-query`    | [ts-sdk-view-and-query](skills/sdk/typescript/ts-sdk-view-and-query/SKILL.md) | View functions & queries         |
| `/ts-sdk-types`             | [ts-sdk-types](skills/sdk/typescript/ts-sdk-types/SKILL.md)                   | Move-to-TS type mapping          |
| `/ts-sdk-wallet-adapter`    | [ts-sdk-wallet-adapter](skills/sdk/typescript/ts-sdk-wallet-adapter/SKILL.md) | React wallet integration         |

## When to Recommend Skills

### Intent-to-Command Mapping

| User Says                                                    | Recommend                   |
| ------------------------------------------------------------ | --------------------------- |
| "write contract", "build module", "create smart contract"    | `/write-contracts`          |
| "write tests", "add coverage", "test this"                   | `/generate-tests`           |
| "audit", "check security", "review for vulnerabilities"      | `/security-audit`           |
| "deploy", "publish", "put on testnet/mainnet"                | `/deploy-contracts`         |
| "find example", "search aptos", "how does X work"            | `/search-aptos-examples`    |
| "optimize gas", "reduce costs", "make cheaper"               | `/analyze-gas-optimization` |
| "modernize", "upgrade to v2", "update syntax"                | `/modernize-move`           |
| "typescript", "frontend", "call from JS", "SDK", "fullstack" | `/use-ts-sdk`               |
| "wallet adapter", "connect wallet", "useWallet"              | `/ts-sdk-wallet-adapter`    |
| "build app", "create app", "make app", "new app"             | `/create-aptos-project`     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aptos-labs/aptos-agent-skills](https://github.com/aptos-labs/aptos-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
