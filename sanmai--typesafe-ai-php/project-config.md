---
trigger: always_on
description: This is a community-maintained PHP client for the TypeSafe AI evaluation API.
---

# AI Agent Guidelines

This is a community-maintained PHP client for the TypeSafe AI evaluation API.

It is designed to be type-safe and easy to use: requests are built with a fluent interface, and each answer type has a typed accessor.

- **PHP Version:** 8.2 or newer.
- **API reference:** https://docs.typesafe.ai (`POST /v1/systemone`, and `GET /v1/models`, which the published reference does not document).
- **Core Architecture:**
    - **Client:** `TypeSafeClient` sends requests and deserializes responses.
    - **Request:** `SystemOneRequest` retains the state and a map of `Question` DTOs (`Noul`, `Choice`, `Score`). The client serializes it with JMS.
    - **Response/DTOs:** `SystemOneResult` retains a map of `Answer` DTOs. JMS selects the subclass using the `type` field.
    - **Models:** `TypeSafeClient::models()` returns a `ModelsResponse` of `DTO\ModelCard`.
    - **Attributes:** `Noul`, `Choice`, and `Score` are PHP attributes too, so a result class can declare its own questions. `TypeSafeClient::evaluate()` is the only entry point. It builds a single `AttributeReader` for the class, sends the questions that the reader parses from the constructor, and then hydrates the class with `hydrate()`. The request and the result know nothing about attributes: keep the attribute logic in the reader and its one caller.

End-user documentation:

@README.md

## Project Navigation

**Key locations:**
- **Core Logic:** @src/TypeSafeClient.php (the main entry point for all API calls).
- **Request:** `src/SystemOneRequest.php` and `src/Question/`.
- **Data Models:** `src/SystemOneResult.php`, `src/ModelsResponse.php`, and `src/DTO/` (all response objects).
- **Tests:** `tests/`, with response fixtures in `tests/data/`.

## Coding Standards

- **Type Hinting:** Use precise type hints for parameters and return types. Use generics (`@template`) where appropriate.
- **DTOs:** Data Transfer Objects are simple classes with public properties.
- **Little logic in DTOs:** Requests and responses are mostly pure data: public properties. Put serialization rules in JMS attributes where possible. When a rule applies to one DTO only, a small serialization hook on that DTO (see `NoulCriteria::descriptions()`) is better than a global strategy in the serialization context of the client. The request builder methods forward their arguments unchanged.
- **No validation:** Question types do not validate their contents; the API does, so the SDK continues to operate when the API relaxes a rule.
- **Naming:** Follow PER-CS coding standards (extended PSR-12). Run `make cs` to validate.

## Implementation Details

- **JSON maps**: The API uses maps keyed by ids and options that you choose. Declare them with a key type, such as `#[Type('array<string, string>')]`: JMS then writes a JSON object, also when the map is empty or has keys such as `"0"`. Declare lists as `array<string>`; JMS re-indexes them.
- **Free-form JSON values**: Instructions and criteria can be text, a JSON object, an array, or null. Declare such a value as `union`, as in `#[Type('array<string, union>')]` for a map and `#[Type('array<union>')]` for a list: JMS dispatches on the runtime type and does not modify the value. A value type of `string` stringifies nested data, `mixed` is not a JMS type and throws, and omitting the type re-indexes numeric keys. `union` works only when serializing; use a bare `#[Type('array')]` for a free-form value that is also deserialized, as `ScoreAnswer::$legend` does.
    - `union` is the name JMS gives a PHP union property itself (`TypedPropertiesDriver`), and `UnionHandler` is registered for that name. Writing the name by hand is not a documented feature, so treat it as load-bearing on JMS internals: `serializeUnion()` ignores the type parameters and dispatches on the runtime type, which is what makes it a pass-through, while `deserializeUnion()` reads them and throws when they are absent.
    - The tripwire is `SystemOneRequestTest::provideRequests()`. Its structured cases assert the exact serialized JSON, so a JMS upgrade that changes this fails the build rather than quietly sending `"Array"` in place of a nested description. Keep those cases when you modify the provider.
- **Nulls**: The client serializes with `serializeNull` enabled, so all null values are sent: in arrays (a choice option without a description) and in the user state. If a DTO has optional fields that the API must not receive as null, the DTO omits them itself: exclude the properties and add an inline virtual property that returns only the values that are set (see `NoulCriteria::descriptions()`). To omit a nested object that yields no values, add `#[SkipWhenEmpty]` (see `Noul::$criteria`).
- **Question type field**: Each question class has a `public string $type` property with a default value. There is no discriminator on requests, so a custom question type does not need a change in the SDK.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanmai/typesafe-ai-php](https://github.com/sanmai/typesafe-ai-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
