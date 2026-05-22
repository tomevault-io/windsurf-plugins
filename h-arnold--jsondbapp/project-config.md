---
trigger: always_on
description: - Synchronous document DB for Google Apps Script (GAS), MongoDB-like syntax.
---

# JsonDbApp Code Generation Guidelines

## Overview

- Synchronous document DB for Google Apps Script (GAS), MongoDB-like syntax.
- CRUD on named collections (JSON files in Google Drive).
- Access via authenticated Apps Script libraries.
- Consistency via ScriptProperties-based master index.

## Core Principles

- **TDD**: Red-Green-Refactor. Write failing tests first, minimal passing code, then refactor.
- **Component Separation**: Single responsibility, dependency injection via constructor.
- **SOLID**: Follow SOLID principles.
- **Reuse**: Check for existing functionality before new code.
- **GAS Limitations**: V8 engine, not full JS support.
- **Style**: Concise, analytical, British English (except for American APIs). Challenge incorrect assumptions.

## File Structure

- `docs/`: General and planning docs
- `docs/developers/`: Feature and class docs
- `src/01_utils/`: ComparisonUtils.js, ErrorHandler.js, JDbLogger.js, IdGenerator.js, ObjectUtils.js, Validation.js
- `src/02_components/`: CollectionCoordinator.js, CollectionMetadata.js, DocumentOperations.js, FileOperations.js
  - `src/02_components/QueryEngine/`: 01_QueryEngineValidation.js, 02_QueryEngineMatcher.js, 99_QueryEngine.js (multi-file structure)
  - `src/02_components/UpdateEngine/`: 01_UpdateEngineFieldOperators.js, 02_UpdateEngineArrayOperators.js, 03_UpdateEngineFieldPathAccess.js, 04_UpdateEngineValidation.js, 99_UpdateEngine.js (multi-file structure)
- `src/03_services/`: DbLockService.js, FileService.js
- `src/04_core/`: Database.js, DatabaseConfig.js, MasterIndex.js
  - `src/04_core/Collection/`: 01_CollectionReadOperations.js, 02_CollectionWriteOperations.js, 99_Collection.js (multi-file structure)
- `tests/data/`: MockQueryData.js (and other mock data)
- `tests/framework/`: 01_AssertionUtilities.js, 02_TestResult.js, 03_TestRunner.js, 04_TestSuite.js, 05_TestFramework.js
- `tests/unit/`: Unit test suites by class/component:
  - Collection/ (multiple test suites)
  - CollectionCoordinator/ (multiple test suites)
  - DbLockService/
  - DocumentOperations/
  - UtilityTests/
  - ...
- `tests/validation/`: Operator validation suites and orchestrator
- `README.md`, `LICENSE`, `package.json`, `appsscript.json`: Project config and metadata

## Naming Conventions

- **Classes**: PascalCase (e.g. `DocumentOperations`)
- **Methods**: camelCase (`insertDocument`)
- **Private methods**: `_underscore` prefix
- **Variables**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Private properties**: `this._underscore`
- **Files**: Match class name
- **Multi-file classes**: For large classes (e.g. Collection), use numbered file prefixes (01*, 02*, 99*) to control load order; `99*\*.js` composes/exports the class
- **Tests**: `ClassNameTest.js`
- **Test functions**: `testClassNameScenario`
- **Errors**: End with `Error`
- **Config**: `config` or `componentConfig`

## Method Template

```javascript
/**
 * Description
 * @param {Type} param - Description
 * @returns {Type} Description
 * @throws {ErrorType} When thrown
 * @remarks *optional*: Additional notes explaining nuances, reasoning behind design choices or explaining the logic flow of complex methods.
 */
methodName(param) {
  if (!param) throw new ErrorHandler.ErrorTypes.INVALID_ARGUMENT('param', param, 'param is required');
  const result = this._performOperation(param);
  return result;
}
```

## Error Standards

- **Base**: `GASDBError`
- **Common**: `DocumentNotFoundError`, `DuplicateKeyError`, `InvalidQueryError`, `LockTimeoutError`, `FileIOError`, `ConflictError`, `InvalidArgumentError`
- **Additional in project**: `MasterIndexError`, `CollectionNotFoundError`, `ConfigurationError`, `FileNotFoundError`, `PermissionDeniedError`, `QuotaExceededError`, `InvalidFileFormatError`, `OperationError`, `LockAcquisitionFailureError`, `ModificationConflictError`, `CoordinationTimeoutError`
- **Codes**: `'DOCUMENT_NOT_FOUND'`, `'DUPLICATE_KEY'`, `'INVALID_QUERY'`, `'LOCK_TIMEOUT'`, `'FILE_IO_ERROR'`, `'CONFLICT_ERROR'`, `'INVALID_ARGUMENT'`, `'MASTER_INDEX_ERROR'`, `'COLLECTION_NOT_FOUND'`, `'CONFIGURATION_ERROR'`, `'FILE_NOT_FOUND'`, `'PERMISSION_DENIED'`, `'QUOTA_EXCEEDED'`, `'INVALID_FILE_FORMAT'`, `'OPERATION_ERROR'`, `'LOCK_ACQUISITION_FAILURE'`, `'MODIFICATION_CONFLICT'`, `'COORDINATION_TIMEOUT'`
- **Message**: `"Operation failed: specific reason"`

## Implementation Requirements

- **Classes**: Constructor validates inputs, JSDoc on all methods, naming/error patterns.
- **Tests**: Descriptive, Arrange-Act-Assert, independent, per-class folder, per-suite file, orchestrator for all tests.
- **Serialisation**: Use `ObjectUtils.serialise()`/`deserialise()`. Classes needing serialisation: implement `toJSON()`, static `fromJSON()`, register in `ObjectUtils._classRegistry`.
- **Validation**: Use `Validate` class; class-specific validation as private method.
- **TDD**: Always follow Red-Green-Refactor.
- **Linting**: `no-magic-numbers` is an error for source code. Tests may use numeric literals for clarity because the rule is disabled for `tests/**/*.js`.

## Calling Sub-Agents

MANDATORY: Every #runSubagent call must include the agent name. Calls that omit this parameter violate the workflow contract and should be rejected/retried.

### Available Sub-Agents


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [h-arnold/JsonDbApp](https://github.com/h-arnold/JsonDbApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
