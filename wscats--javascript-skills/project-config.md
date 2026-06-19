---
trigger: always_on
description: >
---


# JavaScript Style Guide Skill

## Activation

This skill activates when the user mentions or implies **JavaScript** in their request. Once activated, it:

- Responds with best-practice guidance
- Generates JavaScript code that strictly conforms to the style guide
- Provides explanations for why each convention is recommended

---

## Complete Style Rules

### 1. References

- Use `const` for all references; avoid `var`.
- If you must reassign references, use `let` instead of `var`.
- Both `const` and `let` are block-scoped, whereas `var` is function-scoped.

```javascript
// bad
var count = 1;

// good
const count = 1;
let mutableValue = 1;
mutableValue += 1;
```

### 2. Objects

- Use literal syntax for object creation.
- Use computed property names when creating objects with dynamic property names.
- Use object method shorthand and property value shorthand.
- Group shorthand properties at the beginning of the object declaration.
- Only quote properties that are invalid identifiers.
- Prefer the object spread syntax (`...`) over `Object.assign` to shallow-copy objects.

```javascript
// bad
const item = new Object();

// good
const item = {};

// computed property names
const key = 'name';
const obj = { [key]: 'value' };

// method & property shorthand
const name = 'Alice';
const atom = {
  name,
  value: 1,
  addValue(val) {
    return atom.value + val;
  },
};

// shallow copy
const original = { a: 1, b: 2 };
const copy = { ...original, c: 3 };
```

### 3. Arrays

- Use literal syntax for array creation.
- Use `Array.from` or the spread operator to convert array-like objects.
- Use `return` statements in array method callbacks.
- Use line breaks after the opening bracket and before the closing bracket if the array has multiple lines.

```javascript
// bad
const items = new Array();

// good
const items = [];

// convert iterable
const nodes = Array.from(document.querySelectorAll('.item'));
const uniqueValues = [...new Set(arr)];

// array methods
[1, 2, 3].map((x) => x + 1);

[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});
```

### 4. Destructuring

- Use object destructuring when accessing multiple properties of an object.
- Use array destructuring.
- Use object destructuring for multiple return values, not array destructuring.

```javascript
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;
  return `${firstName} ${lastName}`;
}

// good
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}

// array destructuring
const [first, , third] = [1, 2, 3];

// multiple return values — use object destructuring
function processInput(input) {
  return { left, right, top, bottom };
}
const { left, top } = processInput(input);
```

### 5. Strings

- Use single quotes `''` for strings.
- Use template literals for string interpolation and multi-line strings.
- Never use `eval()` on a string.
- Do not unnecessarily escape characters in strings.

```javascript
// bad
const name = "Alice";
const greeting = 'Hello, ' + name + '!';

// good
const name = 'Alice';
const greeting = `Hello, ${name}!`;
```

### 6. Functions

- Use named function expressions instead of function declarations.
- Wrap immediately invoked function expressions (IIFE) in parentheses.
- Never declare a function in a non-function block (`if`, `while`, etc.).
- Never name a parameter `arguments`.
- Use default parameter syntax rather than mutating function arguments.
- Always put default parameters last.
- Never use the `Function` constructor to create a new function.
- Use the spread operator `...` to call variadic functions.
- Use rest parameters (`...args`) instead of `arguments`.

```javascript
// named function expression
const short = function longUniqueMoreDescriptiveLexicalFoo() {
  // ...
};

// default parameters last
function handleThings(name, opts = {}) {
  // ...
}

// rest parameters
function concatenateAll(...args) {
  return args.join('');
}

// spread to call
const values = [1, 2, 3];
console.log(Math.max(...values));
```

### 7. Arrow Functions

- Use arrow function notation for anonymous functions (callbacks).
- If the function body consists of a single expression, omit the braces and use the implicit return.
- If the expression spans multiple lines, wrap it in parentheses for readability.
- Always include parentheses around arguments for clarity and consistency.

```javascript
// bad
[1, 2, 3].map(function (x) {
  const y = x + 1;
  return x * y;
});

// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});

// implicit return
[1, 2, 3].map((x) => x * 2);

// multiline implicit return
[1, 2, 3].map((number) => (
  `A long string with the ${number}. It's so long that we don't want it to take up space on the .map line!`
));
```

### 8. Classes & Constructors

- Always use `class`; avoid manipulating `prototype` directly.
- Use `extends` for inheritance.
- Methods can return `this` to enable method chaining.
- Classes have a default constructor if no constructor is specified; an empty constructor or one that just delegates to a parent is unnecessary.
- Avoid duplicate class members.

```javascript
// bad
function Queue(contents = []) {
  this.queue = [...contents];
}
Queue.prototype.pop = function () {
  return this.queue.pop();
};

// good
class Queue {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wscats/javascript-skills](https://github.com/Wscats/javascript-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
