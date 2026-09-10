---
trigger: always_on
description: A NosTale server reimplementation. .NET 10, EF Core + Npgsql + NodaTime, Autofac,
---

# NosCore

A NosTale server reimplementation. .NET 10, EF Core + Npgsql + NodaTime, Autofac,
Arch.Core ECS, Wolverine for messaging, source generators, `TreatWarningsAsErrors`.

---

## Code conventions

**Comments are the exception.** Write one only when the *why* cannot be inferred from names
and types, and keep it to a line or two. Do not restate what the code says, narrate how a
bug was found, or record where a value came from. Do not describe what changed — that is
the commit message's job.

**English only** — code, comments, commit messages, PR bodies, test names.

**No UTF-8 BOM on `.cs` files.** Some editors add one on save. Check before pushing:

```bash
for f in $(git diff --name-only origin/master...HEAD | grep '\.cs$'); do
  [ "$(head -c 3 "$f" | xxd -p)" = "efbbbf" ] && echo "BOM: $f"
done
```

**No `ConfigureAwait`.** Every host here is a generic-host console app, so there is no
synchronization context to come back to and `ConfigureAwait(false)` does nothing but add
noise to the line. Do not add it; the calls still in the tree are not a precedent.

**Keep semantic types.** Do not flatten a `bool` or an enum to a number because the wire
value happens to be `0` or `1`; the serializer handles the conversion.

**The serializer owns the wire format.** Never massage a value to fit it: no
`Replace(' ', '^')` before assigning a field, no manual padding, no hand-built separator. A
field that needs escaping says so on the packet property - `EscapeSpaces` for the one that
ends the line - so the fix belongs in `NosCore.Packets` and ships as a version bump. A
`Replace` here to make a packet come out right is a defect, including as a stopgap while
the package catches up.

**Ship only what is wired.** No placeholder enums, handlers or UI for behaviour that does
not exist yet. A data file listing a field is not a reason to expose it.

---

## Where things belong

NosCore is a main repo plus independently versioned NuGet packages, each in its own repo
under `NosCoreIO`. **A fact belongs in exactly one place.** Choosing the convenient place
over the correct one is the most common reason a change is sent back.

### Sibling packages

| Package | Owns |
|---|---|
| `NosCore.Packets` | Every packet class and the serializer — all wire-format knowledge |
| `NosCore.Algorithm` | Every stat and progression formula or table |
| `NosCore.Shared` | I18N, logging, cross-cutting enums, config primitives |
| `NosCore.Dao` | Data-access abstraction (`IDao<,>`) |
| `NosCore.Networking` | Session and socket plumbing |
| `NosCore.PathFinder` | Brushfire, flow field, jump point search |
| `NosCore.Analyzers` | Roslyn analyzers |

`NosCore.Algorithm` holds one service per curve — experience, HP/MP, damage, defence,
dodge, hit rate, reputation and so on. **Any number that scales with a level or a stat
belongs there**, as `I<Thing>Service` + `<Thing>Service`, with the ceiling in `Constants.cs`
and an approval table in its `DocumentationTest`. Do not hand-roll a lookup table inside
`NosCore.GameObject`.

### Projects in this repo

| Project | Owns | Must not contain |
|---|---|---|
| `NosCore.Data` | DTOs, enumerations, `.resx` language resources | game logic |
| `NosCore.Database` | EF entities and migrations | game logic |
| `NosCore.GameObject` | ECS bundles/components, game services, event handlers | wire formats, stat curves |
| `NosCore.PacketHandlers` | One handler per client packet | rules that belong in a service |
| `NosCore.Parser` | `.dat` ingestion and documentation generation | — |
| `NosCore.Core` | Shared infrastructure | — |

`NosCore.Data` being data-only and `NosCore.GameObject` holding the logic is deliberate.

### ECS layering

The entity model is Arch components underneath, generated bundles on top — this split is
the decided design, not a migration in flight:

- **Components** (`Ecs/Components/`) own ALL entity state. A new piece of per-entity
  state goes into a component (or a new component), never into a bundle body, a service
  dictionary keyed by entity, or a static.
- **Bundles** (`[ComponentBundle]` partial structs) are the generated facade the rest of
  the code reads and writes. Hand-written bundle members are computed views only —
  no backing fields.
- **Extension methods** (`Ecs/Extensions/`) are the per-entity behaviour layer; they act
  on one entity through its bundle.
- **Systems** (`Ecs/Systems/`) are for iteration-heavy queries over many entities.
  Prefer a system over LINQ across a bundle list when the call site runs per tick.
- **Hot paths do not materialise bundle lists.** `MapInstance.Monsters`/`Npcs` allocate a
  fresh `List` per access; anything called from the map life loop enumerates the backing
  dictionaries or a system query instead.

### Deciding, in order

1. A **wire shape** — field order, separator, sentinel? → `NosCore.Packets`, evidenced by a
   packet trace.
2. A **number that scales with a level or stat**? → `NosCore.Algorithm`.
3. The **meaning of a `.dat` column or a BCard subtype**? → an enum in `NosCore.Data`,
   wired by `NosCore.Parser`. Game data belongs in the database, not in a generated C#
   table — a table cannot be regenerated when the client updates.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NosCoreIO/NosCore](https://github.com/NosCoreIO/NosCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
