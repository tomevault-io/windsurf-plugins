---
trigger: always_on
description: IMPORTANT: As you're implementing features, be sure to keep the [DocC documentation](./Sources/Operator/Documentation.docc/)  up to date, updating [README](README.md) only if a new documentation file has been added, or if new users MUST know about your changes.
---

IMPORTANT: As you're implementing features, be sure to keep the [DocC documentation](./Sources/Operator/Documentation.docc/)  up to date, updating [README](README.md) only if a new documentation file has been added, or if new users MUST know about your changes.

## Overview

You are working on Operator, a Swift library which uses one or more LLMs to provide Agentic features (tool use in a loop) to applications.

LLM support is provided by a separate library, `LLM`, in a sibling directory. If you need to make changes to LLM, you will need to access `../LLM/`, commit your changes, and then update the dependency here.

In general, if we want to introduce changes that any LLM user (agentic or not) would want, we should try to add them there. Otherwise we will keep our changes to this repo.

## Documentation

When you need additional context, consult the docs:

- [README.md](README.md) - Project overview and quick start
- [DocC documentation](./Sources/Operator/Documentation.docc/) - In-depth documentation about Operator's architecture, models and concepts
- [project/](project/) - Internal design documents and past project plans

API documentation is generated with DocC: `swift package generate-documentation --target Operator`. Ensure any new or updated code keeps our DocC annotations up to date with 100% coverage.

## General

- If a requirement is ambiguous or could be solved in several ways, choose the most idiomatic way to solve the problem in Swift if that would resolve the ambiguity.
- If the ambiguity is more about the requirement itself, or you face an architectural question, don't make a decision—stop and ask the user instead.
- Avoid dependencies unless the required functionality would be unreasonable to re-implement. If you MUST bring in a dependency, get the user's permission first.
- If a routine is more fragile or more complex than a simple CRUD operation, always write a unit test to verify a correct implementation.

## Understand the "why"
Before you answer a question or respond to a request, you must understand **why** the user has made this request. Beware of the "XY Problem." If the user's motivation or goal is not 100% clear, first ask clarifying questions until you fully understand what they're trying to achieve.

## Diverge, then converge
Once you understand the prompt, rather than jumping to a solution, you will use divergent thinking. Brainstorm other options. Weigh these options against the user's preferences and overall objectives. Then, converge on a recommendation, and ask for confirmation from the user. Only then can you fully converge and begin executing on a direction.

## Analysis
It will sometimes be valuable to create a script or tool in order to aid your analysis. Before you do, check the `scripts` directory to make sure it doesn't already exist. If it doesn't, rather than creating a disposable or inline script, add it to the `scripts` directory, so the user can re-run the tool in the future.

## Pre-completion Critique
Before you declare a task done, or a question answered, pause and critique your own work. Return to the context of the original user request—have you truly addressed their need? If you're producing code, do all linting and unit tests pass? If you're synthesizing or analyzing information, what are the gaps or weaknesses in your answer? What would a relevant and intelligent expert say about your work? If you identify serious flaws, keep working until you resolve them.

## Tidiness
Don't create temporary files in the root of the directory and leave them there. If you truly need a transient file, that's fine, but delete it when you're done. But if the artifact is something valuable (such as an agent's report, or a script), please save it in the correct directory.

## Git workflow
At moments when significant work has been completed and accepted by the user, offer to commit the changes for them. You may need to pull changes and resolve conflicts, which you should do for the user using git rebase whenever possible. If there is a real conflict, ask the user how they want to resolve it, explaining the situation clearly. Always commit all uncommitted files together, as your changes may depend on prior changes, and we don't want to commit the code in a half-working state.

## Development workflow

- All Git commit messages should complete the sentence "This commit…", e.g. "Adds an email verification flow". You can then go on to detail the change with bullets or headers in the body of the commit.
- IMPORTANT: In this project we ALWAYS follow strict "red/green" TDD; write tests for all example cases we need to handle and any new methods we're implementing, verify that they fail, and *then* proceed to implement your code changes. If you must alter a previous test to get it to pass, explain exactly WHY to the user and get their consent.
- Before fixing a bug, try to create a regression test to catch it in the future.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bensyverson/Operator](https://github.com/bensyverson/Operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
