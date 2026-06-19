---
trigger: always_on
description: Production Daml on Canton Network — templates, choices, interfaces, propose/accept, token-standard implementations, transfers, subscriptions, locking, upgrades, exceptions, testing, and the local verify loop. Use when writing or modifying .daml files, designing Daml workflows, implementing CIP token-standard interfaces (Holding/Transfer/Allocation/Metadata), or building Splice/Canton applications.
---


# Writing Daml the Splice way

You are writing Daml that will run on Canton. The reference for "production-grade Daml" is the Hyperledger Splice codebase (Amulet, Wallet, DSO governance, ANS, Splitwell) and the Canton Network token-standard interfaces. **Read them when in doubt — don't guess.**

## Setup: locate the reference corpus

Run this once at the start of any session where you might need to consult the reference repos (idempotent, ~50 ms when repos exist, ~60 s on first clone). The script works on macOS, Linux, and Windows — it uses Python 3 and `git`, both of which are present on any Daml dev machine.

```bash
# macOS / Linux
python3 "${CLAUDE_SKILL_DIR}/scripts/ensure_refs.py"
```

```powershell
# Windows (PowerShell)
python "$env:CLAUDE_SKILL_DIR\scripts\ensure_refs.py"
```

Capture the printed path as `$DAML_REFS` and use it as the base for every subsequent `grep` or `Read`. Resolution order: `$DAML_REFS` env var → `~/Developer/daml/` → OS-appropriate cache (`~/.cache/daml-claude-skill/refs` on mac/Linux, `%LOCALAPPDATA%\daml-claude-skill\refs` on Windows). On first use in the cache branch, `canton`, `splice`, and `splice-wallet-kernel` are cloned shallow from GitHub.

Every file:line citation in this skill and its references is relative to that base, so `splice/daml/splice-amulet/daml/Splice/Amulet.daml:185` means `$DAML_REFS/splice/daml/splice-amulet/daml/Splice/Amulet.daml` at line 185.

The reference subdirectories under `$DAML_REFS`:

- `splice/daml/` — production app templates (Amulet, Wallet, DSO, ANS, Splitwell)
- `splice/token-standard/` — CIP interface contracts (HoldingV1, TransferInstructionV1, AllocationV1, MetadataV1, AllocationInstructionV1, AllocationRequestV1, BurnMintV1)
- `canton/` — Canton platform; canonical for upgrades, exceptions, interface tests
- `splice-wallet-kernel/` — TS wallet that exercises these contracts

Before writing non-trivial Daml, **grep these repos for an analogous contract and copy its shape**. The Splice authors have made a lot of subtle decisions; reproducing them is almost always right.

## The five rules

1. **Don't invent, mimic.** A transfer? Read `Splice/Wallet/TransferOffer.daml` and `Splice/Amulet/TwoStepTransfer.daml`. A subscription? `Splice/Wallet/Subscriptions.daml`. A token? `Splice/Amulet.daml`. A factory? `Splice/Api/Token/TransferInstructionV1.daml`.
2. **Avoid contract keys.** Splice uses contract IDs and ACS queries exclusively. Default answer: no key.
3. **One template per state-machine phase.** Don't archive-and-recreate to add a signatory. Model `Offer → Accepted → Completed` as separate templates with widening signatory sets.
4. **Choices return named records, never tuples.** This keeps result types forward-compatible across upgrades.
5. **Implement the token standard interfaces.** Don't invent your own asset/transfer/allocation/metadata interface — implement `HoldingV1` / `TransferInstructionV1` / `AllocationV1` / `MetadataV1` / `BurnMintV1`.

Plus three smaller hard rules:
- `Decimal`, never `Numeric n`, for amounts.
- `getTime` is for **decisions** in choice bodies, never for **field values**. Bind it once per choice and reuse.
- Always `fetchChecked` (`Splice.Util`), never raw `fetch`.

## Verify locally — non-negotiable

Every time you write or edit a `.daml` file:

```bash
cd <package-with-daml.yaml>
daml build       # must succeed; fix every error before continuing
daml test        # must pass; covers all Scripts in the package
```

A change is **not done** until both pass. If `daml` isn't installed: `curl -sSL https://get.daml.com/ | sh -s -- --install-with-hash`, then `export PATH="$HOME/.daml/bin:$PATH"`. If the SDK version doesn't match the project's `daml.yaml` pin: `daml install project`. Full setup notes: [`references/local-setup.md`](references/local-setup.md).

Splice's repos pin SDK `3.3.0-snapshot.20250502.13767.0.v2fc6c7e2`. If you're working in a Splice subproject, run from inside the package directory so `daml.yaml` is picked up. The full SBT/nix outer build is *not* the loop you want — `daml build && daml test` in the package is.

## Package layout

```
{project}-v1/
├── daml.yaml
└── daml/
    └── {Project}/
        └── V1/
            ├── Token.daml                    -- one template per file
            ├── TokenInstrument.daml
            ├── TokenFactory.daml
            ├── TokenTransferFactory.daml
            ├── TokenBurnMintFactory.daml
            ├── TokenTransferInstruction.daml
            ├── TokenAllocation.daml
            ├── Token/
            │   ├── StandardInterfaces.daml   -- project-specific interfaces
            │   └── Util.daml                 -- helpers + cross-cutting types
            ├── CantoryRules.daml
            ├── CantoryLicensedFactory.daml
            ├── PendingLicensePayment.daml
            └── CantoryProxy.daml

{project}-v1-test/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Swofty-Developments/daml-claude-skill](https://github.com/Swofty-Developments/daml-claude-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
