---
trigger: always_on
description: `AddBehavior`/`InsertBehavior` (in `src/Stunts/StuntExtensions.cs`) are generic extension
---

# Stunts design notes

## Behavior configuration API

`AddBehavior`/`InsertBehavior` (in `src/Stunts/StuntExtensions.cs`) are generic extension
methods constrained to `IStunt`:

```csharp
public static TStunt AddBehavior<TStunt>(this TStunt stunt, ...) where TStunt : IStunt
```

The constraint makes the API discoverable only on actual stunts (instead of any object), and
the generic type parameter preserves the concrete receiver type so calls can be chained.

## `Stunt.Of` vs `Stunt.For` vs `Stunt.Get`

`src/Stunts.Package/Stunt.cs` ships as content into consuming projects and is the
`[StuntGenerator]`-annotated factory API:

| API | Returns | Use |
|-----|---------|-----|
| `Stunt.Of<T>(...)` | `T` | Just the stunt, no behavior configuration needed |
| `Stunt.For<T>(...)` | `StuntReference<T>` | Create and configure behaviors |
| `Stunt.Get<T>(T stunt)` | `StuntReference<T>` | Configure an already created stunt |
| `Stunt.Builder()` | `StuntBuilder` | Configure behaviors once, build many stunts |

`StuntReference<T>` implements `IStunt`, so the behavior extension methods apply to it
directly, and exposes the stunt via `ToObject()`.

`Stunt.For<T>` passes a `Lazy<T>`. The stunt is constructed on the first `ToObject()` call
(the implicit conversion calls `ToObject()` too). That call installs the behaviors configured
so far with `BehaviorPipelineFactory.UseAmbient`, so virtual members invoked from a base
constructor are intercepted. If none were added, the factory already current at that call is
used. After construction, `Behaviors` forwards to the stunt's own pipeline. `Stunt.Get<T>`
still wraps an already constructed instance and forwards `Behaviors` immediately.

An implicit conversion to `T` is also declared, but **C# does not allow user-defined
conversions to interface types**, so it only kicks in for class and delegate stunts. Since most
stunts are interfaces, `ToObject()` is the usage pattern tests and docs should show. This
limitation is why `Stunt.Of<T>` keeps returning `T` instead of the reference type.

Delegate stunts are unwrapped when the instance is materialized: for a delegate `T`, the
generated stunt is the delegate's `Target`.

The VB content file (`src/Stunts.Package/Stunt.vb`) mirrors the C# one: `StuntReference(Of T)`,
`StuntBuilder`, `Get`, and the `Of`/`For`/`Build` overloads (including the delegate ones), with two
VB-specific notes:

- VB cannot constrain a type parameter to `System.Delegate` (BC32061), so the delegate `Of`/`For`
  overloads are declared as `(Of T)(implementation As T)` without a constraint. Behavior is
  identical (the argument is passed as the single constructor argument), but a call passing a
  lone `Nothing` (e.g. `Stunt.Of(Of IFoo)(Nothing)`) is ambiguous: use
  `Stunt.Of(Of IFoo)(New Object() {Nothing})`. The overload is kept because it's what gives
  lambdas their parameter type inference in `Stunt.Of(Of MyDelegate)(Function(x, y) x + y)`.
- The `Widening` conversion to `T` has the same interface limitation as C# (BC30512 under
  `Option Strict On`), so `ToObject()` is the pattern to show.

There is no VB project in the solution, so changes to this file are validated by compiling it
in a scratch VB project referencing `src/Stunts/bin/Debug/netstandard2.0/Stunts.dll`.

Every factory overload (including the `For` and `Build` ones) must carry `[StuntGenerator]`: the source
generator keys off that attribute on the invoked method (instance or static) and uses the call site's
generic type arguments to decide which stunt types to generate.

### Generic wrappers

Every generated stunt is a closed, non-generic type: there are no open generic templates.
A generic method passing its own type parameters to a generator method (e.g.
`static IDictionary<string, T> Make<T>() => Stunt.Of<IDictionary<string, T>>()`) is a *wrapper*
and must itself carry `[StuntGenerator]` (`ST016`, with a code fix adding the attribute).
`StuntClosure` (shared by analyzers and the generator) computes each wrapper's *stunt definitions*: the
stunt types it creates expressed over its own type parameters, flattened through nested wrappers.
Each concrete call site (`Make<int>()`) substitutes its type arguments into the definitions, so only
the combinations actually used are generated. Wrappers cannot use type parameters of a containing
type (`ST017`), be virtual/abstract/interface members (`ST018`), recurse with different type
arguments or chain more than `StuntClosure.MaxDepth` (8) wrappers (`ST019`). Local functions are
supported; their symbols are normalized to the closure's own semantic model, since local function
symbols bound by different models are not equal.

Cross-assembly: each stunt definition records the assembly that calls the leaf `Stunt.Of<T>` at run time
(the internal `Stunt` content class passes `typeof(Stunt).Assembly`). Libraries emit a public
`Stunts.Generated.StuntDefinitions` class whose `[StuntDefinition(docId, assembly)]` methods carry the
stunt definitions of externally visible wrappers as parameter types (trivial forwarding definitions are omitted).
Consumers read them from metadata, generate the closed stunts, and register them under the leaf
assembly's name.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devlooped/stunts](https://github.com/devlooped/stunts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
