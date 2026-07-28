---
trigger: always_on
description: **OSlash** is an educational Python library that brings functional programming abstractions from Haskell to Python. With 740+ stars, it's used by developers learning functional programming concepts through practical Python implementations.
---

# CLAUDE.md - OSlash Development Guide

## 📚 Project Overview

**OSlash** is an educational Python library that brings functional programming abstractions from Haskell to Python. With 740+ stars, it's used by developers learning functional programming concepts through practical Python implementations.

This project demonstrates:

- **Monadic composition** - Chain computations with context
- **Functors and Applicatives** - Map and apply operations
- **Protocol-based design** - Structural subtyping for flexibility
- **Modern Python 3.12+** - PEP 695 type parameters and match statements
- **Strict type safety** - Zero tolerance for type errors in CI

## 🎯 Philosophy

### Educational First

OSlash prioritizes learning and clarity over production concerns:

- **Explicit implementations** - Show how abstractions work, not just that they work
- **No backward compatibility constraints** - Always use latest Python features
- **Type safety as documentation** - Types teach correct usage patterns
- **Examples in code** - Tests and examples show idiomatic usage

### Functional Purity

Where practical, we maintain functional programming principles:

- **Immutability** - Data structures don't mutate
- **Referential transparency** - Functions return same output for same input
- **Composition over inheritance** - Build complex behavior from simple parts
- **Explicit effects** - IO monad makes side effects visible in types

### Modern Python

We embrace Python 3.12+ features aggressively:

- **PEP 695 type parameters** - `class Maybe[T]:` instead of `Generic[TypeVar]`
- **Pattern matching** - `match` statements for type discrimination
- **Type aliases** - `type Continuation[T, R] = Callable[[T], R]`
- **Protocol-based design** - Structural subtyping with `@runtime_checkable`

## 🏗️ Architecture

### Core Abstractions (Protocols)

Located in [oslash/typing/](oslash/typing/), these define the "laws" of functional programming:

#### 1. Functor - Mappable Containers

```python
@runtime_checkable
class Functor[T](Protocol):
    def map[U](self, fn: Callable[[T], U]) -> Functor[U]:
        """Transform values inside the container."""
        ...
```

**Law**: `functor.map(f).map(g) == functor.map(lambda x: g(f(x)))`

**Examples**: All monads are functors. Use `map` to transform wrapped values.

#### 2. Applicative - Apply Wrapped Functions

```python
@runtime_checkable
class Applicative[T](Protocol):
    def apply[U](self, something: Applicative[Callable[[T], U]]) -> Applicative[U]:
        """Apply a wrapped function to wrapped values."""
        ...

    @classmethod
    def pure(cls, value: T) -> Self:
        """Wrap a pure value."""
        ...
```

**Use case**: Lift functions of multiple arguments into the applicative context.

#### 3. Monad - Chainable Computations

```python
@runtime_checkable
class Monad[T](Protocol):
    def bind[U](self, fn: Callable[[T], Monad[U]]) -> Monad[U]:
        """Chain computations that produce wrapped values."""
        ...

    @classmethod
    def unit(cls, value: T) -> Self:
        """Wrap a pure value."""
        ...
```

**Law**: Left identity, right identity, associativity (see Haskell monad laws)

**Operators**:

- `|` for bind: `monad | func`
- `>>` for then: `monad1 >> monad2`

#### 4. Monoid - Combinable Values

```python
@runtime_checkable
class Monoid[T](Protocol):
    @classmethod
    def empty(cls) -> Self:
        """Identity element."""
        ...

    def __add__(self, other: Monoid[T]) -> Self:
        """Associative binary operation."""
        ...
```

**Laws**:

- Identity: `x + empty() == x` and `empty() + x == x`
- Associativity: `(x + y) + z == x + (y + z)`

### Monad Implementations

All located in [oslash/](oslash/):

#### 1. [Identity](oslash/identity.py) - Simplest Monad

```python
class Identity[T]:
    """Wraps a value with no additional context."""
```

**Use case**: Teaching monad laws, building monad transformers.

#### 2. [Maybe](oslash/maybe.py) - Optional Values

```python
class Maybe[T]:
    """Represents a computation that might fail."""

class Just[T](Maybe[T]):
    """Contains a value."""

class Nothing[T](Maybe[T]):
    """Represents absence of value."""
```

**Use case**: Handle null/None without explicit checks.

**Example**:

```python
def safe_div(x: float, y: float) -> Maybe[float]:
    if y == 0:
        return Nothing()
    return Just(x / y)

result = Just(10) | (lambda x: safe_div(x, 2)) | (lambda x: Just(x + 1))
# Result: Just(6.0)
```

#### 3. [Either](oslash/either.py) - Success or Failure

```python
class Either[T, E]:
    """Represents a computation that can fail with an error."""

class Right[T, E](Either[T, E]):
    """Success case."""

class Left[T, E](Either[T, E]):
    """Failure case with error information."""
```

**Use case**: Error handling with context about what went wrong.

**Example**:

```python
def parse_int(s: str) -> Either[int, str]:
    try:
        return Right(int(s))
    except ValueError:
        return Left(f"Cannot parse '{s}' as int")
```

#### 4. [List](oslash/list.py) - Immutable Linked List

```python
class List[T]:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbrattli/OSlash](https://github.com/dbrattli/OSlash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
