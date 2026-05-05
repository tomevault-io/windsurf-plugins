---
trigger: always_on
description: **ALWAYS write a failing test before fixing a bug.**
---

# JustBash Development Guidelines

## Non-Negotiable Rules

### 1. Test-First Bug Fixing
**ALWAYS write a failing test before fixing a bug.**
```
1. Discover bug
2. Write test that reproduces it (must FAIL)
3. Verify failure
4. Fix bug
5. Verify test passes
6. Full suite passes - no regressions
```

### 2. Zero Warnings — All Tooling Must Pass
```bash
mix compile --warnings-as-errors   # Zero compiler warnings
mix dialyzer                       # Zero dialyzer errors (skips in .dialyzer_ignore.exs are OK)
mix credo --strict                 # Clean (only intentional test fixture findings)
mix format --check-formatted       # Properly formatted
mix test                           # All tests pass
```
**Run all five before committing. No exceptions.** A passing test suite alone is not
sufficient — dialyzer catches type errors and dead code that tests miss.

### 3. Trust Tests, Not Code
We do not trust the implementation. Tests are the only source of confidence. Write adversarial tests. Compare against real bash when possible.

---

## Idiomatic Elixir

### Let It Crash
Don't defensively code against every possible error. Let processes crash and be restarted. Use supervisors. Handle expected errors, crash on unexpected ones.

```elixir
# Good - crash on unexpected
def fetch!(key), do: Map.fetch!(data, key)

# Good - handle expected errors at boundaries
case parse_input(raw) do
  {:ok, parsed} -> process(parsed)
  {:error, :invalid_format} -> {:error, "Invalid input format"}
end
```

### Pattern Matching Everywhere
Pattern match in function heads, `case`, `with`, and assignments:

```elixir
# Function heads - preferred
def handle(%{type: :success, data: data}), do: {:ok, data}
def handle(%{type: :error, reason: reason}), do: {:error, reason}

# Destructure in assignments
%{name: name, age: age} = user
[head | tail] = list
{:ok, result} = must_succeed()
```

### Pipeline Orientation
Design functions to take the "subject" as first argument for pipeline compatibility:

```elixir
# Good - pipeable
def add_header(request, name, value)
def parse(string, opts \\ [])

request
|> add_header("Content-Type", "application/json")
|> add_header("Authorization", token)
|> send()

# Avoid - not pipeable
def add_header(name, value, request)
```

### Transform, Don't Mutate
Think in data transformations, not state mutations:

```elixir
# Good - transform
def process(state) do
  state
  |> validate()
  |> enrich()
  |> persist()
end

# Avoid - imperative mutation style
def process(state) do
  state = validate(state)
  state = enrich(state)
  persist(state)
end
```

### Use `with` for Happy Path
Chain operations that may fail, bail on first error:

```elixir
with {:ok, user} <- fetch_user(id),
     {:ok, account} <- fetch_account(user),
     :ok <- verify_permissions(account) do
  {:ok, account}
else
  {:error, :not_found} -> {:error, "User not found"}
  {:error, :forbidden} -> {:error, "Access denied"}
end
```

### Keyword Lists for Options, Maps for Data
```elixir
# Options - keyword list
def connect(host, opts \\ []) do
  port = Keyword.get(opts, :port, 443)
  timeout = Keyword.get(opts, :timeout, 5000)
end

# Data - map
user = %{name: "Alice", email: "alice@example.com"}
```

### Structs for Domain Entities
```elixir
defmodule User do
  @type t :: %__MODULE__{
    id: pos_integer(),
    name: String.t(),
    email: String.t()
  }
  
  @enforce_keys [:id, :name, :email]
  defstruct [:id, :name, :email]
end
```

### Behaviours for Contracts
```elixir
defmodule Command do
  @callback execute(bash :: JustBash.t(), args :: [String.t()], stdin :: String.t()) ::
    {result(), JustBash.t()}
end

defmodule Echo do
  @behaviour Command
  
  @impl true
  def execute(bash, args, _stdin), do: ...
end
```

---

## Code Style

### Function Clauses
Group clauses together. Order from specific to general:

```elixir
# Specific first
def parse("true"), do: {:ok, true}
def parse("false"), do: {:ok, false}
def parse(<<"0x", hex::binary>>), do: parse_hex(hex)
def parse(str) when is_binary(str), do: parse_string(str)
def parse(_), do: {:error, :invalid}
```

### Guards
Use guards for type checks and simple conditions:

```elixir
def process(n) when is_integer(n) and n > 0, do: ...
def process(s) when is_binary(s), do: ...
def process(list) when is_list(list) and length(list) > 0, do: ...
```

### Private Functions
Prefix with `do_` for recursive helpers or implementation details:

```elixir
def traverse(tree), do: do_traverse(tree, [])

defp do_traverse(nil, acc), do: Enum.reverse(acc)
defp do_traverse(%{left: l, right: r, value: v}, acc) do
  acc = do_traverse(l, acc)
  acc = [v | acc]
  do_traverse(r, acc)
end
```

### Reduce Patterns
```elixir
# Simple accumulation
Enum.reduce(items, 0, fn item, sum -> sum + item.value end)

# Building a map
Enum.reduce(items, %{}, fn item, acc ->
  Map.put(acc, item.id, item)
end)

# Early termination
Enum.reduce_while(items, {:ok, []}, fn item, {:ok, acc} ->
  case validate(item) do
    :ok -> {:cont, {:ok, [item | acc]}}
    {:error, _} = err -> {:halt, err}
  end
end)
```

### Comprehensions
Use for filtering and transformation:

```elixir
# Filter and transform
for user <- users, user.active, do: user.email

# Multiple generators (cartesian product)
for x <- 1..3, y <- 1..3, do: {x, y}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elixir-ai-tools/just_bash](https://github.com/elixir-ai-tools/just_bash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
