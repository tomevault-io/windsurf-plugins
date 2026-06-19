---
trigger: always_on
description: >
---


# safe-programs

Three modes in one skill.

- **brainstorm** - design-time thinking partner. Surfaces architectural risks
  before code exists. Produces `design-notes.md` with captured decisions, open
  questions, and framework/testing choices.
- **build** - scaffold a new program (Anchor, Native Rust, or Pinocchio) with
  tests and a filled-out security checklist. Security baked in before the first
  line is written.
- **audit** - fan 8 parallel agents over existing program code, deduplicate,
  gate-validate, produce a findings report.

Same rules under the hood. Design decisions in brainstorm, enforcement in
build, hunting in audit.

---

## Pick the mode first

| Trigger | Mode |
|---|---|
| "thinking about building", "how would I design...", "should I use Anchor or Pinocchio for X", "what should I consider for Y", "brainstorm a solana program", "talk me through designing..." | **brainstorm** |
| "write a solana program", "scaffold", "build an anchor program", "create a native rust solana program", "help me write a program that does X on solana" | **build** |
| "audit this", "review for security", "check this program", "find the bugs", `/safe-programs`, `/safe-programs --deep`, `/safe-programs <file>` | **audit** |

If the ask is ambiguous, ask which one. Don't default.

The natural flow is **brainstorm → build → audit**. Brainstorm commits design
decisions, build scaffolds against them, audit hunts for what slipped through.
A user can enter at any mode.

---

# Brainstorm mode

Design-time thinking partner. Use this when the user is still figuring out what
to build, which framework to use, or whether a design will hold up. Surfaces the
architectural risks before code exists - the decisions that can't be patched
later by adding a line of code.

Produces a `design-notes.md` capturing decisions, alternatives, and open
questions. When the user is ready to scaffold, hand off to build mode with
those decisions pre-filled.

## Step 1 - understand the shape

Ask in one message, not a form. Just enough to know which risks apply:

- What does it do? (functionality, user flow)
- Scale - single user, multi-user, TVL potential
- Custody - holds SOL, holds tokens, holds NFTs, or no custody
- External integrations - CPIs to other protocols, oracles, token standards
- Time dynamics - time-locks, epochs, vesting, auctions, deadlines
- Constraints the user already knows about (team familiarity, audit budget, CU targets)

If the user's description already covers most of this, proceed and note any
assumptions.

## Step 2 - load references

Read `references/shared-base.md` fully. Brainstorm mode leans on the
architectural sections - decisions that are hard or impossible to undo post-deploy:

- §5.8 Defense-in-depth (global vault vs user-specific PDAs)
- §21 Reward accounting (especially §21.4 dead share price, §21.5 inflation attack, §21.7 reward source solvency)
- §22 Vault withdrawal paths
- §23 Token-2022 extension validation at init
- §24 Admin key rotation
- §25 BPF stack frame
- §26 State machine & lifecycle integrity
- §28 Bonding curve / AMM
- §29 Permissionless initialization
- §31.4 Treasury sweepability

If the user is leaning toward a specific framework, also load the matching
framework file (`references/anchor.md`, `references/native-rust.md`,
`references/pinocchio.md`). Don't load `references/litesvm.md` or the audit
references - those come later.

## Step 3 - classify risk tier and architectural category

Risk tier (same scale as build mode):

| Level | Criteria |
|---|---|
| 🟢 Low | no custody, no CPI, single user, read-heavy |
| 🟡 Medium | token transfers, basic CPI, multi-user state, PDAs |
| 🔴 Critical | vaults, multi-CPI chains, admin keys, large TVL |

Architectural categories (a program can hit multiple):

| Category | Maps to |
|---|---|
| Share-based pool (stX / totalStaked) | §21.4, §21.5 |
| Reward / yield system | §21 (whole section) |
| PDA-controlled token vault | §22, §31.4 |
| Multi-CPI chain | §5 (especially §5.8 defense-in-depth) |
| Admin key / privileged control | §24, §29 |
| Token-2022 mint consumption | §7, §23 |
| Bonding curve / AMM | §28 |
| Time-gated logic | §16, §26.6 |
| Permissionless creation | §29 |
| Large account contexts | §25 |

State the tier and matching categories back to the user.

## Step 4 - walk through the architectural risks

For each category that applies, surface the risks that are **architectural** -
can't be patched later without a migration. Frame each as a decision the user
needs to make, not a lecture.

Examples of the shape this should take:

- **Share-based pool:** dead share price requires the exchange-rate update path exists at design time (§21.4). Inflation attack requires dead-shares / min-deposit / virtual balances (§21.5) - pick one now, rewriting later means a new pool.
- **Global vault vs user-specific PDAs:** this is the blast radius of a CPI exploit (§5.8). User-specific PDAs contain a breach to one user. Global vault risks everything. Retrofit is a migration.
- **Admin key model:** single immutable key vs two-step rotation (§24.2) vs multisig vs timelocked multisig. Each has different recovery paths. Starting with single-key then migrating is painful.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meowyx/safe-programs](https://github.com/meowyx/safe-programs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
