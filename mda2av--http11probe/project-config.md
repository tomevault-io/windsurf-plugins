---
trigger: always_on
description: Use when close is acceptable instead of a status code.
---

# Http11Probe — AI Agent Contribution Guide

This file is designed for LLM/AI agent consumption. It contains precise, unambiguous instructions for adding a new test or a new framework to the Http11Probe platform.

## Project overview

Http11Probe is an HTTP/1.1 compliance and security tester. It sends raw TCP requests to servers and validates responses against RFC 9110/9112. The codebase is C# / .NET 10. Documentation is a Hugo + Hextra static site under `docs/`.

---

## TASK A: Add a new test

Adding a test requires changes to **5 locations** (sometimes 4 if URL mapping is automatic).

### Step 1 — Add the test case to the suite file

Choose the correct suite file based on category:

| Category | File path |
|----------|-----------|
| Compliance | `src/Http11Probe/TestCases/Suites/ComplianceSuite.cs` |
| Smuggling | `src/Http11Probe/TestCases/Suites/SmugglingSuite.cs` |
| Malformed Input | `src/Http11Probe/TestCases/Suites/MalformedInputSuite.cs` |
| Normalization | `src/Http11Probe/TestCases/Suites/NormalizationSuite.cs` |
| Cookies | `src/Http11Probe/TestCases/Suites/CookieSuite.cs` |

Append a `yield return new TestCase { ... };` inside the `GetTestCases()` method. Here is the full schema:

```csharp
yield return new TestCase
{
    // REQUIRED fields
    Id = "COMP-EXAMPLE",                           // Unique ID. Prefix conventions below.
    Description = "What this test checks",         // One-line human description.
    Category = TestCategory.Compliance,            // Compliance | Smuggling | MalformedInput | Normalization
    PayloadFactory = ctx => MakeRequest(           // Builds the raw HTTP bytes to send.
        $"GET / HTTP/1.1\r\nHost: {ctx.HostHeader}\r\n\r\n"
    ),
    Expected = new ExpectedBehavior                // How to validate the response. See below.
    {
        ExpectedStatus = StatusCodeRange.Exact(400),
    },

    // OPTIONAL fields
    RfcLevel = RfcLevel.Must,                      // Must (default) | Should | May | OughtTo | NotApplicable
    RfcReference = "RFC 9112 §5.1",                // Use § not "Section". Omit if no RFC applies.
    Scored = true,                                 // Default true. Set false for MAY/informational tests.
    AllowConnectionClose = false,                  // On Expected. See validation rules below.
    BehavioralAnalyzer = (response) => ...,        // Optional Func<HttpResponse?, string?> for analysis notes.
};
```

**Test ID prefix conventions:**

| Prefix | Suite |
|--------|-------|
| `COMP-` | Compliance |
| `SMUG-` | Smuggling |
| `MAL-` | Malformed Input |
| `NORM-` | Normalization |
| `COOK-` | Cookies |
| `RFC9112-X.X-` or `RFC9110-X.X-` | Compliance (maps directly to an RFC section) |

**Validation patterns — choose ONE:**

Pattern 1 — Exact status, no alternatives:
```csharp
Expected = new ExpectedBehavior
{
    ExpectedStatus = StatusCodeRange.Exact(400),
}
```
Use for strict MUST-400 requirements (e.g. SP-BEFORE-COLON, MISSING-HOST, DUPLICATE-HOST, OBS-FOLD, CR-ONLY).

Pattern 2 — Status with connection close as alternative:
```csharp
Expected = new ExpectedBehavior
{
    ExpectedStatus = StatusCodeRange.Exact(400),
    AllowConnectionClose = true,
}
```
Use when close is acceptable instead of a status code.

Pattern 3 — Custom validator (takes priority over ExpectedStatus):
```csharp
Expected = new ExpectedBehavior
{
    CustomValidator = (response, state) =>
    {
        if (state == ConnectionState.ClosedByServer && response is null) return TestVerdict.Pass;
        if (response is null) return TestVerdict.Fail;
        if (response.StatusCode == 400) return TestVerdict.Pass;
        if (response.StatusCode >= 200 && response.StatusCode < 300) return TestVerdict.Warn;
        return TestVerdict.Fail;
    },
    Description = "400 or close = pass, 2xx = warn",
}
```
Use for pass/warn/fail logic, timeout acceptance, or multi-outcome tests.

**Available StatusCodeRange factories:**
- `StatusCodeRange.Exact(int code)` — single status code
- `StatusCodeRange.Range(int start, int end)` — inclusive range
- `StatusCodeRange.Range2xx` — 200-299
- `StatusCodeRange.Range4xx` — 400-499
- `StatusCodeRange.Range4xxOr5xx` — 400-599

**Available TestVerdict values:** `Pass`, `Fail`, `Warn`, `Skip`, `Error`

**Available ConnectionState values:** `Open`, `ClosedByServer`, `TimedOut`, `Error`

**Helper method available in all suites:**
```csharp
private static byte[] MakeRequest(string request) => Encoding.ASCII.GetBytes(request);
```

**RfcLevel values:**
- `RfcLevel.Must` — (default) RFC says MUST / MUST NOT. Only set explicitly if you want to be clear.
- `RfcLevel.Should` — RFC says SHOULD / SHOULD NOT / RECOMMENDED.
- `RfcLevel.May` — RFC says MAY / OPTIONAL. Both behaviors are compliant.
- `RfcLevel.OughtTo` — RFC uses "ought to" (weaker than SHOULD).
- `RfcLevel.NotApplicable` — No single RFC 2119 keyword applies (best-practice / defensive tests).

Check the [RFC Requirement Dashboard](docs/content/docs/rfc-requirement-dashboard.md) for classification guidance and to verify your assignment matches the existing pattern.

**Critical rules:**
- NEVER set `AllowConnectionClose = true` for MUST-400 requirements where the RFC explicitly says "respond with 400".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MDA2AV/Http11Probe](https://github.com/MDA2AV/Http11Probe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
