---
trigger: always_on
description: Skyline is a 17-year-old mature C# codebase for proteomics mass spectrometry data analysis. The project maintains strict conventions for maintainability, localization (English UI with Chinese/Japanese translations), and zero-warning builds. Tests run in multiple locales (English, French, Turkish) to verify number formatting and culture-invariant string operations.
---

# GitHub Copilot Code Review Instructions - Skyline Project

## Project Context

Skyline is a 17-year-old mature C# codebase for proteomics mass spectrometry data analysis. The project maintains strict conventions for maintainability, localization (English UI with Chinese/Japanese translations), and zero-warning builds. Tests run in multiple locales (English, French, Turkish) to verify number formatting and culture-invariant string operations.

## Critical Review Focus Areas

### 1. Asynchronous Programming (CRITICAL)

**NEVER accept `async`/`await` keywords in C# code.**

- Skyline uses `ActionUtil.RunAsync()` for background operations in `pwiz_tools/Skyline/`
- Common libraries (`pwiz_tools/Shared/`) use `CommonActionUtil.RunAsync()`
- Background threads communicate with UI thread via `Control.Invoke()` or `Control.BeginInvoke()`
- Flag any PR introducing `async`, `await`, or `Task<T>` return types
- Exception: External library integrations where async is unavoidable must be wrapped

**Why**: Historical STA threading model and UI synchronization requirements.

**Benefits of `RunAsync()` pattern**:
- Automatic `CurrentCulture`/`CurrentUICulture` propagation for localization testing
- Exception handling wrapper prevents unhandled exceptions from crashing the application
- Consistent threading model across the codebase

**Pattern**:
```csharp
// ✅ ACCEPT - RunAsync with UI updates via Invoke
ActionUtil.RunAsync(() =>
{
    var result = PerformBackgroundWork();
    
    // Update UI on UI thread
    control.BeginInvoke((Action)(() =>
    {
        textBox.Text = result;
    }));
});

// ❌ REJECT - async/await
public async Task DoWorkAsync()
{
    await Task.Run(() => PerformBackgroundWork());
}
```

### 2. Localization and Resource Strings (CRITICAL)

**ALL user-facing text must be in .resx files, NEVER string literals.**

```csharp
// ❌ REJECT - String literal for UI
MessageBox.Show("File not found");
throw new IOException("Cannot open file");

// ✅ ACCEPT - Resource string
MessageBox.Show(Resources.ErrorMessage_FileNotFound);
throw new IOException(Resources.ErrorMessage_CannotOpenFile);
```

**Flag**:
- String literals in MessageBox, exception messages shown to users, dialog text, menu items
- Missing `.Designer.cs` updates when `.resx` is modified
- Resource properties not in alphabetical order

**Exception**: Debug-only strings using `$@""` format are acceptable (e.g., `Debug.WriteLine`, `ToString()`, internal exception messages never shown to users).

**Do NOT flag**: Missing entries in localized resx siblings (`*.ja.resx`, `*.zh-CHS.resx`) when a new entry is added to the master `*.resx`. Translations are handled by a separate translator workflow; the .NET resource fallback (localized missing -> master) means JP/ZH users see English until translators sync, which is the intended steady state.

### 3. Translation-Proof Testing (CRITICAL)

**NEVER accept English text literals in test assertion comparisons.**

```csharp
// ❌ REJECT - Hardcoded English in comparison
Assert.IsTrue(errorMessage.Contains("File not found"));
Assert.AreEqual("Error", dialogTitle);

// ✅ ACCEPT - Resource-based comparison
AssertEx.Contains(errorMessage, Resources.ErrorMessage_FileNotFound);
AssertEx.AreEqual(Resources.DialogTitle_Error, dialogTitle);

// ✅ ACCEPT - English failure message (acceptable and desirable)
Assert.AreEqual(expected, actual, "Skyline version should match configuration");
AssertEx.Contains(errorMessage, expectedSubstring, "Error message should contain expected fragment");
```

**Flag**:
- English string literals used in **comparisons** (the `expected` or `substring` arguments)
- Direct `.Contains()` instead of `AssertEx.Contains()`
- Network error assertions without `HttpClientTestHelper.GetExpectedMessage()`

**Do NOT flag**:
- English text in assertion failure messages (the optional message parameter)
- English comments explaining test logic

**Why**: Tests must pass in multiple UI languages (English, Chinese, Japanese) and locales (English, French, Turkish). The *comparison values* come from localized resources, but *failure messages* remain in English for developer clarity.

### 4. Test Structure and Performance

**REJECT multiple `[TestMethod]` for related validations.**

```csharp
// ❌ REJECT - Separate test methods (causes 15+ sec overhead each)
[TestMethod] public void TestImportStep1() { }
[TestMethod] public void TestImportStep2() { }
[TestMethod] public void TestImportStep3() { }

// ✅ ACCEPT - Consolidated test with private helpers
[TestMethod]
public void TestImportWorkflow()
{
    RunFunctionalTest();
}

protected override void DoTest()
{
    TestImportStep1();
    TestImportStep2();
    TestImportStep3();
}

private void TestImportStep1() { /* validation */ }
private void TestImportStep2() { /* validation */ }
private void TestImportStep3() { /* validation */ }
```

**Why**: Functional test overhead (SkylineWindow creation) is 5-10 seconds per test method.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ProteoWizard/pwiz](https://github.com/ProteoWizard/pwiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
