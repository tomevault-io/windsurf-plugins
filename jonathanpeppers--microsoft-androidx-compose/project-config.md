---
trigger: always_on
description: generates enums (e.g. `AndroidX.Compose.UI.State.ToggleableState`) as
---

# Microsoft.AndroidX.Compose — agent instructions

C#-only .NET-for-Android app hosting Jetpack Compose UI via official
`Xamarin.AndroidX.Compose.*` bindings. No Kotlin sources, no custom bindings,
no `[InterceptsLocation]`. Every C# composable either calls a generated binding
or a JNI bridge in `ComposeBridges.cs`.

See `README.md`, `docs/architecture.md`, `docs/compose-internals.md`,
`docs/NOTES.md` for background. This file is the rule set agents **must**
follow.

## Layout

- `src/Microsoft.AndroidX.Compose/` — public C# facade. **One class per file**, named
  after the class. `ComposeBridges.cs` holds raw-JNI bridges;
  `ComposeDefaults.cs` holds *only* assembly attributes driving the generator.
- `src/Microsoft.AndroidX.Compose.SourceGenerators/` — Roslyn incremental generators
  (`ComposeDefaultsGenerator`, `ComposeBridgeGenerator`,
  `ComposeFacadeGenerator`) emitting `[Flags]` enums for Kotlin `$default`
  bitmasks, JNI bridge bodies, and facade classes.
- `src/Microsoft.AndroidX.Compose.SourceGenerators.Tests/` — xUnit, no Android SDK needed.
- `src/Microsoft.AndroidX.Compose.Gallery/` — runnable Android app (on-device demo harness).

## Build / test / run

```pwsh
dotnet test  src/Microsoft.AndroidX.Compose.SourceGenerators.Tests   # generator unit tests
dotnet build src/Microsoft.AndroidX.Compose                  # facade only
dotnet build src/Microsoft.AndroidX.Compose.Gallery                  # full Android build (needs android workload)
dotnet build src/Microsoft.AndroidX.Compose.Gallery -t:Run           # deploy to device
```

Run generator tests on any change to `Microsoft.AndroidX.Compose.SourceGenerators` or
`ComposeDefaults.cs`. Run the facade build on any change to
`Microsoft.AndroidX.Compose`.

## Generated `$default` enums — DO NOT hand-roll

Every `@Composable` takes a trailing `int $default` bitmask: bit *N* set =
"param *N* not supplied; use Kotlin default." A `[Flags]` enum names each bit
so call sites read `(int)ButtonDefault.All` instead of magic numbers.

**Always generate via `ComposeDefaultsAttribute` in `ComposeDefaults.cs`. Never
hand-write `[Flags] enum FooDefault`.**

### Generic form — preferred when Kt method is bindable

```csharp
[assembly: ComposeDefaults<ColumnKt>("Column", "ColumnDefault")]
```

Generator picks the longest static overload, walks params up to first
`IComposer`, names each bit after the param (PascalCased), skips
`Kotlin.Jvm.Functions.IFunction*` slots (content lambdas — always provided),
emits an `All` constant.

### Declarative form — when Kt method is stripped

The binder strips overloads with mangled JVM names (`Text--4IGK_g`,
`Surface-T9BRK9s`, …) from Kotlin `@JvmInline value class` params (`Color`,
`Dp`, `TextUnit`, `FontWeight`, …). Until [dotnet/java-interop#1440] lands,
hand the generator the Kotlin parameter names:

```csharp
[assembly: ComposeDefaults("ButtonDefault",
    "!onClick", "modifier", "enabled", "shape", "colors",
    "elevation", "border", "contentPadding", "interactionSource", "!content")]
```

- Each name occupies one bit at its positional index.
- `!` prefix consumes the bit but emits no enum member (params the caller
  always provides — `onClick`, content lambdas, required values).
- Keep optional slot lambdas the caller toggles per-call (e.g. `AlertDialog`'s
  `dismissButton`/`icon`/`title`/`text`) *as enum members* — the call site
  clears the bit when supplying the slot.

### Wide masks (> 31 slots) — `: long` + `Split` helper

Kotlin lowers `@Composable` with > 32 defaultable params into a pair of `int`
`$default` slots (`II` in signature) — e.g.
`androidx.compose.material3.lightColorScheme` has 48 slots. When the slot list
exceeds 31 entries the generator switches to a `long`-backed enum and emits a
`Split` extension returning `(int Mask0, int Mask1)`:

```csharp
[Flags]
internal enum ColorSchemeDefault : long
{
    None = 0, Primary = 1L << 0, /* … */ Slot47 = 1L << 47,
    All  = Primary | /* … */,
}

internal static class ColorSchemeDefaultExtensions
{
    public static (int Mask0, int Mask1) Split(this ColorSchemeDefault value) =>
        ((int)((long)value & 0xFFFFFFFFL), (int)(((long)value >> 32) & 0xFFFFFFFFL));
}
```

Call sites build the mask normally (`defaults |= ColorSchemeDefault.Primary`)
and `.Split()` immediately before passing the two ints to JNI — never hand-roll
`1 << N` or the low/high split. Threshold is 32 slots (not 33) so bit 31 lands
as `1L << 31` rather than `int.MinValue`. ≤ 31 slots stay byte-for-byte
identical to pre-wide output. > 63-bit masks not modelled — add `ulong` if/when
a Compose API needs them.

When the upstream binder fix lands, swap a declarative attribute to generic
form one-for-one.

[dotnet/java-interop#1440]: https://github.com/dotnet/java-interop/pull/1440

### Generator diagnostics

| ID     | Meaning                                              |
|--------|------------------------------------------------------|
| CN1001 | Generic form: named static method not found on `T`.  |
| CN1002 | Generic form: method has no `IComposer` parameter.   |
| CN1003 | Either form: attribute arguments couldn't be read.   |

Tests live in `GeneratorTests.cs` — synthetic compilations, no Android refs.
**Add a test for any new generator behaviour.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonathanpeppers/Microsoft.AndroidX.Compose](https://github.com/jonathanpeppers/Microsoft.AndroidX.Compose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
