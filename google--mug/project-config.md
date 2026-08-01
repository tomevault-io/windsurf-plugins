---
trigger: always_on
description: When generating or refactoring code using the `dot-parse` library, you MUST
---

# Rules for AI Agents Generating dot-parse Code

When generating or refactoring code using the `dot-parse` library, you MUST
follow these rules to ensure safety, performance, and idiomatic style.

## 1. Data Model Design

- **Always** make the domain data model immutable (e.g., using Java `record`s).
- **Use "wither" methods** (methods that return a new instance with the updated
  property) for optional properties or modifiers.
- **Rule of thumb for withers**:

  - The language to be parsed represents the user's mental model.
  - If in the mental model, the data can be expressed with or without a
    property (which usually has a default value), the data model should
    reflect that.
  - The convenience factory method should assume that default value, and an
    incremental "wither" should be used to attach it optionally.
- This enables clean functional chaining using **method references** (e.g.,
  `AbcNote::withDuration`) rather than lambdas.
- **Prefer** method references over lambdas when using withers in combinators
  like `optionallyFollowedBy()` and `sequence()`.

  ```java
  public record AbcNote(
      Accidental accidental, char pitch, int octave, NoteDuration duration) {
    // Factory assumes default duration and octave = 0
    public static AbcNote middle(char pitch) {
      return new AbcNote(null, pitch, 0, NoteDuration.of(1));
    }
  
    // Factory assumes default duration and octave = 1
    public static AbcNote high(char pitch) {
      return new AbcNote(null, pitch, 1, NoteDuration.of(1));
    }

    // Wither attaches it optionally
    public AbcNote withDuration(NoteDuration duration) {
      return new AbcNote(accidental, pitch, octave, duration);
    }
  }
  ```

- **Prefer** creating enums in the data model for reserved words, operators,
  etc., with their `toString()` returning the canonical form (e.g.,
  `DOUBLE_SHARP("^^")`). This enables building parsers cleanly using the
  `anyOf(Enum[])` overload (e.g.,
  `anyOf(MyEnum.values())`).

## 2. Static Import

- **Always** static import `dot-parse` factory methods from the `Parser` class
  and `CharPredicate` classes.
- Do NOT use `Parser.consecutive()`, `Parser.anyOf()`, etc. Use
  `consecutive()`, `anyOf()` directly.

  ```java
  import static com.google.common.labs.parse.Parser.anyOf;
  import static com.google.common.labs.parse.Parser.consecutive;
  import static com.google.common.labs.parse.Parser.string;
  ```

## 3. Safety against Infinite Loops (Zero-Width Parsers)

Unlike most parser combinator libraries, Dot-Parse deliberately outlaws
zero-width parsers because it's too easy to shoot yourself in the foot,
running into an infinite loop by sneakily nesting it within a repetition
parser.

You don't see parser people talk about it often but when it happens, the
program hangs. And even if you manually kill the VM and take a thread dump,
the dump would only show that it failed in a loop deeply inside the `many()`
library method body, but giving you no idea which of *your* grammar had
incorrectly used a zero-width parser, because that parser object construction
code is somewhere in the wild, just not in the stack trace of the `parse()`
call! It's like saying: "Yeah man, you were dead. It was bad, death was
bad.", but just won't tell you what killed you.

The `optional()`, `orElse()` and `zeroOrMore()` are only to be used in safe
places like `followedBy()`, `then()`, `between()`, `immediatelyBetween()` etc.
where the composite parser is guaranteed to consume input.

**Always** attach optional Parser rules to a Parser that consumes input.

While it may feel tempting to want to do something like this:

```java {.bad}
// Won't compile!
Parser<String> parent = word().followedBy(".").orElse("this");
Parser<List<String>> ancestors = parent.atLeastOnce();
```

It would have opened a can of worms named infinite loops, if the API had
allowed it. That's why in the `Parser` API, the code above would not compile
(because `orElse()` returns a special `OrEmpty` type, not a `Parser`). So
don't try it! You are forced to complete the fluent chain using methods on
`OrEmpty` that ensure safety.

-   **Chaining Optional Parsers**: Optional parsers (e.g., from `.optional()`,
    `.orElse()`, `.zeroOrMore()`) return a `Parser.OrEmpty` instance. You can
    chain them using `OrEmpty` methods like `.then()`, `.followedBy()`, and
    `.delimitedBy()`, which continue to return `OrEmpty`.
-   **Exiting the Unsafe Zone**: To convert an `OrEmpty` chain back into a
    standard `Parser`, you must eventually attach it to a non-empty `Parser`
    using methods like `Parser.then()`, `Parser.followedBy()`,
    `Parser.sequence(Parser, Production...)` or `OrEmpty.between()` /
    `OrEmpty.immediatelyBetween()`. This ensures the composite parser is
    guaranteed to consume input.
-   **The `Production` Interface**: Represents either a `Parser` or a
    `Parser.OrEmpty`. Overloads like `sequence(Parser, Production...)` can take
    either type for the 2nd production rule and the remaining.
-   **Common Combinators**: The `Production` interface defines common methods
    shared by both `Parser` and `OrEmpty`, including `between()`,
    `immediatelyBetween()`, `then()`, `followedBy()`, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [google/mug](https://github.com/google/mug) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
