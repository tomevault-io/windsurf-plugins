---
trigger: always_on
description: Soli is a dynamically-typed, high-performance web framework and programming language written in Rust. It combines Ruby-like expressiveness with excellent performance (170k+ req/sec).
---

# Soli Lang

Soli is a dynamically-typed, high-performance web framework and programming language written in Rust. It combines Ruby-like expressiveness with excellent performance (170k+ req/sec).

## Project Structure

```
src/                    # Rust interpreter source
├── ast/               # Abstract Syntax Tree
├── lexer/             # Tokenizer
├── parser/            # Parser
├── interpreter/       # Runtime (builtins in interpreter/builtins/)
├── vm/                # Virtual machine
└── template/          # ERB template engine
tests/                 # Soli test files (.sl)
examples/              # Example Soli programs
stdlib/                # Standard library
template/              # MVC project template (used by `soli new`)
www/                   # Documentation website
```

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Files | `snake_case.sl` | `home_controller.sl` |
| Classes | `PascalCase` | `UsersController` |
| Functions | `snake_case` | `get_user_by_id` |
| Constants | `SCREAMING_SNAKE_CASE` | `MAX_CONNECTIONS` |

## Key Syntax

### Variables and Types
```soli
let name = "Alice";           // Type inference
let age: Int = 30;           // Explicit type annotation
const MAX_SIZE = 100;        // Constants (immutable)
```

### Functions
```soli
fn add(a: Int, b: Int) -> Int {
    return a + b;
}

// Optional parentheses for no-param functions
fn greet {
    "Hello!"
}

// Default and named parameters
fn configure(host: String = "localhost", port: Int = 8080) -> Void {
    print("Connecting to \(host):\(port)");
}
configure(port: 3000, host: "example.com");  // Named params in any order
```

### Lambdas and Pipelines
```soli
// fn() syntax
let add = fn(a, b) { return a + b; };

// Pipe syntax
let double = |x| { return x * 2; };

// Pipeline for chaining
let result = 5 |> double() |> addOne();
[1, 2, 3] |> map(fn(x) x * 2) |> filter(fn(x) x > 2);
```

### Collections

**Arrays:**
```soli
let numbers = [1, 2, 3, 4, 5];
let combined = [...a, ...b];

numbers.map(fn(x) x * 2);
numbers.filter(fn(x) x > 2);
numbers.each(fn(x) print(x));
```

**Hashes:**
```soli
let person = {"name": "Alice", "age": 30};
person.name;           // Dot notation preferred
person["name"];        // Bracket notation also works
person.keys();
person.has_key("name");
```

### Classes
```soli
class Person {
    name: String;
    age: Int;
    
    new(name: String, age: Int) {
        this.name = name;
        this.age = age;
    }
    
    fn greet() -> String {
        return "Hello, I'm " + this.name;
    }
}

class Employee extends Person {
    salary: Float;
    
    new(name: String, age: Int, salary: Float) {
        super(name, age);
        this.salary = salary;
    }
}
```

### Control Flow
```soli
// Postfix conditionals (idiomatic)
print("adult") if age >= 18;
print("minor") unless age >= 18;

// Pattern matching
let result = match value {
    42 => "the answer",
    n if n > 0 => "positive: " + str(n),
    [first, ...rest] => "first: " + str(first),
    {name: n, age: a} => n + " is " + str(a),
    _ => "wildcard"
};
```

### Error Handling
```soli
try {
    risky_operation();
} catch error {
    print("Error: " + str(error));
} finally {
    cleanup();
}
```

## Best Practices

1. **Use type annotations** - Catch errors early, improve readability
   ```soli
   fn process_user(user: Hash) -> Hash { ... }
   ```

2. **Prefer immutability** - Use `const` for values that shouldn't change

3. **Chain iteration methods** - Avoid manual loops when possible
   ```soli
   let result = data
       .filter(fn(x) x["active"])
       .map(fn(x) x["name"])
       .join(", ");
   ```

4. **Use pattern matching** - Cleaner than nested if/elsif for complex conditionals

5. **Use named parameters** - For functions with multiple optional params

6. **Implicit returns** - Last expression in a function block is automatically returned
   ```soli
   fn get_greeting(name) {
       "Hello, " + name + "!"  // No return needed
   }
   ```

7. **Use pipelines** - For readable data transformation chains

## SOLID Principles

Apply these OOP design principles for maintainable code:

**Single Responsibility (S)** - Each class does one thing:
```soli
class UserValidator { /* only validation */ }
class UserRepository { /* only database operations */ }
```

**Open/Closed (O)** - Open for extension, closed for modification:
```soli
class Shape { fn area() -> Float; }
class Circle extends Shape { radius: Float; fn area() { 3.14 * radius * radius; } }
```

**Liskov Substitution (L)** - Subclasses can replace their parent:
```soli
# Don't override methods with incompatible behavior
```

**Interface Segregation (I)** - Many small interfaces over one large:
```soli
interface Printable { fn print(); }
interface Exportable { fn export(); }
```

**Dependency Inversion (D)** - Depend on abstractions:
```soli
interface Repository { fn find(id: Int) -> User; }
class Service { repo: Repository; fn get(id) { repo.find(id); } }
```

## Linting

Run `soli lint` to check code for issues:

```bash
soli lint                    # Lint entire project

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/solisoft) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
