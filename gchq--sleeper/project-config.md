---
trigger: always_on
description: There are a number of ways we tend to write and structure our code, where if we keep them consistent it might improve
---

Coding Conventions
==================

There are a number of ways we tend to write and structure our code, where if we keep them consistent it might improve
the maintainability of the codebase as a whole. We try to keep to these conventions unless there's a good reason within
a specific part of the code. We try to remain open to alternative approaches if they improve maintainability.

## Tests

Please see our [test strategy](test-strategy.md), which also includes conventions when writing tests. Many of our
conventions for this are covered in a linked [test design article](test-design.md).

## Java keyword usage

### Access level modifiers

Within a class we try to declare the minimum access level for all elements. Instance fields are almost always private,
with getters added only when necessary. Methods should only be public when they are intended to be called from outside
the class. Usually if a method has no usages from outside a class it can either be private or be deleted. Nested classes
can be private, but builders are usually public.

Classes are usually public, but may be at a lower access level for implementation details.

In a class defining JUnit tests, access level modifiers may be omitted based on preference. Other test helper or test
base classes should follow the same style as non-test code.

### Use of final

We usually only use the final keyword in field declarations. Fields should be declared final in all cases unless they
need to be reassigned. In practice, most fields should be final, as we usually create a new object if a field would
otherwise need to be reassigned.

We don't use final for parameters or local variables. We don't usually declare final classes.

In a class defining JUnit tests, the final keyword may be omitted based on preference. Other test helper or test base
classes should follow the same style as non-test code.

### Use of this

We usually only use the this keyword in a constructor or a setter. If the this keyword is needed to assign any of the
fields in a constructor, we use this in all the field assignments.

For example:

```java
public Partition(String id, Range range) {
    this.id = id;
    this.region = new Region(List.of(range));
}
```

With a builder, the this keyword can be omitted:

```java
public Partition(Builder builder) {
    id = builder.id;
    region = builder.region;
}
```

## Null handling

When we need an empty value, we prefer a non-null value, e.g. a no-op instead of a null for a Supplier object. Many
types have a more natural empty value than a null. Usually it is not necessary to wrap a value in an Optional to achieve
this, but it can be useful to clarify the optionality in a limited context.

We avoid using an Optional as a function parameter or a field. A null can be used for these purposes, but we use
various patterns to avoid dealing with a null explicitly.

When a class does contain a nullable field or an optional parameter, we expect it to bear the responsibility of managing
null values within a limited context, internal to the class. It should present an API that avoids the need for null
checks, or passing a null explicitly. For example, we avoid passing a null to test helper methods.

We avoid using null as an uninitialised value for later initialisation. We avoid leaving fields uninitialised, except in
a builder, where fields will be set before the object is built. Wherever possible we ensure an object is fully
initialised after it's constructed.

### Patterns for optional values

We consider using multiple function signatures, where one includes the optional parameter as non-optional, and the other
doesn't include it at all.

We consider a [parameter object](https://refactoring.guru/introduce-parameter-object), where setting the null can be the
responsibility of a builder or a static factory method.

A builder pattern is often helpful for managing optional fields, where we can document explicitly which are optional,
and we don't need to call the setter methods when not initialising those fields.

In a data class, a field can be null if it's optional by necessity. An Optional can be used to expose the value outside
the class. Wherever possible, we avoid using null values in a class responsible for logic, wiring or high level policy.

## Ordering within a Java class

We try to keep to this ordering of elements in a class declaration:

1. Static fields
2. Instance fields
3. Constructors
4. Static methods that return an instance of the class (static constructors)
5. Static methods that return a builder of the class
6. Other public static methods
7. Public instance methods other than those mentioned below
8. Private methods
9. Public getter instance methods that return the value of a field with no other code
10. In a builder, a build method that creates an instance of the class being built
11. Implementations of equals, hashCode, toString when needed (on data classes, not on builders)
12. Nested classes/interfaces

Within this ordering, methods should be in the order that they are expected to be used.

Note that many classes will not contain many of these elements.

## Javadoc


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gchq/sleeper](https://github.com/gchq/sleeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
