---
trigger: always_on
description: SPDX-License-Identifier: Apache-2.0
---

<!--
  SPDX-License-Identifier: Apache-2.0
  SPDX-FileCopyrightText: 2021 The Elixir Team
-->

# Simple state with agents

In this chapter, we will learn how to keep and share state between multiple entities. If you have previous programming experience, you may think of globally shared variables, but the model we will learn here is quite different. The next chapters will generalize the concepts introduced here.

If you have skipped the *Getting Started* guide or read it long ago, be sure to re-read the [Processes](../getting-started/processes.md) chapter. We will use it as a starting point.

## The trouble with (mutable) state

Elixir is an immutable language where nothing is shared by default. If we want to share information, this is typically done by sending messages between processes.

When it comes to processes though, we rarely hand-roll our own, instead we use the abstractions available in Elixir and OTP:

  * `Agent` — Simple wrappers around state.
  * `GenServer` — "Generic servers" (processes) that encapsulate state, provide sync and async calls, support code reloading, and more.
  * `Task` — Asynchronous units of computation that allow spawning a process and potentially retrieving its result at a later time.

Here, we will use agents, and create a module named `KV.Bucket`, responsible for storing our key-value entries in a way that allows them to be read and modified by other processes.

## Agents 101

`Agent`s are simple wrappers around state. If all you want from a process is to keep state, agents are a great fit. Let's start a `iex` session inside the project with:

```console
$ iex -S mix
```

And play a bit with agents:

```elixir
iex> {:ok, agent} = Agent.start_link(fn -> [] end)
{:ok, #PID<0.57.0>}
iex> Agent.update(agent, fn list -> ["eggs" | list] end)
:ok
iex> Agent.get(agent, fn list -> list end)
["eggs"]
iex> Agent.stop(agent)
:ok
```

We started an agent with an initial state of an empty list. The `start_link/1` function returned the `:ok` tuple with a process identifier (PID) of the agent. We will use this PID for all further interactions. We then updated the agent's state, adding our new item to the head of the list. The second argument of `Agent.update/3` is a function that takes the agent's current state as input and returns its desired new state. Finally, we retrieved the whole list. The second argument of `Agent.get/3` is a function that takes the state as input and returns the value that `Agent.get/3` itself will return. Once we are done with the agent, we can call `Agent.stop/3` to terminate the agent process.

The `Agent.update/3` function accepts as a second argument any function that receives one argument and returns a value:

```elixir
iex> {:ok, agent} = Agent.start_link(fn -> [] end)
{:ok, #PID<0.338.0>}
iex> Agent.update(agent, fn _list -> 123 end)
:ok
iex> Agent.update(agent, fn content -> %{a: content} end)
:ok
iex> Agent.update(agent, fn content -> [12 | [content]] end)
:ok
iex> Agent.update(agent, fn list -> [:nop | list] end)
:ok
iex> Agent.get(agent, fn content -> content end)
[:nop, 12, %{a: 123}]
```

As you can see, we can modify the agent state in any way we want. Therefore, we most likely don't want to access the Agent API throughout many different places in our code. Instead, we want to encapsulate all Agent-related functionality in a single module, which we will call `KV.Bucket`. Before we implement it, let's write some tests which will outline the API exposed by our module.

Create a file at `test/kv/bucket_test.exs` (remember the `.exs` extension) with the following:

```elixir
defmodule KV.BucketTest do
  use ExUnit.Case, async: true

  test "stores values by key" do
    {:ok, bucket} = KV.Bucket.start_link([])
    assert KV.Bucket.get(bucket, "milk") == nil

    KV.Bucket.put(bucket, "milk", 3)
    assert KV.Bucket.get(bucket, "milk") == 3
  end
end
```

`use ExUnit.Case` is responsible for setting up our module for testing and imports many test-related functionality, such as the `test/2` macro.

Our first test starts a new `KV.Bucket` by calling the `start_link/1` and passing an empty list of options. Then we perform some `get/2` and `put/3` operations on it, asserting the result.

Also note the `async: true` option passed to `ExUnit.Case`. This option makes the test case run in parallel with other `:async` test cases by using multiple cores in our machine. This is extremely useful to speed up our test suite. However, `:async` must *only* be set if the test case does not rely on or change any global values. For example, if the test requires writing to the file system or access a database, keep it synchronous (omit the `:async` option) to avoid race conditions between tests.

Async or not, our new test should obviously fail, as none of the functionality is implemented in the module being tested:

```text
1) test stores values by key (KV.BucketTest)
   test/kv/bucket_test.exs:4
   ** (UndefinedFunctionError) function KV.Bucket.start_link/1 is undefined (module KV.Bucket is not available)
```

In order to fix the failing test, let's create a file at `lib/kv/bucket.ex` with the contents below. Feel free to give a try at implementing the `KV.Bucket` module yourself using agents before peeking at the implementation below.

```elixir
defmodule KV.Bucket do
  use Agent


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elixir-lang/elixir](https://github.com/elixir-lang/elixir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
