---
trigger: always_on
description: Run a council review as a game-theoretic decision process: players, incentives, strategies, payoffs, dominated options, stability claims, mechanisms, and verification tests.
---


# Game-Theoretic Council

Use this skill when a decision needs structured multi-perspective review and a test of whether the recommendation survives incentives.

The council pattern is useful because different roles notice different failure modes. Game theory is useful because consensus is not stability. A recommendation is only strong when it still works after each relevant actor follows their own incentives, constraints, and information.

## Trigger conditions

Load this skill for:

- product, roadmap, positioning, or prioritization decisions;
- multi-agent, delegation, profile, team, or governance design;
- authority, permissioning, safety, or approval-boundary choices;
- adversarial, abuse, or red-team analysis;
- stakeholder conflict, bargaining, or coordination problems;
- expensive, irreversible, ambiguous, or high-trust decisions;
- council-style reviews that risk becoming a list of opinions instead of a decision model.

Do not use it for simple factual lookup or routine execution when the next step is obvious and reversible.

## Core rule

> Council proposes and critiques. Game theory decides whether the recommendation survives incentives.

A valid review names:

1. the decision question;
2. players and incentives;
3. candidate strategies;
4. payoff dimensions;
5. information asymmetries;
6. likely counter-moves;
7. dominated options;
8. equilibrium or stability claim;
9. mechanisms that make the desired behavior stable;
10. verification tests and stop conditions.

## Fast path

Use this shape when time is limited:

```markdown
# Game-Theoretic Council: <decision>

## Decision

## Players and incentives

## Candidate strategies

## Dominated options

## Stability claim

## Recommendation

## Verification tests
```

## Full process

### 1. Frame the decision

State the decision as a concrete question.

Good:

> Should this system use direct live mutation or a reviewed ChangeSet workflow?

Bad:

> What is the best architecture?

### 2. Identify players

Include everyone who can act, benefit, lose, exploit, maintain, inherit, constrain, or verify the decision.

Common player classes:

- user, owner, or principal;
- assistant, operator, or delegated agent;
- future maintainers;
- reviewers and verifiers;
- customers or downstream users;
- external competitors or baselines;
- attackers, abusers, or misuse cases;
- infrastructure, budget, policy, and time constraints.

For each player, name incentives and powers.

```yaml
players:
  - name: owner
    incentives: [useful outcome, low risk, low maintenance burden]
    powers: [approve scope, accept risk, stop work]
  - name: delegated_agent
    incentives: [complete assigned task, appear successful]
    powers: [write files, call tools within scope]
  - name: verifier
    incentives: [catch false claims, preserve trust]
    powers: [block completion, demand proof]
```

### 3. Define candidate strategies

Name the real available moves. Include tempting bad options because they expose dominated-strategy checks.

```yaml
strategies:
  - do_nothing
  - direct_live_mutation
  - reviewed_changeset_workflow
  - fully_autonomous_execution
```

### 4. Choose payoff dimensions

Use multiple dimensions. Avoid one blended score too early.

Useful default dimensions:

| Dimension | Meaning |
|---|---|
| user_value | How much the strategy helps the intended user. |
| strategic_value | Whether it creates durable advantage or clarity. |
| safety | Whether it prevents harm, drift, misuse, or false success. |
| implementation_cost | Build and maintenance burden. |
| reversibility | Whether mistakes can be undone. |
| auditability | Whether future readers can reconstruct what happened. |
| delegation_correctness | Whether authority and responsibility stay aligned. |
| long_term_value | Whether the choice compounds across future work. |

Scores are scaffolding, not proof. Explain the score or omit it.

### 5. Classify the game

Use the model that fits. Do not force the wrong one.

| Game type | Use when |
|---|---|
| Principal-agent | One actor delegates work or authority to another. |
| Repeated game | Trust, maintenance, reputation, or future interactions matter. |
| Mechanism design | Rules/defaults must make desired behavior easier than bad behavior. |
| Coordination game | Multiple actors need compatible choices or shared state. |
| Bargaining / coalition | Stakeholders have partially aligned interests and possible vetoes. |
| Bayesian game | Players have incomplete or private information. |
| Sequential / Stackelberg | One actor moves first and others respond. |
| Zero-sum / minimax | Security, abuse, exploitation, or direct adversaries dominate. |

### 6. Run council roles

Use only roles that materially help. Five sharp roles are better than twelve shallow personas.

#### Proposer

- Best candidate strategy.
- Intended payoff.
- Assumptions required.
- What success looks like.

#### Red Team / Adversary

- Likely exploit or counter-move.
- How the plan fails if actors optimize locally.
- Worst credible outcome.
- Mitigation or stop condition.

#### Mechanism Designer

- Desired behavior.
- Current incentives.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaverickKB/game-theoretic-council-skill](https://github.com/MaverickKB/game-theoretic-council-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
