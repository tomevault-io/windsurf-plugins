---
trigger: always_on
description: >
---


# Swift Testing (`import Testing`)

Xcode 16+ | Swift 6.0+ (coverage through 6.3) | iOS 16+ (runtime backdeployed) | `import Testing`

## Repo-Policy Override

**Use the framework already established in the target.** If the test target uses XCTest (`XCTestCase`, `XCTAssert*`), write new tests in XCTest unless the user explicitly requests Swift Testing or migration. This skill activates only when Swift Testing syntax is already present or explicitly requested.

## High-Risk Mistakes

### 1. Generates XCTest boilerplate instead of Swift Testing

```swift
// WRONG — XCTest pattern
import XCTest
class FooTests: XCTestCase {
    func testBar() {
        XCTAssertEqual(foo(), 42)
    }
}

// CORRECT — Swift Testing
import Testing
@Suite struct FooTests {
    @Test func bar() {
        #expect(foo() == 42)
    }
}
```

### 2. Wraps async code in `Task { }` — silent pass

`Task { }` fires and forgets. The test function returns before the Task body executes. `#expect` failures inside are never reported. **The test PASSES even when the expectation fails.**

```swift
// WRONG — test always passes regardless of result
@Test func fetchData() {
    Task {
        let result = await api.fetch()
        #expect(result == expected)  // never reported
    }
}

// CORRECT — mark function async
@Test func fetchData() async {
    let result = await api.fetch()
    #expect(result == expected)
}
```

### 3. Uses `XCTAssertThrowsError` instead of `#expect(throws:)`

```swift
// WRONG
XCTAssertThrowsError(try foo()) { error in XCTAssertEqual(error as? MyError, .bar) }

// CORRECT — specific error value
#expect(throws: MyError.bar) { try foo() }

// CORRECT — error type only
#expect(throws: MyError.self) { try foo() }

// BEST (Swift 6.1+) — capture returned error for inspection
let error = try #require(throws: MyError.self) { try foo() }
#expect(error == .bar)
#expect(error.details.contains("expected"))
```

The error matcher closure pattern is **deprecated** since ST-0006. Prefer capturing the returned error.

### 4. Uses `XCTestExpectation` instead of `confirmation()`

```swift
// WRONG — XCTest callback pattern
let exp = expectation(description: "callback")
sut.onComplete { exp.fulfill() }
waitForExpectations(timeout: 5)

// CORRECT — Swift Testing block-scoped pattern
await confirmation { confirm in
    sut.onComplete { confirm() }
}

// Use `try await` only when the closure body throws
try await confirmation { confirm in
    try sut.riskyStart { confirm() }
}
```

`confirmation()` is block-scoped — NOT create-then-fulfill. See [references/async-and-concurrency.md](references/async-and-concurrency.md).

### 5. Forgets `try` with `#require` — it throws

```swift
// WRONG — compile error
let value = #require(optionalValue)

// CORRECT — #require throws, must use try
let value = try #require(optionalValue)
```

### 6. Writes order-dependent tests without `.serialized`

Tests run in **parallel by default, in random order**. Tests that share mutable state and depend on execution order will fail intermittently.

```swift
// WRONG — will fail intermittently
@Suite struct NumberEntry {
    let shared = SharedState.instance
    @Test func recordTwo() { shared.record("2") }        // may not run first
    @Test func recordThree() { shared.record("3") }      // may not run second
}

// CORRECT — use .serialized when order matters
@Suite(.serialized) struct NumberEntry {
    let shared = SharedState.instance
    @Test func recordTwo() { shared.record("2") }
    @Test func recordThree() { shared.record("3") }
}
```

### 7. Thinks `.serialized` prevents ALL parallelism

`.serialized` only guarantees order **within that suite**. Sibling serialized suites still run in parallel with each other.

```swift
// These two suites run IN PARALLEL with each other (both share CurrentValue.shared)
@Suite(.serialized) struct NumberEntry { ... }   // runs its tests in order
@Suite(.serialized) struct DecimalPoint { ... }  // runs its tests in order, BUT parallel with NumberEntry

// CORRECT — nest under parent serialized suite
@Suite(.serialized) struct DisplayEntry {
    @Suite struct NumberEntry { ... }
    @Suite struct DecimalPoint { ... }
}
```

### 8. Uses setUp/tearDown instead of init/deinit

```swift
// WRONG — XCTest lifecycle
class Tests: XCTestCase {
    override func setUp() { ... }
    override func tearDown() { ... }
}

// CORRECT — Swift Testing uses init/deinit
@Suite struct Tests {
    let sut: MyType
    init() { sut = MyType() }       // runs before each test
    // deinit runs after each test (only for classes)
}
```

Each `@Test` function gets a **fresh struct instance**. Properties set in one test don't carry over to another.

### 9. Uses instance members in `@Test(arguments:)` — no instance exists yet

The `arguments:` expression is evaluated before any suite instance is created, so instance members are unavailable. Use `static` properties, literals, or `try`/`await` expressions.

```swift
// WRONG — instance member not available
let samples = [1, 2, 3]
@Test(arguments: samples) func test(x: Int) { ... }
// ERROR: Instance member 'samples' cannot be used on type 'MyTests'

// CORRECT — static property
static let samples = [1, 2, 3]
@Test(arguments: samples) func test(x: Int) { ... }

// ALSO CORRECT — literal
@Test(arguments: [1, 2, 3]) func test(x: Int) { ... }
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [farkasseb/swift-testing-skill](https://github.com/farkasseb/swift-testing-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
