---
trigger: always_on
description: This document defines the hard rules that must be followed when defining correct `testcase` specs. These rules are derived from the `testcase` framework's philosophy and best practices.
---

# testcase Spec user guide

## Spec Definition Hard Rules

This document defines the hard rules that must be followed when defining correct `testcase` specs. These rules are derived from the `testcase` framework's philosophy and best practices.

### Test Function Naming

#### Rule 1: The test function name MUST be the testing subject's name

```go
// ✅ Correct
func TestMyType(t *testing.T) { ... }
func TestParseInput(t *testing.T) { ... }
func TestRepository(t *testing.T) { ... }

// ❌ Incorrect
func TestSomething(t *testing.T) { ... }
func TestMainLogic(t *testing.T) { ... }
```

#### Rule 2: Test functions MUST follow the `Test<Subject>` convention

- Use PascalCase for the subject name
- Do not prefix with "Test" more than once
- Keep it concise and descriptive

---

### Spec Declaration

#### Rule 3: Create spec immediately in test function body

```go
func TestMyType(t *testing.T) {
    s := testcase.NewSpec(t)
    
    // ... spec definition
}
```

#### Rule 4: Do not defer spec creation

```go
// ❌ Incorrect
func TestMyType(t *testing.T) {
    var s *testcase.Spec
    t.Run("setup", func(t *testing.T) {
        s = testcase.NewSpec(t)
    })
}

// ✅ Correct
func TestMyType(t *testing.T) {
    s := testcase.NewSpec(t)
}
```

---

### Subject Definition

#### Rule 5: Define the System Under Test (SUT) at spec root level

```go
func TestMyType(t *testing.T) {
    s := testcase.NewSpec(t)
    
    subject := let.Var(s, func(t *testcase.T) MyType {
        return MyType{}
    })
}
```

#### Rule 6: Subject MUST be accessible via getter function or `.Get(t)`

```go
// Using Var
subject := let.Var(s, func(t *testcase.T) MyType { ... })
// Access: subject.Get(t)

// Or helper function
func subject(t *testcase.T) MyType { ... }
```

---

### Describe Blocks

#### Rule 7: A `Spec#Describe` MUST have a dedicated `act` defined as a function

The `act` function takes `*testcase.T` only as its input and executes the operation that the current `Spec#Describe` block is about.

```go
s.Describe("#MyFunc", func(s *testcase.Spec) {
    var (
        input = let.Var(s, func(t *testcase.T) string {
            return t.Random.String()
        })
    )
    
    // ✅ Correct: act defined at top of Describe block
    act := func(t *testcase.T) string {
        return subject.Get(t).MyFunc(input.Get(t))
    }
    
    s.Then("returns input unchanged", func(t *testcase.T) {
        assert.Equal(t, act(t), input.Get(t))
    })
})
```

#### Rule 8: Each `Describe` block MUST have exactly one ACT

```go
// ❌ Incorrect - multiple different acts in same Describe
s.Describe("#MyFunc", func(s *testcase.Spec) {
    s.Then("test 1", func(t *testcase.T) {
        result := subject.Get(t).MyFunc(input.Get(t))
    })
    
    s.Then("test 2", func(t *testcase.T) {
        err := subject.Get(t).Save(ctx.Get(t), entity.Get(t)) // Different method!
    })
})

// ✅ Correct - one ACT per Describe block
s.Describe("#MyFunc", func(s *testcase.Spec) {
    act := func(t *testcase.T) string {
        return subject.Get(t).MyFunc(input.Get(t))
    }
    
    s.Then("test 1", func(t *testcase.T) {
        _ = act(t)
    })
})

s.Describe("#Save", func(s *testcase.Spec) {
    act := func(t *testcase.T) error {
        return subject.Get(t).Save(ctx.Get(t), entity.Get(t))
    }
    
    s.Then("test 2", func(t *testcase.T) {
        _ = act(t)
    })
})
```

#### Rule 9: Describe blocks MUST group tests by method or feature

```go
s.Describe("#MyFunc", func(s *testcase.Spec) { ... })
s.Describe("#Save", func(s *testcase.Spec) { ... })
s.Describe("validation", func(s *testcase.Spec) { ... })
```

---

### Variables and Inputs

#### Rule 10: Test variables MUST be defined using `let.Var` or `let.Let`

```go
var (
    input = let.Var(s, func(t *testcase.T) string {
        return t.Random.String()
    })
    
    expected = let.Var(s, func(t *testcase.T) string {
        return input.Get(t)
    })
)
```

#### Rule 11: Variables MUST be accessed via `.Get(t)` in test functions

```go
// ✅ Correct
s.Then("returns expected value", func(t *testcase.T) {
    assert.Equal(t, act(t), input.Get(t))
})

// ❌ Incorrect - capturing variable outside test
var capturedInput string
s.Before(func(t *testcase.T) {
    capturedInput = input.Get(t) // Wrong!
})
```

#### Rule 12: Use `t.Random` for property testing to enable reproducible failures

```go
input := t.Random.String()           // Random string
id := t.Random.UUID()                // Random UUID  
num := t.Random.IntBetween(1, 100)   // Random int in range
data := t.Random.Bytes(32)           // Random bytes
```

---

### The ACT Pattern

#### Rule 13: `act` MUST be defined at the top of each `Describe` block

```go
s.Describe("#MyFunc", func(s *testcase.Spec) {
    // ✅ Correct - act at top
    act := func(t *testcase.T) string {
        return subject.Get(t).MyFunc(input.Get(t))
    }
    
    var input = let.Var(s, ...)
    
    s.Then("...", func(t *testcase.T) { ... })
})

// ❌ Incorrect - act defined after tests
s.Describe("#MyFunc", func(s *testcase.Spec) {
    s.Then("...", func(t *testcase.T) { ... }) // Can't use act yet!
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamluzsi/testcase](https://github.com/adamluzsi/testcase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
