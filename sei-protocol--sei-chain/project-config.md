---
trigger: always_on
description: `testutil/configtest` and `testutil/fuzzing` are the harness that pins how a seid
---

# Configuration Characterization

`testutil/configtest` and `testutil/fuzzing` are the harness that pins how a seid
node resolves configuration. The suites built on them record the current behavior
of the legacy configuration path as executable tests, including the parts of that
behavior nobody would choose on purpose.

The surface is worth pinning because it is large and mostly implicit. A value
reaches running code from four layers (an in-code default, a TOML file, an
environment variable, a cobra flag), resolved through several viper instances whose
environment prefixes differ, and it lands in two places the rest of the boot reads,
a Tendermint config struct and a flat key-value map. None of that was written down
anywhere a second implementation could be compared against, which is what these
tests supply. They exist so the SeiConfigManager work (PLT-775) can replace that
path and prove the replacement resolves every key the same way.

## Standing Rule

Inside the surface the suite covers, a change to how a configuration value is read,
defaulted, named, or cast is a change to a pinned contract, so the suite fails. That
failure is the review prompt. Record the new behavior and put the old and new value in
the diff, rather than loosening the assertion until it passes.

What that surface is has to be read alongside the rule, because the rule is
unconditional only inside it. A key added to a struct field some row already claims is
not caught, and nothing mechanical will prompt you (`Adding a Key to an Existing
Section`). A rename fails here and still has to be carried by hand into the app.toml
template, the flag registration and the documentation (`Renaming a Key`). And whole
classes of read sit outside the suite (`Out of Scope`). None of that softens the
paragraph above for the reads the suite does cover: there, the failure is not optional
and not something to route around.

Four ways of making a failure go away are wrong here, because each one turns a
visible change into an invisible one:

1. `t.Skip` on a row whose behavior changed. CI stays green and a skip line in
   verbose output is what nobody reads.
2. Widening an assertion to accept both the old and new value.
3. Deleting a row rather than updating it.
4. Editing a row's `Cast`, `Unguarded` or `Checked` until it matches a reader you
   changed, without having intended the behavior change. A row describes the reader,
   so editing one is correct only alongside a deliberate change to that reader in the
   same PR.

If a pinned behavior is genuinely wrong and worth fixing, fix it in the production
reader and update the row in the same PR. The row then records the improvement.

## Primitives

`CheckRow` is `CheckKey` plus `CheckDeterministic`, so there is one fewer property than there
are calls. A fuzz target names only `CheckRow` and gets both. The table below is the enumeration,
and `TestGuideListsEveryPrimitive` holds it to the exported surface.

| Check | The failure it prevents | Held against |
|---|---|---|
| `CheckDefaults` | a declared default moves with nothing independent to compare against | `testdata/<section>.golden` |
| `CheckKeyNames` | an operator-facing key is renamed while the row and the reader move together | `testdata/<section>.keys.golden` |
| `CheckKey` | a reader does not resolve `Key` into `Path` through `Cast` | the reader's own empty-`AppOpts` result, with the row's leaf spliced in |
| `CheckDeterministic` | a reader is not a pure function of its `AppOpts` | a second read of the same input |
| `CheckAbsent` | an omitted key resolves to something other than the declared default | the declared defaults struct |
| `CheckManifestCoversEveryField` | a resolved field no row claims | the manifest's `Path` and `AlsoWrites` entries |
| `CheckEveryRowHasADiscriminatingSeed` | a row whose every seed would also pass against a reader that never looks its key up | the recorded seed corpus |
| `CheckWiring` | one of the calls above is deleted | `testdata/wiring_coverage.txt` |

The third column is the spec, and it is the one to read before wiring anything. Three of these
compare against a checked-in file, one against the declared defaults, one against the reader's own
output, one against a second read of the same input, one against the manifest, and one against the
seeds the target declared. A check whose right-hand side comes from the same place as its left-hand
side holds for any reader.

Two of them read no prediction column, and that is the invariant any new check
inherits. `CheckKeyNames` is blind to `Path`, `Cast`, `Unguarded` and `Checked`, and
`CheckEveryRowHasADiscriminatingSeed` to `Cast`, `Unguarded` and `Checked`. A check that
read the column it exists to hold could be silenced by editing that column, which is
forbidden move 4 above.

`CheckKey` compares against the reader's own empty-`AppOpts` result rather than the
declared defaults, because some readers fill fields from outside the config.
`CheckAbsent` is what ties that result to the declared defaults, so a section wired for
rows and not for `CheckAbsent` has an unanchored baseline.

**Before adding one.** Advancing coverage is normally wiring an existing check to another

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sei-protocol/sei-chain](https://github.com/sei-protocol/sei-chain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
