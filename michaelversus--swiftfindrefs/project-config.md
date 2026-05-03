---
trigger: always_on
description: - Use uppercase for types (and protocols), lowercase for everything else
---

## Naming
- Use camel case
- Use uppercase for types (and protocols), lowercase for everything else
- Prefer clarity over brevity
- Avoid abbreviations
- Include all the words needed to avoid ambiguity
- Use names based on roles, not types. In case of similar properties with different type (e.g inside a function), we may use greetingStr or greetingAttrStr.
- Prefer to name the first parameter instead of including its name in the method name (exception: delegate parameter as in the following rule)
- Compensate for weak type information to clarify a parameter’s role
- Begin factory methods with make (or create, or newElement). If in the class itself, make() suffices. If in a factory, e.g BikePartsFactory, we can add makeFrontWheel(), makeRearWheel() etc, where FrontWheel and RearWheel are classes
- Name functions and methods according to their side-effects
    - those without side-effects should read as noun phrases, e.g. x.distance(to: y), i.successor()
    - those with side-effects should read as imperative verb phrases, e.g., print(x), x.sort(), x.append(y)
    - name Mutating/nonmutating method pairs consistently. A mutating method will often have a nonmutating variant with similar semantics, but that returns a new value rather than updating an instance in-place
    - when the operation is naturally described by a verb, use the verb’s imperative for the mutating method and apply the “ed” or “ing” suffix to name its nonmutating counterpart
    - when the operation is naturally described by a noun, use the noun for the nonmutating method and apply the “form” prefix to name its mutating counterpart
- Uses of Boolean methods and properties should read as assertions about the receiver when the use is nonmutating, e.g. x.isEmpty, line1.intersects(line2)
- Protocols that describe what something is should read as nouns (e.g. Collection, CollectionProtocol)
- Protocols that describe a capability should be named using the suffixes able, ible, or ing (e.g. Equatable, ProgressReporting)
- The names of other types, variables, and constants should read as nouns
- Label tuple members and name closure parameters

## Documentation
- Document public APIs with `///` doc comments.
- Annotate deprecated APIs with `@available`.

# Error handling
- Use `throws` and `do-try-catch` for error propagation.
- Define custom `Error` types with descriptive cases.

## Type inference and Type declaration
- Take advantage of type inference when declaring new variables or properties. Type inference is generally fast enough for simple expressions.
- You may use explicit types for complex expressions, though, if you feel it improves readability or helps Xcode figure out the types.
- Prefer the shortcut versions of type declarations over the full generics syntax.

## Code organization
- Use extensions to organize your code into logical blocks of functionality. Each extension should be set off with a `// MARK: - comment` to keep things well-organized.
- Prefer adding a separate extension per protocol conformance
- Add lifecycle methods with the order they are naturally called
- Prefer to add the rest methods with the order they are called within a classes
- Import only the modules a source file requires. For example, don't import UIKit when importing Foundation will suffice. Likewise, don't import Foundation if you must import UIKit
- sort imports alphabetically.

## Interface Segregation Principle
- Apply Interface Segregation Principle when creating protocols

## Spacing
- Indent using 4 spaces
- There should be one blank line between methods and up to one blank line between type declarations
- There should be no blank lines before a closing brace
- Colons always have no space on the left and one space on the right. Exceptions are the ternary operator ? : , empty dictionary [:] and #selector syntax addTarget(_:action:)
- For closures with multiple parameters, there should be a single space after every comma. (example: `{ _, value, otherValue in ... }`)

## Comments
- When they are needed, use comments to explain why a particular piece of code does something. Comments must be kept up-to-date or deleted.
- Avoid block comments inline with code, as the code should be as self-documenting as possible. Exception: This does not apply to those comments used to generate documentation.
- Avoid the use of C-style comments (/* ... */). Prefer the use of double- or triple-slash.

## Classes and Structs
- Structs have value semantics. Use structs for things that do not have an identity. An array that contains [a, b, c] is really the same as another array that contains [a, b, c] and they are completely interchangeable. It doesn't matter whether you use the first array or the second, because they represent the exact same thing. That's why arrays are structs.
- Classes have reference semantics. Use classes for things that do have an identity or a specific life cycle. You would model a person as a class because two person objects are two different things. Just because two people have the same name and birthdate, doesn't mean they are the same person. But the person's birthdate would be a struct because a date of 3 March 1950 is the same as any other date object for 3 March 1950. The date itself doesn't have an identity.

## Redundants

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michaelversus/SwiftFindRefs](https://github.com/michaelversus/SwiftFindRefs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
