---
trigger: always_on
description: This document outlines the standards and best practices for developing an Elixir library. These guidelines ensure the library remains reliable, maintainable, secure, and easy to integrate into other Elixir applications.
---

## Overview

This document outlines the standards and best practices for developing an Elixir library. These guidelines ensure the library remains reliable, maintainable, secure, and easy to integrate into other Elixir applications.

This is a pure Elixir library intended to be used as a dependency in other Elixir applications. It is **not** a Phoenix or Nerves project. Instead, it focuses on providing functional building blocks using idiomatic Elixir and OTP constructs.

## Core Principles

- Write clean, composable, and testable code
- Adhere to functional programming principles—avoid OOP patterns
- Maintain clear boundaries between modules and domains
- Ensure code is robust, secure, and easy to reason about
- Provide rich documentation and helpful logging
- Create libraries that integrate seamlessly into any Elixir application

## Project Structure

### Directory Layout

```
.
├── lib/
│   ├── your_library/
│   │   ├── core/           # Core functionality and behaviors
│   │   ├── components/     # Main component modules
│   │   ├── otp/           # OTP components (supervisors, workers)
│   │   ├── utils/         # Utility functions and helpers
│   │   └── types/         # Custom types and specs
│   └── your_library.ex     # Main entry point
├── test/
│   ├── your_library/
│   │   ├── core/          # Tests mirroring lib structure
│   │   ├── components/
│   │   └── otp/
│   ├── support/           # Test helpers and shared fixtures
│   └── test_helper.exs
├── mix.exs
└── mix.lock
```

### Structural Guidelines

- **Data First**: Define data structures and types before implementing operations on them
- **Module Organization**: Group modules by domain or functionality
- **Test Mirroring**: Tests should mirror the directory structure
- **Minimal Dependencies**: Avoid circular dependencies between modules
- **Clear Boundaries**: Each module should have a single responsibility

## Code Organization

### Data Structure Definition

1. Start with pure data structures using structs:

```elixir
defmodule YourLibrary.Types.Task do
  use TypedStruct

  typedstruct do
    field :id, String.t()
    field :name, String.t()
    field :status, :pending | :running | :completed
    field :created_at, DateTime.t()
  end

  @type validation_error :: 
    :invalid_name |
    :invalid_status |
    {:invalid_date, String.t()}

  @spec validate(t()) :: :ok | {:error, validation_error()}
  def validate(%__MODULE__{} = task) do
    # Validation logic
  end
end
```

2. Then define modules that operate on these structures:

```elixir
defmodule YourLibrary.Core.TaskOperations do
  alias YourLibrary.Types.Task
  
  @spec create_task(String.t()) :: {:ok, Task.t()} | {:error, Task.validation_error()}
  def create_task(name) do
    task = %Task{
      id: generate_id(),
      name: name,
      status: :pending,
      created_at: DateTime.utc_now()
    }
    
    case Task.validate(task) do
      :ok -> {:ok, task}
      {:error, _reason} = error -> error
    end
  end
end
```

3. Finally, implement process lifecycle modules:

```elixir
defmodule YourLibrary.OTP.TaskManager do
  use GenServer
  alias YourLibrary.Core.TaskOperations
  
  def start_link(opts) do
    GenServer.start_link(__MODULE__, opts, name: __MODULE__)
  end
  
  @impl true
  def init(opts) do
    {:ok, %{tasks: %{}, opts: opts}}
  end
  
  # ... rest of GenServer implementation
end
```

### Function Heads and Guards

Use multiple function heads for clarity and control flow:

```elixir
defmodule YourLibrary.Core.DataProcessor do
  # Match on specific values
  def process(:empty), do: {:ok, []}
  
  # Use guards for type checking
  def process(data) when is_list(data) do
    {:ok, Enum.map(data, &transform/1)}
  end
  
  # Pattern match on complex structures
  def process(%{items: items, status: :ready} = data)
    when is_list(items) and length(items) > 0 do
    {:ok, process_items(items, data)}
  end
  
  # Catch-all case
  def process(_invalid) do
    {:error, :invalid_input}
  end
  
  # Private functions can also use guards
  defp transform(item) when is_binary(item) do
    String.upcase(item)
  end
  
  defp transform(item) when is_integer(item) do
    Integer.to_string(item)
  end
end
```

### Behaviors

Define behaviors to establish contracts between modules:

```elixir
defmodule YourLibrary.Core.Processor do
  @doc """
  Defines the contract for processing data.
  """
  @callback process(data :: term()) :: 
    {:ok, term()} | 
    {:error, term()}
    
  @doc """
  Optional callback for data validation.
  """
  @callback validate(input :: term()) :: 
    :ok | 
    {:error, term()}
    
  @optional_callbacks validate: 1
  
  # Can include default implementations
  defmacro __using__(_opts) do
    quote do
      @behaviour YourLibrary.Core.Processor
      
      # Default implementation for validate
      @impl true
      def validate(_input), do: :ok
      
      # Allow overrides
      defoverridable validate: 1
    end
  end
end

# Implementation example
defmodule YourLibrary.Core.StringProcessor do
  use YourLibrary.Core.Processor
  
  @impl true
  def process(data) when is_binary(data) do

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentjido/jido_chat_deprecated](https://github.com/agentjido/jido_chat_deprecated) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
