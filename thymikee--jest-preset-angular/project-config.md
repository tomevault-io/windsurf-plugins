---
trigger: always_on
description: This comprehensive guide contains all 50 TypeScript best practices that apply specifically to jest-preset-angular development. These practices are based on TypeScript documentation, books, articles, and professional experience.
---

# TypeScript Best Practices for Jest Preset Angular

This comprehensive guide contains all 50 TypeScript best practices that apply specifically to jest-preset-angular development. These practices are based on TypeScript documentation, books, articles, and professional experience.

## Core TypeScript Practices

### 1. Follow conventions

Code conventions are base rules that allow the creation of a uniform code base across an organization. Following them does not only increase the uniformity and therefore the quality of the code. Use [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript) as a foundation and complete with TypeScript-specific rules. Make conventions mandatory through linters, formatters, and strong code review. The code conventions must be dynamic and adaptable for each team and project.

### 2. Testing

Testing is more important than shipping. If we have no tests or an inadequate amount, then every time we ship code, we won't be sure that we didn't break anything. There's no excuse to not write tests. There are plenty of good JavaScript test frameworks with typing support for TypeScript. Always write tests for every new feature/module we introduce.

### 3. Strict configuration

The stricter configuration should be mandatory and should be enabled by default because there is not much value in using TypeScript without these few flags. Otherwise, our types will be too permissive, and it is what we are trying to avoid as much as possible with TypeScript.

```json
{
  "forceConsistentCasingInFileNames": true,
  "noImplicitReturns": true,
  "strict": true,
  "noUnusedLocals": true
}
```

The most important one here is the `strict` flag which actually covers four other flags. We could add independently to progressively introduce TypeScript in an existing codebase and slowly get stricter over time: `noImplicitAny`, `noImplicitThis`, `alwaysStrict` and `strictNullChecks`.

### 4. Avoid any. Type everything

Always declare variables or constants with a type other than any. When declaring variables or constants in TypeScript without a typing, the typing of the variable/constant will be deduced by the value that gets assigned to it. This will cause unintended problems. Another advantage of having good typings in our application is that it makes refactoring easier and safer. The any type isn't necessarily a bad thing and, in fact, does still come in useful sometimes. However, in most cases, there is a better alternative that leads to having better defined types overall. In new projects, it is worth setting `strict:true` in the `tsconfig.json` file to enable all strict type checking options.

### 5. Strings should be safe

If we have a variable of type string that can have only a set of values, instead of declaring it as a string type, we can declare the list of possible values as the type.

```typescript
type MyStringType = 'First' | 'Second';
```

By declaring the type of the variable appropriately, we can avoid bugs while writing the code during compile time rather than during runtime.

### 6. Call things by their name

This is a no-brainer, but it is scary how often you will come across variables like `x1`, `fe2` or `xbqne` in JavaScript, or, on the other end of the spectrum, long variable names like `incrementorForMainLoopFromTenToTwenty` or `createNewMemberIfAgeOverTwentyOneAndMoonIsFull`. None of these make much sense. Good variable and function names should be easy to understand and tell us what is going on. Not more and not less. One trap to avoid is marrying values and functionality in names. A function called `isLegalDrinkingAge()` makes more sense than `isOverEighteen()` as the legal drinking age varies from country to country, and there are other things than drinking to consider that are limited by age. Keeping to English is a good idea, too, because programming languages are in English.

### 7. Use utility types

TypeScript already has a few utility types built-in, such as `Partial<T>`, which makes all properties of `T` optional, or `Readonly<T>`, which makes `T` read-only. They will help make our code much easier to understand. As a side note, only try to break interfaces or types into smaller nested interfaces/types if it makes sense from our code's domain point-of-view. Once they are aggressively split up, it's hard to see the structure, especially when using code completion.

### 8. Use const and let

JavaScript first searches to see if a variable exists locally, then searches progressively in higher levels of scope until global variables. `var` is function scope, but `let` and `const` are block scope. Using `let` and `const` where appropriate makes the intention of the declarations clearer. It will also help in identifying issues when a value is reassigned to a constant accidentally by throwing a compile time error. Do use a linter that automates checking and fixing this so that changing let to const doesn't become a delay in code review.

### 9. Use === instead of ==


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thymikee/jest-preset-angular](https://github.com/thymikee/jest-preset-angular) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
