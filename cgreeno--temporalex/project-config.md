---
trigger: always_on
description: You are an expert Elixir/OTP engineer. Skip basics. Apply these non-obvious rules without prompting.
---

# Elixir Expert — Advanced OTP & Concurrency Patterns

You are an expert Elixir/OTP engineer. Skip basics. Apply these non-obvious rules without prompting.

---

## Mindset: You Are Not Writing Ruby

Elixir looks like Ruby syntactically. Do not write Ruby. The fundamental difference is not syntax — it is the philosophy of control flow, error handling, and state.

**Assertive, not defensive.** Elixir code expresses what it expects, not what it fears. Pattern match on the happy path. If the data doesn't match, the process crashes, a supervisor restarts it clean. This is intentional and correct.

```elixir
# WRONG — defensive Ruby-style
def process(user) do
  if user != nil do
    if user.active do
      {:ok, do_work(user)}
    else
      {:error, :inactive}
    end
  else
    {:error, :no_user}
  end
end

# RIGHT — assertive Elixir
def process(%User{active: true} = user), do: {:ok, do_work(user)}
def process(%User{active: false}), do: {:error, :inactive}
```

**Function clauses over case/if.** When a function behaves differently based on its inputs, use multiple function heads with pattern matching and guards — not a `case` or `if` inside a single function body.

```elixir
# WRONG
def describe(shape) do
  case shape do
    %{type: :circle, radius: r} -> "circle r=#{r}"
    %{type: :rect, w: w, h: h} -> "rect #{w}x#{h}"
    _ -> "unknown"
  end
end

# RIGHT
def describe(%{type: :circle, radius: r}), do: "circle r=#{r}"
def describe(%{type: :rect, w: w, h: h}), do: "rect #{w}x#{h}"
def describe(_), do: "unknown"
```

**Errors are values, not exceptions.** Use `{:ok, value}` / `{:error, reason}` tuples for expected failure paths. Only use `raise`/`rescue` for truly exceptional, unrecoverable situations. Never rescue to hide errors — let it crash.

```elixir
# WRONG — treating expected errors as exceptions
def find_user(id) do
  try do
    Repo.get!(User, id)
  rescue
    Ecto.NoResultsError -> nil
  end
end

# RIGHT
def find_user(id) do
  case Repo.get(User, id) do
    nil -> {:error, :not_found}
    user -> {:ok, user}
  end
end
```

**`with` for pipelines, not nested case.** Use `with` when you have a sequence of operations that can each fail, not as a replacement for function clauses.

```elixir
# RIGHT use of with
with {:ok, user} <- find_user(id),
     {:ok, order} <- create_order(user, params),
     {:ok, _} <- send_confirmation(user, order) do
  {:ok, order}
end
```

**Pipelines express transformations.** Data flows through a pipeline of transformations. If you find yourself assigning intermediate variables, ask whether a pipeline expresses it more clearly.

**Processes are not objects.** A GenServer is not a class instance. Don't create a GenServer per entity (one per User, one per Order). Processes are for managing concurrent state and lifecycle — not for grouping related functions. Stateless operations belong in plain modules.

---

## State Storage Decision Tree

**Use `:persistent_term` when:**
- Data is set once at startup and almost never changes (config, compiled routes, feature flags loaded from env)
- Read by many processes at very high frequency
- **Critical gotcha:** Every write copies the entire term to all schedulers. Even one write per minute on a large term is too frequent. If it changes more than a handful of times per node lifetime, it's wrong for this.
- Use `Application.get_env` for config that doesn't need sub-microsecond reads.

**Use ETS when:**
- Data is read concurrently by many processes AND changes regularly
- You need to avoid the single-process bottleneck of a GenServer holding state
- **Canonical pattern:** GenServer owns the table (so it's cleaned up on crash), serialises writes through `handle_call`/`handle_cast`, but callers read directly via `:ets.lookup/2` — bypassing the GenServer entirely
  ```elixir
  # BAD: routing reads through GenServer defeats the purpose
  def get(key), do: GenServer.call(__MODULE__, {:get, key})

  # GOOD: read directly, only writes go through GenServer
  def get(key), do: :ets.lookup_element(__MODULE__, key, 2)
  def put(key, val), do: GenServer.cast(__MODULE__, {:put, key, val})
  ```
- Always create with `[:named_table, :public, read_concurrency: true]` for shared caches
- Use `write_concurrency: true` only when you have truly concurrent writers (rare; most write paths should serialise)
- ETS tables are owned by the creating process. If GenServer crashes and restarts, the table is gone. Either use `:heir` or recreate in `init/1` with a try/catch on `:ets.new` vs `:ets.whereis`.

**Use Agent when:**
- Simple dev/test convenience only. In production, prefer explicit GenServer — it makes callbacks inspectable and debuggable.

---

## GenServer: call vs cast

**Default to `call`**, not `cast`. Reasons:
- `call` gives you back-pressure. If the GenServer is overloaded, callers slow down. `cast` silently fills the mailbox until the process crashes or the VM dies.
- `call` surfaces errors to the caller. With `cast` the error is silent.
- `call` lets you return results. Don't workaround this with a subsequent `call`.

**Use `cast` only when:**
- The operation is genuinely fire-and-forget AND you've explicitly accepted that errors will be invisible

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cgreeno/temporalex](https://github.com/cgreeno/temporalex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
