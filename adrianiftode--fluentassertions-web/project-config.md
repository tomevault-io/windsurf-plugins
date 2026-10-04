---
trigger: always_on
description: Conventions for modifying **FluentAssertions.Web** — an assertion extension library over
---

# CONVENTIONS.md

Conventions for modifying **FluentAssertions.Web** — an assertion extension library over
`System.Net.Http.HttpResponseMessage`. Derived from the `master` branch.

Findings are tagged **[Established]** (repeated throughout / explicitly configured), **[Likely]**
(several examples, not universal) or **[Unclear]** (limited or contradictory evidence).

---

## 1. What this library is

It adds HTTP-specific assertions (`response.Should().Be200Ok()`, `.BeAs<T>()`, `.HaveHeader()`,
`.HaveError()`, `.Satisfy()`) and — more importantly — makes every failure message print the **whole HTTP
request and response** so a failing integration test can be diagnosed without a debugger.

It asserts on `HttpResponseMessage` only. There is no `HttpRequestMessage` assertion surface (the request is
read from `response.RequestMessage` for *reporting*, never asserted directly). **[Established]**

---

## 2. Repository layout

```
src/
 FluentAssertions.Web/ ← THE single source of truth for all assertion code
 FluentAssertions.Web.v8/ ← no .cs files; link-compiles the above with FAV8
 AwesomeAssertions.Web/ ← no .cs files; link-compiles the above with FAV8;AAV
 FluentAssertions.Web.Types/ ← ISerializer, FluentAssertionsWebConfig, DeserializationException
 AwesomeAssertions.Web.Types/ ← link-compiles the above with AAV
 HttpMessageFormatter/ ← assertion-framework-agnostic HTTP formatter (own NuGet package)
 FluentAssertions.HttpMessageFormatter/ ← 20-line IValueFormatter adapter onto the above
 AwesomeAssertions.HttpMessageFormatter/ ← link-compiles the above with AAV
 *.Web.Serializers.NewtonsoftJson/ ← optional Newtonsoft serializer, one package per flavour
test/
 FluentAssertions.Web.Tests/ ← THE single source of truth for all specs
 FluentAssertions.Web.v8.Tests/ ← link-compiles the above with FAV8
 AwesomeAssertions.Web.Tests/ ← link-compiles the above with FAV8;AAV
 HttpMessageFormatter.Tests/ ← formatter-only, framework-agnostic
 Sample.Api.Tests/ (+ .v8, .AwesomeAssertions) ← end-to-end tests against samples/
samples/ ← ASP.NET Core sample APIs used by the e2e tests
```

### 2.1 The multi-flavour compilation model — read this before editing anything **[Established]**

One source tree ships as **three** packages. The sibling projects contain *no* `.cs` files of their own; they
link the canonical files:

```xml
<!-- src/AwesomeAssertions.Web/AwesomeAssertions.Web.csproj -->
<PropertyGroup><DefineConstants>$(DefineConstants);FAV8;AAV</DefineConstants></PropertyGroup>
<Compile Include="..\FluentAssertions.Web\**\*.*"
 Exclude="**\bin\**;**\obj\**;**\Properties\**;**\*.csproj;**\GlobalUsings.cs">
 <Link>%(RecursiveDir)%(Filename)%(Extension)</Link>
</Compile>
```

Consequences you must respect:

| Rule | Why |
|---|---|
| Edit only `src/FluentAssertions.Web/**` and `test/FluentAssertions.Web.Tests/**`. Never add a `.cs` file to `*.v8`, `*.AwesomeAssertions*` projects. | They are generated views; a new file is picked up automatically by the glob. |
| A new file needs **no csproj change** anywhere. | The glob is recursive. |
| Every file that declares a namespace opens with the `#if AAV` namespace switch. | `AwesomeAssertions.Web` must not leak a `FluentAssertions` namespace. |
| Three symbols exist: nothing (FA 6/7), `FAV8` (FA ≥ 8), `FAV8;AAV` (AwesomeAssertions). There is **no** `AAV`-without-`FAV8` combination. | AwesomeAssertions 9 is an FA-8-shaped API. |
| `GlobalUsings.cs` is excluded from the *AwesomeAssertions* glob (it has its own) but **not** from the *v8* glob (v8 reuses the `FluentAssertions.*` usings). | Namespace differences. |

The canonical namespace header, verbatim:

```csharp
#if AAV
namespace AwesomeAssertions.Web;
#else
namespace FluentAssertions.Web;
#endif
```

---

## 3. Where a new assertion belongs

| The assertion is about… | File in `src/FluentAssertions.Web/` | Declared on |
|---|---|---|
| a status code | `HttpStatusCodeAssertions.cs` | `partial class HttpResponseMessageAssertions` |
| the response body | `HttpResponseContentAssertions.cs` | `partial class HttpResponseMessageAssertions` |
| an arbitrary header, or header presence | `HeadersAssertions.cs` | presence → `partial class HttpResponseMessageAssertions`; value-level → `class HeadersAssertions` |
| the `Location` header | `LocationAssertions.cs` | `class LocationAssertions : HeadersAssertions` |
| an ASP.NET Core `ValidationProblemDetails` body | `BadRequestAssertions.cs` | `class BadRequestAssertions : HttpResponseMessageAssertions` |
| running a user lambda against the response / a deserialized model | `SatisfyHttpResponseMessageAssertions.cs` / `SatisfyModelAssertions.cs` | `partial class HttpResponseMessageAssertions` |

**Every one of these files has a sibling `…Extensions.cs`** (`HttpStatusCodeAssertionsExtensions.cs`,
`HeadersAssertionsExtensions.cs`, …). Adding a public assertion means editing **two** files. **[Established]**

**Grouping inside a file:** `#region <MethodName>` … `#endregion` around each assertion (or each logical pair).
Used exhaustively in `HttpStatusCodeAssertions.cs` and the spec files. **[Established]**

---

## 4. How to implement an assertion

### 4.1 The instance method (in `*.Web` namespace)

```csharp
/// <summary>
/// Asserts that an HTTP response ....

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adrianiftode/FluentAssertions.Web](https://github.com/adrianiftode/FluentAssertions.Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
