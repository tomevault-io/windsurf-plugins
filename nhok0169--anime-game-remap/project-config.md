---
trigger: always_on
description: **The whole `regEdits` family moves a `KVP` around; until this, none of them could turn a value into
---

# Ini Graph Editing

## `RegNewVals` can write a value computed FROM the old one (2026-09-29)

**The whole `regEdits` family moves a `KVP` around; until this, none of them could turn a value into
another value the caller already knows.** `RegRemap` moves the KEY (its `RemappedKeyData` may *test*
the value and cannot change it), `RegRemove` drops one, `RegNewVals` wrote a value that does not
depend on the old, and `RegAssetRemap` maps a value through `ModMappedAssets` -- the right answer for
a hash or an index and no help for anything else.

So the thing a fixer actually keeps needing -- *bind the edited copy of whatever this slot was
already bound to* -- had nowhere to go, and the WuWa fixer was doing it by **rendering the section to
text and rewriting the lines**.

**The gap was one argument wide, so it is a new alternative rather than a new class.**
`RegNewVals::ValProducer` is `(modType) -> V`; `OldValProducer` is `(oldValue, modType) -> V`, added
to the `NewVal` variant. That is the same widening `ModTypePredicate` already is over
`IfContentPart::Predicate`, and the class's own header already explained why: a register edit always
knows the `ModType` it is running for, which is what it is doing in the fixer layer rather than being
a plain `replaceVals` call.

Three things to know before reaching for it:

- **It is additive.** Every existing spec form is untouched and still goes through
  `IfContentPart::replaceVals`; only a key whose spec *holds* an `OldValProducer` takes the new path,
  which resolves it per occurrence with `getValsWithInds` / `setValByInd`. `RegNewVals_OldValProducer_test.cpp`
  pins that explicitly, because "the other forms did not move" is the whole argument for extending
  rather than writing a fifth class.
- **The three spec forms mean exactly what they mean without it** -- a bare spec writes every
  occurrence, a list is positional and an entry past the end is unused, a conditional writes what its
  predicate accepts -- and an **occurrence** past the end of a list is left alone. That last half is
  the one that can go wrong: an implementation that clamps to the last entry gets everything else
  right, and the test only caught it once a case with more occurrences than entries was added.
- **It cannot remove.** A first draft of this was a separate `RegValRemap` whose producer returned
  `std::nullopt` to drop the `KVP`; that is `RegRemove`'s job, and `RegRemove` already takes a
  `RemoveKeyCheck` of `(true positional index, value)`, which can decide by value. Compose the two --
  removal first, then the rewrite -- rather than teaching one edit both.

From `Python` it is the `FromOldVal` marker (`PyRegNewVals.h`), the way `ReplaceList` and `ReplaceIf`
mark which of `replaceVals`' forms they are: a bare callable in a value slot already means
`newVal(modType)`, so the one that also reads the old value has to say so. `FromOldVal(f)` calls
`f(oldValue, modType)`. It is bound because a prototype is the oracle every WuWa remap is checked
against, and a compiled fix that can express something the prototype cannot breaks that.

## Rendering a section to text to edit it is the thing to look for (2026-09-29)

`WWMIFixer::bindLine` builds a `CommandList` that binds one texture role, for a role the mod toggles
between variants -- it copies the mod's own section and turns each `this = <resource>` into
`<register> = <resource>`. It used to do that by calling `renderIfTemplate`, `std::getline`ing the
result, finding the `=` in each line, skipping the one that starts with `[`, lowercasing the text
before the `=` to recognise a key, rewriting the text after it, and gluing a `[name]` header on the
front. **109 lines, and every one of them is the section model written out and read straight back
in**, in a file whose every other edit goes through that model.

The composition it is now:

| what it does | the module |
| --- | --- |
| copy the mod's section | `IfTemplate::deepcopy`, then set `name` and clear `prefix` |
| drop the matching keys, which mean nothing in a list | `RegRemove` |
| drop a branch naming a resource nothing declares | the same `RegRemove`, with a `RemoveKeyCheck` |
| `this = <res>` -> `this = <our edited copy>` | `RegNewVals` with an `OldValProducer` |
| `this` -> the register the target's draw reads | `RegRemap` |
| the `if` / `endif`, the indentation, the header | `renderIfTemplate`, once, at the end |

**One thing the string version had that a naive port loses: tolerance of case.** `THIS = ResourceX`
is legal 3dmigoto and the old code lowercased before comparing; a `regEdit` matches a key exactly. So
the rules are built per part from **the keys that part actually has**, matched case-insensitively,
and each edit is handed the exact spelling it found. A remap is only as robust as its worst
assumption about what a modder wrote, and this one costs four lines.

## An external command covers NOTHING -- `isKeyFullyCover` said it covered everything (2026-09-24)

`IfTemplate::isKeyFullyCoverNode` asks whether every path through a section binds a key, following
`run =` into the command lists the graph holds. A call to a name that is NOT a section of the graph

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nhok0169/Anime-Game-Remap](https://github.com/nhok0169/Anime-Game-Remap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
