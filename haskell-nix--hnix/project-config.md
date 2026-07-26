---
trigger: always_on
description: This guide provides essential context for working with HNix - a Haskell implementation of the Nix expression language using advanced functional programming techniques including recursion schemes and abstract definitional interpreters.
---

# HNix Codebase Guide for Claude Code

This guide provides essential context for working with HNix - a Haskell implementation of the Nix expression language using advanced functional programming techniques including recursion schemes and abstract definitional interpreters.

## Quick Start Commands

### Essential Development Workflow
```bash
# Enter development environment
nix-shell

# Build the project
cabal v2-configure
cabal v2-build

# Run a single test
cabal v2-test --test-options="--pattern '/Parser/basic literals/'"

# Interactive REPL for exploration
cabal v2-repl
> :load Nix.Eval
> :type evalExprLoc

# Quick evaluation test
cabal v2-run hnix -- --eval --expr '1 + 1'
```

### Testing Commands
```bash
# Standard test suite
cabal v2-test

# All tests including Nixpkgs parsing (slow)
env ALL_TESTS=yes cabal v2-test

# Only Nixpkgs compatibility tests
env NIXPKGS_TESTS=yes cabal v2-test

# Pretty-printer round-trip tests
env PRETTY_TESTS=yes cabal v2-test

# Test with coverage
cabal v2-configure --enable-coverage
cabal v2-test --enable-coverage
```

### Debugging & Profiling
```bash
# Memory profiling (upload .prof to speedscope.app)
cabal v2-run --enable-profiling --flags=profiling \
  hnix -- --eval --expr 'builtins.length [1 2 3]' +RTS -hy -l

# Stack trace on error
cabal v2-run hnix -- --trace --eval --expr 'throw "error"' +RTS -xc

# Heap profiling for thunk leaks
cabal v2-run hnix -- --eval --expr 'import <nixpkgs> {}' \
  +RTS -h -i0.1 -RTS && hp2ps -e8in -c hnix.hp

# Reduce complex expressions for minimal repro
hnix --reduce bug.nix --eval --expr 'import ./bug.nix'
```

## Architecture: Working with Recursion Schemes

### Core Expression Types
```haskell
-- The functor (non-recursive structure)
data NExprF r  -- 18 constructors: NConstant, NStr, NSym, NList, etc.

-- Fixed point gives recursion
type NExpr = Fix NExprF

-- Location annotations via composition
type NExprLoc = Fix (AnnF SrcSpan NExprF)
```

### Using ADI for Custom Behavior

The `adi` function (`src/Nix/Utils.hs:345`) enables behavior injection:

```haskell
-- Example: Add tracing to evaluation
tracingEval :: NExprLoc -> m (NValue t f m)
tracingEval = adi addTrace baseEval
  where
    addTrace :: Transform NExprLocF (m (NValue t f m))
    addTrace f e = do
      traceM $ "Evaluating: " ++ show (void e)
      result <- f e
      traceM $ "Result: " ++ show result
      pure result
```

Common ADI use cases:
- **Error context**: `evalWithMetaInfo = adi addMetaInfo evalContent`
- **Profiling**: Inject timing measurements at each recursion
- **Memoization**: Cache results of sub-expressions
- **Debugging**: Track evaluation path

### Free Monad Value System

```haskell
type NValue t f m = Free (NValue' t f m) t
-- Pure t = thunk (unevaluated)
-- Free v = evaluated value
```

**Memory implications**:
- Thunks accumulate until forced
- Use `force` explicitly to prevent buildup
- Monitor with `+RTS -s` for thunk statistics

## Working with the Effect System

### Core Type Classes

```haskell
class MonadEval v m where
  evalExprLoc :: NExprLoc -> m v  -- Evaluate expression
  evalError :: Doc v -> m a        -- Report error

class MonadThunk t m a | t -> m a where
  thunk :: m a -> m t              -- Create thunk
  force :: t -> m a                -- Force evaluation

class (MonadEval v m, MonadThunk t m v) => MonadNix e t f m
```

### Adding New Effects

```haskell
-- Define capability
class Monad m => MonadMyEffect m where
  myOperation :: String -> m Int

-- Add to evaluation monad
newtype MyNix m a = MyNix (ReaderT MyEnv m a)
  deriving (Functor, Applicative, Monad)

instance MonadMyEffect (MyNix m) where
  myOperation s = MyNix $ asks (lookupThing s . myEnvData)
```

## Extending HNix

### Adding Built-ins

1. Add to `src/Nix/Builtins.hs`:
```haskell
builtinsList :: [(Text, BuiltinType)]
builtinsList =
  [ ("myBuiltin", arity2 myBuiltinImpl)
  -- ...
  ]

myBuiltinImpl :: MonadNix e t f m => NValue t f m -> NValue t f m -> m (NValue t f m)
myBuiltinImpl arg1 arg2 = do
  -- Force evaluation if needed
  str <- fromStringNoContext =<< fromValue arg1
  num <- fromValue arg2
  -- Perform operation
  pure $ nvStr $ makeNixString (str <> show num)
```

2. Test in `tests/EvalTests.hs`
3. Document behavior matching Nix semantics

### Modifying Evaluation

```haskell
-- Hook into evaluation via MonadEval instance
instance MonadEval (NValue t f m) MyCustomNix where
  evalExprLoc expr = do
    -- Pre-evaluation hook
    logExpression expr
    -- Delegate to standard evaluation
    result <- standardEvalExprLoc expr
    -- Post-evaluation hook
    recordMetrics expr result
    pure result
```

## Common Pitfalls & Solutions

### Memory Issues

**Problem**: Thunk accumulation causing memory exhaustion
```haskell
-- BAD: Builds huge thunk chain
foldl' (\acc x -> thunk (acc + x)) 0 [1..1000000]

-- GOOD: Forces evaluation incrementally
foldl' (\acc x -> force acc >>= \a -> pure (a + x)) 0 [1..1000000]
```

**Problem**: Lazy fields in strict data
```haskell
-- BAD: ~ makes field lazy despite ! on data
data MyData = MyData { ~myField :: !Int }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haskell-nix/hnix](https://github.com/haskell-nix/hnix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
