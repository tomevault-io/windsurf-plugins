---
trigger: always_on
description: Development standards — architecture patterns, extensions, platform standards, code smells
---


# Development Standards — Architecture & Platform

## 1. Architecture Patterns (extends project_rules.mdc)

### Code Placement
- Business logic — **in common modules**, not in form modules
- Server common modules — suffixes: `*ServerCall`, `*ObjectModule`, `*ManagerModule`
- Client common modules — suffix: `*Client`
- Form-related modules — suffix: `*Forms`
- Server object modules — mandatory preprocessor: `#If Server Or ThickClientOrdinaryApplication Or ExternalConnection Then`

### "Result-Structure" Pattern
Return compound results via Structure:

```bsl
Result = New Structure;
Result.Insert("CheckPassed", CheckResult);
Result.Insert("ErrorText", ErrorText);
Return Result;
```

### "Early Return" Pattern
Reduce nesting by returning early on precondition failures:

```bsl
If Cancel Then
	Return;
EndIf;

If Not ValueIsFilled(ActionDate) Then
	Return DefaultValue;
EndIf;
```

### "Value Table Search" Pattern

```bsl
SearchParameters = New Structure("WorkwearType", CurrentRow.WorkwearType);
FoundRows = DataTable.FindRows(SearchParameters);
If FoundRows.Count() = 0 Then
	Continue;
EndIf;
```

### Event Subscriptions
Preferable over modifying typical modules. All subscription methods — via common module `{PREFIX}EventSubscriptions`.

### New Metadata Objects Placement
Determined by `{NEW_OBJECTS_IN}` parameter from `.dev.env`:

| `{NEW_OBJECTS_IN}` | Behavior |
|---|---|
| `main_configuration` | New objects go into main configuration. Extension — only for event interception |
| `extension` | New objects may be placed in extension. Main configuration not modified without explicit instruction |

Default: `main_configuration`.

### Background Jobs
Operations taking > 10 seconds — move to background jobs with progress indication. Do not block UI.

### Defensive Type Checking
BSL has no strict typing. Check type at function entry when critical:

```bsl
If TypeOf(DocumentsOrRef) <> Type("Array") Then
	DocumentsArray = New Array;
	DocumentsArray.Add(DocumentsOrRef);
Else
	DocumentsArray = DocumentsOrRef;
EndIf;
```

### Safe Structure Property Access
Always check key existence before access:

```bsl
If ReportParameters.Property("StartDate", StartDate) Then
	// use StartDate
EndIf;
```

### Collection Normalization
Normalize input to a single collection type for uniform processing. Use `CommonClientServer.ValueInArray()` for single-to-array conversion.

## 2. Extensions

### Modification Priority
1. **Event subscriptions** (preferred)
2. **Extensions**
3. **Typical code modification** (last resort)

### Extension Directives
- `&Before` / `&After` — preferred
- `&Instead` — only for functions, with mandatory `ContinueCall()`

### Placement Rules (when `{NEW_OBJECTS_IN} = main_configuration`)
- New metadata objects → main configuration
- New attributes of typical objects → main configuration
- Roles → main configuration

Regardless of `{NEW_OBJECTS_IN}`:
- Typical roles → DO NOT modify (create new ones with `{PREFIX}`)

### Forms in Extensions
Visual form editing in extensions — **minimize**. Changes — programmatically through code.

## 3. Platform Standards (extends project_rules.mdc)

### Async and Modality
- Modal calls are **PROHIBITED**: `DoQueryBox()`, `ShowMessageBox()`, `InputNumber()`, etc.
- Approach depends on `{PLATFORM_VERSION}` from `.dev.env`:

| `{PLATFORM_VERSION}` | Approach |
|---|---|
| < 8.3.18 | `NotifyDescription` (callback) |
| ≥ 8.3.18 | `Async` / `Await` (preferred) |

- Inside `Async` procedures use ONLY async analogs. Mixing `Async/Await` with non-async methods is **PROHIBITED**
- Any dialog calls on the server are **PROHIBITED**

### Client-Server Interaction
- **`&AtServerNoContext` is MANDATORY** for all server methods that do not access form data. `&AtServer` only when method directly reads/writes form attributes or elements
- If a method only needs a form attribute value — pass it as parameter and use `&AtServerNoContext`

### Security
- `Execute()` and `Eval()` — **PROHIBITED** without extreme necessity
- **Hardcoded credentials are PROHIBITED** — passwords, tokens, API keys in code are FORBIDDEN
- **RLS** — design with access restriction requirements in mind

### Error Handling (extends project_rules.mdc)
- String localization — `NStr("en = '...'")` with `StringFunctionsClientServer.SubstituteParametersToString()`
- Error collection — into a single variable via `Chars.LF`
- Logging — `DetailErrorDescription(ErrorInfo())`, NOT `BriefErrorDescription()`
- Empty exception handlers are **PROHIBITED** — always log or re-raise

### Dates
- On server — `CurrentSessionDate()` instead of `CurrentDate()`

### Queries (extends project_rules.mdc)
- Temporary tables — prefixed with `TT_`
- `INNER JOIN` preferred over `LEFT JOIN` when possible
- Filter by dimensions first when accessing registers
- Do not modify register movements directly — only through posting mechanism

### Cross-Platform Compatibility
- **COM objects** (`New COMObject(...)`) are **PROHIBITED** unless explicitly specified in task
- For Excel — use spreadsheet document or SSL, not `Excel.Application`
- File paths — use `/` or system functions, do not hardcode `\`

### Platform Version Compatibility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/comol) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
