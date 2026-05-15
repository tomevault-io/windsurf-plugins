---
trigger: always_on
description: **Package:** `tsentials` (npm)
---

# tsentials — Agent Guide

## Quick Reference

**Package:** `tsentials` (npm)  
**Purpose:** Railway-oriented programming — error-as-value, no exceptions  
**Node:** ≥18, TypeScript ≥5.0, ESM only

## Import Paths

```typescript
import { Result, ResultChain, ResultAsync, fromAsync, chain, maybeToResult, resultToMaybe } from 'tsentials/result';
import { Err, AppError, ErrorType, ErrorMetadata } from 'tsentials/errors';
import { Maybe, tryFirst, tryLast, tryFind, choose, asMaybe } from 'tsentials/maybe';
import { RuleEngine } from 'tsentials/rules';
import type { Rule } from 'tsentials/rules';
import { createEntityBase, createSoftDeletable } from 'tsentials/entity';
import type { DomainEvent } from 'tsentials/entity';
import { fetchResult, RequestBuilder } from 'tsentials/http';
import { SystemDateTimeProvider, createFakeDateTimeProvider } from 'tsentials/time';
import { deepClone, cloneArray } from 'tsentials/clone';
import { Union } from 'tsentials/union';
import { safeJsonParse, safeJsonStringify, parseAndValidate, isJsonObject, isJson } from 'tsentials/json';
import type { Json, JsonObject } from 'tsentials/json';
import { pipe, flow } from 'tsentials/function';
import { Eq } from 'tsentials/eq';
import { Ord, sortBy } from 'tsentials/ord';
import { Predicate } from 'tsentials/predicate';
import { NonEmptyArray, head, asNonEmptyArray } from 'tsentials/array';
import { These } from 'tsentials/these';
import { Tree } from 'tsentials/tree';
import { Record } from 'tsentials/record';
```

---

## Core Patterns

### Result\<T\> — error handling without exceptions

```typescript
import { Result } from 'tsentials/result';
import { Err } from 'tsentials/errors';

// Factory
Result.success(value)
Result.failure(Err.validation('Code', 'message'))
Result.ok()                                          // void success
Result.successIf(cond, value, err)                   // conditional success
Result.failIf(cond, value, err)                      // conditional failure
Result.try(() => JSON.parse(raw), e => Err.validation('JSON.Invalid', 'Bad JSON'))

// Type guards
Result.isSuccess(r)   // r is { ok: true; value: T }
Result.isFailure(r)   // r is { ok: false; errors: AppError[] }
Result.firstError(r)  // AppError
Result.lastError(r)   // AppError

// Pipeline (sync)
Result.then(r, fn)                // monadic bind → Result<U>  ← NOT .bind()
Result.map(r, fn)                 // transform value
Result.ensure(r, pred, err)       // guard — err can be a factory fn
Result.tap(r, fn)                 // side effect on success
Result.tapError(r, fn)            // side effect on failure
Result.match(r, onOk, onErr)      // exhaustive exit

// Conditional pipeline
Result.bindIf(r, cond, fn)        // bind only if condition/predicate true
Result.tapIf(r, cond, fn)         // tap only if condition/predicate true
Result.tapErrorIf(r, cond, fn)    // tapError only if condition/predicate true

// Error recovery
Result.compensate(r, fn)          // recover from any failure
Result.compensateFirst(r, fn)     // recover using first error
Result.recover(r, pred, fn)       // recover only if first error matches predicate
Result.mapError(r, fn)            // transform error array
Result.else(r, fallback)          // fallback value on failure

// Extraction
Result.unwrap(r)                  // throws if failure
Result.unwrapOr(r, default)       // default value on failure
Result.unwrapOrElse(r, fn)        // computed default on failure
Result.deconstruct(r)             // [ok, value, errors] tuple

// Combination
Result.and([r1, r2])              // all must succeed — collects ALL errors
Result.or([r1, r2])               // first success wins — collects all errors if all fail
Result.combine(r1, r2, r3)        // heterogeneous tuple Result<[T1, T2, T3]>
Result.flatten(r)                 // Result<Result<T>> → Result<T>
Result.always(r, fn)              // unconditional cleanup — returns fn result
Result.ap(fab, fa)                // applicative apply
Result.partition(results)         // split into { ok: T[], err: AppError[] }
Result.sequence(promises)         // await Promise<Result<T>>[] → Result<T[]>
```

### Async Pipeline — fromAsync / ResultAsync\<T\>

`ResultAsync<T>` builds the chain synchronously, resolves in one `await` at the end.

```typescript
import { fromAsync } from 'tsentials/result';

const profile = await fromAsync(fetchUser(userId))
  .andThen(user => validateUser(user))     // NOTE: andThen, NOT then
  .ensure(user => user.isActive, Err.validation('User.Inactive', 'Not active'))
  .map(user => user.profile)
  .tap(p => console.log('fetched', p.name))
  .match(profile => profile, () => null);

// Async variants on Result namespace
await Result.thenAsync(r, async fn)
await Result.mapAsync(r, async fn)
await Result.ensureAsync(r, async pred, err)
await Result.tapAsync(r, async fn)
await Result.tapErrorAsync(r, async fn)
await Result.compensateAsync(r, async fn)
await Result.compensateFirstAsync(r, async fn)
await Result.recoverAsync(r, pred, async fn)
await Result.mapErrorAsync(r, async fn)
await Result.bindIfAsync(r, cond, async fn)
await Result.alwaysAsync(r, async fn)
```

### ResultChain\<T\> — fluent sync wrapper

```typescript
import { chain } from 'tsentials/result';

const r = chain(Result.success(5))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [senrecep/tsentials](https://github.com/senrecep/tsentials) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
