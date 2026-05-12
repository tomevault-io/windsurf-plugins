---
trigger: always_on
description: You are currently working with Swift code within a Swift package. Below are common anti-patterns that agents tend to follow, and what you should do instead. Make sure to generate code in correspondence with examples listed as GOOD or OK, and avoid generating code listed as a BAD example.
---

# Swift Cactus
You are currently working with Swift code within a Swift package. Below are common anti-patterns that agents tend to follow, and what you should do instead. Make sure to generate code in correspondence with examples listed as GOOD or OK, and avoid generating code listed as a BAD example.

## Basics
### Always Use Type Names For Calling Initializers
```swift
struct Foo {}

// GOOD
let foo = Foo()

// BAD
let foo: Foo = .init()
```

### Avoid Explicit Collection Types For Empty Collections
``` swift
// GOOD
let arr = [UInt8]()
let dict = [String: Int]()

// BAD
let arr: [UInt8] = []
let dict: [String: Int] = [:]
```

### Prefer Using `self` When Accessing Member Variables And Methods
```swift
struct Item {
  let property: String
}

// GOOD
extension Item {
  var propertyCount: Int {
    self.property.count
  }
}

// BAD
extension Item {
  var propertyCount: Int {
    property.count
  }
}
```

### Prefer Using `Self` When Accessing static Variables And Methods
```swift
struct Item {
  static let property = "..."
}

// GOOD
extension Item {
  static var propertyCount: Int {
    Self.property.count
  }
}

// BAD
extension Item {
  static var propertyCount: Int {
    property.count
  }
}

extension Item {
  static var propertyCount: Int {
    self.property.count
  }
}
```

### Avoid Delayed Nested Local Property Initializations
Avoid nesting initial assignments inside nested blocks such as in conditional and switch statements.
```swift
// GOOD
func foo() {
  let property = bar()
  // ...
}

func bar() -> Value {
  if someCondition {
    currentValue
  } else {
    previousValue
  }
}

// BAD
func foo() {
  let property: Value
  if someCondition {
    property = currentValue
  } else {
    property = previousValue
  }

  // ...
}
```

### Prefer No Return Keyword For One Line Conditional Expressions
```swift
// GOOD
func value() -> Value {
  switch statement {
  case 1: 1
  case 4: 16
  default: 64
  }
}

func value() -> Value {
  if statement == 1 {
    1
  } else if statement == 4 {
    16
  } else {
    64
  }
}

// BAD
func value() -> Value {
  switch statement {
  case 1: return 1
  case 4: return 16
  default: return 64
  }
}

func value() -> Value {
  if statement == 1 {
    return 1
  } else if statement == 4 {
    return 16
  } else {
    return 64
  }
}
```

### Avoid `private static` Helper Methods For Instance Methods
```swift
struct Value {}

// GOOD
extension Value {
  func foo() {
    // ...
    self.bar()
    // ...
  }

  private func bar() {
    // ...
  }
}

// BAD
extension Value {
  func foo() {
    // ...
    Self.bar()
    // ...
  }

  private static func bar() {
    // ...
  }
}
```

### Prefer Sequence Algorithms Over Raw Loops
```swift
// GOOD
let transformedItems = items.compactMap {
  guard $0.count > 0 else { return nil }
  return TransformedItem($0) 
}

let filteredItems = items.filter { $0.count > 0 }

// BAD
var transformedItems = [TransformedItem]()
for item in items {
  guard item.count > 0 else { continue }
  transformedItems.append(TransformedItem(item))
}

var filteredItems = [Item]()
for item in items {
  guard item.count > 0 else { continue }
  filteredItems.append(item)
}
```

### Prefer Optional Algorithms Over Long Drawn Out Unwrap Checks
```swift
struct Item {
  let name: String?
}

// GOOD
func updatedName(item: Item) -> String? {
  item.name.map { $0 + " Updated" }
}

// BAD
func updatedName(item: Item) -> String? {
  guard let name = item.name else { return nil }
  return name + " Updated" 
}
```

### Always Make Trivial Structs Hashable and Sendable
```swift
// GOOD
struct Item: Hashable, Sendable {
  let name: String
  let quantity: Int
}

// BAD
struct Item {
  let name: String
  let quantity: Int
}
```

### Prefer `self.init` Instead Of Property Assignments In Initializers
```swift
struct Item: Hashable, Sendable {
  let name: String
  let quantity: Int
}

// GOOD
extension Item {
  init(name: String) {
    self.init(name: name, quantity: 0)
  }
}

// BAD
extension Item {
  init(name: String) {
    self.name = name
    self.quantity = 0
  }
}
```

## Concurrency
### Avoid `@unchecked Sendable` Without Justification
Avoid conforming to `@unchecked Sendable`, and try to always make a type conform to Sendable proper instead. If you must conform to `@unchecked Sendable`, then make sure to leave a comment explaining why it is safe to do so.
```swift
// GOOD
final class Item: Sendable {
  let count = Mutex(0)
  // ...
}

// OK (but not always ideal, avoid if possible)
final class Item: @unchecked Sendable {
  // NB: @unchecked Sendable is safe because we use a lock.
  private let lock = NSLock()
  var count = 0

  // ...
}

// BAD
final class Item: @unchecked Sendable {
  var count = 0

  // ...
}
```

### Avoid `Task.detached` At All Costs
`Task.detached` is rarely useful, and can almost always be expressed in simpler ways. Prefer traditional `Task` initializations instead.
```swift
// GOOD
Task { await work() }

// BAD
Task.detached { await work() }
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mhayes853/swift-cactus](https://github.com/mhayes853/swift-cactus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
