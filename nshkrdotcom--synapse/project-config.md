---
trigger: always_on
description: - `/agentjido/` is kept out of version control via the `.gitignore` entry at `/agentjido/`, but it is available locally for experimentation.
---

# Agents Reference

## Local Agent Workspace

- `/agentjido/` is kept out of version control via the `.gitignore` entry at `/agentjido/`, but it is available locally for experimentation.
- The directory contains three sibling libraries that power our agent work: `agentjido/jido`, `agentjido/jido_signal`, and `agentjido/jido_action`.
- The Jido guides live under `agentjido/jido/guides` (explore that folder for detailed walkthroughs).

● **Agents vs Pipelines: The Architecture of World Domination 🌍**

You're asking the right question. Let me break down the fundamental differences and show you how to actually build toward multi-agent dominance.

---

**Core Concepts**

**Pipelines (Runners)**

What: Static, deterministic data transformations  
Decision-making: Zero - always follows the same path  
Tools: No - just pure functions chained together

```elixir
# Pipeline: Always does A -> B -> C
input
|> validate()
|> transform()
|> format()
|> output()
```

When to use:
- You know the exact steps beforehand
- No decisions needed
- Same path every time
- Performance critical

**Agents**

What: Autonomous entities with state, goals, and decision-making  
Decision-making: High - chooses actions based on context  
Tools: Yes - can select and use different tools

```elixir
# Agent: Decides what to do based on input
agent
|> observe(situation)
|> reason() # "Is this simple? Complex? Urgent?"
|> decide_action() # Pick: quick_review OR deep_analysis OR escalate
|> use_tool(selected_action)
|> learn_from_result()
```

When to use:
- Need decision-making
- Adapt to different scenarios
- Learn from experience
- Coordinate with other agents

---

**The Fundamental Difference**

Pipeline: "Do these steps"

```elixir
# Always the same, no choices
def pipeline(input) do
  input
  |> step1()
  |> step2()
  |> step3()
end
```

Agent: "Achieve this goal"

```elixir
# Different paths based on reasoning
def agent_decide(input, agent_state) do
  case analyze_situation(input) do
    :simple ->
      agent |> use_tool(QuickReview)

    :complex ->
      agent
      |> use_tool(DeepAnalysis)
      |> maybe(use_tool(SecurityCheck))
      |> maybe(use_tool(PerformanceCheck))

    :urgent ->
      agent
      |> use_tool(FastTrack)
      |> signal_other_agent(:escalate)
  end
end
```

---

**Your Current State (Honest Assessment)**

What You Have Now

```elixir
# This is a PIPELINE dressed as agents
def evaluate(input) do
  input
  |> SimpleExecutor.cmd(Echo)        # Always echo
  |> CriticAgent.cmd(CriticReview)   # Always review
  |> GenerateCritique.run()          # Always LLM
end
```

Reality: This is a static pipeline using agent infrastructure. The current Synapse runtime already ships with a declarative
orchestrator (`priv/orchestrator_agents.exs`) that decides which specialists run for each review. Instead of hand-written
GenServers like `CoordinatorAgentServer` or `SecurityAgentServer`, every agent is now described as configuration.

```elixir
# priv/orchestrator_agents.exs (excerpt)
%{
  id: :coordinator,
  type: :orchestrator,
  actions: [Synapse.Actions.Review.ClassifyChange],
  orchestration: %{
    classify_fn: &Synapse.Orchestrator.Config.Classifier.fast_or_deep/1,
    spawn_specialists: [:security_specialist, :performance_specialist],
    aggregation_fn: &Synapse.Orchestrator.Config.Aggregation.combine/2
  },
  signals: %{subscribes: [:review_request, :review_result], emits: [:review_summary]}
}
```

That declarative Config is what the runtime executes today; the rest of this document explains how to evolve it into
multi-agent dominance (specialist negotiation, learning, etc.).

What You Should Have (Multi-Agent)

```elixir
# Real multi-agent: the declarative runtime decides everything
defmodule Synapse.Orchestrator.Config do
  def coordinator_spec do
    %{
      id: :coordinator,
      type: :orchestrator,
      actions: [Synapse.Actions.Review.ClassifyChange],
      orchestration: %{
        classify_fn: &Strategies.classify/1,
        spawn_specialists: &Strategies.choose_team/2,
        fast_path_fn: &Strategies.fast_path/2,
        aggregation_fn: &Strategies.aggregate/2,
        negotiate_fn: &Strategies.resolve_conflicts/2
      },
      signals: %{subscribes: [:review_request, :review_result], emits: [:review_summary]}
    }
  end
end
```

```elixir
# Runtime consumes that config – no GenServers required
{:ok, _runtime} =
  Synapse.Orchestrator.Runtime.start_link(
    config_source: {:priv, "orchestrator_agents.exs"},
    include_types: :all,
    router: :synapse_router,
    registry: :synapse_registry
  )

# Sending a review request is just a signal publish
Synapse.SignalRouter.publish(
  :synapse_router,
  :review_request,
  %{review_id: "123", diff: diff, labels: ["security"]}
)

# RunConfig + Workflow.Engine classify, spawn specialists, and emit review.summary
```

---

**Tools: The Agent's Superpowers**

What Are Tools?

In agent systems: Tools are capabilities an agent can choose to use.

```elixir
# Agent has a toolbox
defmodule CriticAgent do
  use Jido.Agent,
    actions: [
      # These are TOOLS the agent can use
      StaticAnalysis,
      SecurityScan,
      PerformanceBenchmark,
      LLMReview,
      GitBlame,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nshkrdotcom/synapse](https://github.com/nshkrdotcom/synapse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
