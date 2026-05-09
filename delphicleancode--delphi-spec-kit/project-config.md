---
trigger: always_on
description: ﻿# GitHub Copilot — Instructions for Delphi Projects
---

﻿# GitHub Copilot — Instructions for Delphi Projects

## Contexto

This is a **Delphi (Object Pascal)** project that follows SOLID principles, clean code and the Object Pascal Style Guide. See `AGENTS.md` in the project root for the complete convention reference.

## General Guidelines

1. **Always generate code in Object Pascal** (Delphi) unless explicitly requested in another language.
2. **Use PascalCase** for all identifiers. Lowercase reserved words.
3. **Respect the prefixes** of the Pascal convention: `T` (classes), `I` (interfaces), `E` (exceptions), `F` (private fields), `A` (parameters), `L` (local variables).
4. **Prefer interfaces** over concrete classes for dependencies.
5. **Use constructor injection** for dependency injection.
6. **Never put business logic in form event handlers** (`OnClick`, `OnChange`, etc.). Delegate to services.

## Code Style

### Indentation and Formatting
- Indentation: **2 spaces** (no tabs)
- `begin` on the **same line** of `if`, `for`, `while`, `with` when in a single block
- `begin` on **new line** for method implementations
- Limit of **120 characters** per line

### Unit Sections
Order unit sections according to:
```
unit Nome;

interface

uses
  { RTL units },
  { Units do projeto };

type
  { Enums e Records }
  { Interfaces }
  { Classes }

implementation

uses
  { Units adicionais só necessárias na implementação };

{ Implementações }

end.
```

### Variable Declaration
```pascal
// Preferir inline var quando disponível (Delphi 10.3+)
var LCustomer := TCustomer.Create('João');

// Ou declaraction explícita com prefixo L
var
  LCustomer: TCustomer;
  LCount: Integer;
```

## Error Handling

- Use **specific exceptions** (create exception classes per domain):
  ```pascal
  EBusinessRuleException = class(Exception);
  EEntityNotFoundException = class(Exception);
  EValidationException = class(Exception);
  ```
- **Guard clauses** at the beginning of the method instead of deep nesting
- **Try/finally** for memory management
- **Try/except** only for actual error handling, never for control flow

## Documentation

- Generate **XMLDoc** for public methods and properties
- Comments in **Portuguese** for Brazilian projects
- Do not comment self-explanatory code

## Design Patterns

When creating new features, follow the layered architecture:
- **Domain:** Entities, Value Objects, Interfaces
- **Application:** Services, Use Cases, DTOs
- **Infrastructure:** Repositories (FireDAC), external APIs
- **Presentation:** Forms VCL/FMX

## What NOT to generate

- ❌ Do not use `with` statement
- ❌ Do not create global variables
- ❌ Do not use `AnsiString` when `string` (UnicodeString) is appropriate
- ❌ Don't use magic numbers — declare constants
- ❌ Don't do generic catch (`except on E: Exception do ShowMessage`)
- ❌ Don't mix UI logic with business logic
- ❌ Do not create methods with more than 20 lines
- ❌ Don't ignore `Free` of temporary objects (use try/finally)

## REST Frameworks

### Horse
- Controller: class with `class procedure RegisterRoutes`
- Handler: `class procedure Nome(AReq: THorseRequest; ARes: THorseResponse; ANext: TProc)`
- Middleware: `THorse.Use(Jhonson)`, `THorse.Use(CORS)`, `THorse.Use(HandleException)`
- Routes: kebab-case, plural — `/api/customers`, `/api/order-items`
- Always delegate to Services — never access data in the controller

### DelphiMVCFramework
- Controller: inherits `TMVCController` with `[MVCPath('/api/resource')]`
- Routes: attributes `[MVCPath]`, `[MVCHTTPMethod([httpGET])]`
- Active Record: inherits `TMVCActiveRecord` with `[MVCTable]`, `[MVCTableField]`
- Serialization via `Render()` — do not use `Response.Content` directly
- JWT: `TMVCJWTAuthenticationMiddleware`

### Dext Framework
- Minimal API: `App.Builder.MapGet`, `MapPost` using anonymous functions (handlers)
- Native routing with Auto Model Binding populating DTOs
- Dependency Injection: `App.Services.AddSingleton`, `AddScoped`
- Entity ORM: `DbContext.Where(U.Age > 18)` (Smart Properties expressions instead of SQL strings)
- Async: use `TAsyncTask` for asynchronism and promises

### DevExpress Components
- DevExpress component prefixes: `grd` (TcxGrid), `tvw` (TcxGridDBTableView), `lyt` (TdxLayoutControl), `skn` (TdxSkinController)
- Prefer `TdxLayoutControl` to manual positioning
- Configure grid via code when columns are dynamic
- Export: use `cxGridExportLink` for Excel/PDF

### ACBr Project (Commercial Automation)
- **Golden Rule:** Do not attach components (`TACBrNFe`, `TACBrCTe`, etc.) directly to UI forms.
- Isolate tax logic in Service classes (e.g. `TNFeService`) or Repositories.
- Configure certificates and cryptographic libraries (WinCrypt/OpenSSL) via code, with data dynamically obtained from abstraction classes.
- Always guarantee memory freeing if you build ACBr components dynamically in a Service (`try...finally Free;`).
- Common prefixes in the base UI or DataModules: `acbrNFe`, `acbrECF`, `acbrTef`, `acbrBoleto`.

### Firebird Database
- **Rule of Thumb:** Dialect 3 ALWAYS (`SQLDialect := '3'`), CharacterSet UTF8, PageSize 16384.
- **RETURNING:** `INSERT INTO ... RETURNING id` requires `LQuery.Open`, NEVER `ExecSQL` (which discards the result).
- **Generators:** Use `GEN_ID(generator, 1)` in `BEFORE INSERT` or `IDENTITY` triggers (Firebird 3+).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [delphicleancode/delphi-spec-kit](https://github.com/delphicleancode/delphi-spec-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
