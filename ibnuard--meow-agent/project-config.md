---
trigger: always_on
description: > The canonical rulebook for anyone (human or AI) writing code in Meow Agent.
---

# AGENTS_2.md — Meow Agent Development Rules

> The canonical rulebook for anyone (human or AI) writing code in Meow Agent.
> Every change MUST conform to these rules. When a rule here conflicts with
> older docs (the now-removed AGENTS.md / SKILLS.md), **this file wins**.
>
> Companion docs:
> - **[DESIGN_2.md](./DESIGN_2.md)** — visual language, layout, spacing, components.
> - **[ARCHITECTURE_2.md](./ARCHITECTURE_2.md)** — runtime, LLM, agent, data flow.

---

## 0. The Five Non-Negotiables

1. **Accuracy over everything.** Never hallucinate a capability or claim success you cannot verify. Missing data / absent capability → say so honestly and stop. No guessing, no fabricating, no silent retry.
2. **Language-generic, always.** No per-language word lists, no per-case patches, no language-specific branches in engine, routing, or prompts.
3. **Reusable first.** Before writing a widget, helper, or prompt, find the existing one. Duplication is a defect.
4. **One source of truth per concern.** UI copy → `AppStrings`. Prompts → `prompt_*` files. Tool existence → `ModulePlugin`. Permissions → the gate maps. Never fork these.
5. **Verify before declaring done.** State re-check (snapshot probe, registry re-read, result-data keys) gates every mutation. The LLM's "done" is not proof.

---

## 1. Localization Rules (STRICT)

### 1.1 `isId` must NEVER appear in a screen or widget

`isId` is a private detail of `AppStrings` (`code == 'id'`). It exists so the
string class can pick a variant. It must not leak into presentation code.

**Banned in `lib/features/**/presentation/**` and `lib/app/widgets/**`:**
- `final isId = resolveLanguageCode(...) == 'id';`
- `AppStrings(isId ? 'id' : 'en')`
- a `bool isId` parameter threaded through a widget constructor
- `isId: s.isId` passed into a child widget

**Why:** every `isId` branch in a screen is a place a translation can silently
go wrong, and it spreads the language concept across the UI instead of keeping
it sealed inside `AppStrings`. The string class already knows the language —
ask it for the finished string, never for the language flag.

### 1.2 The only correct pattern

Resolve once at the top of `build` (or once per widget that needs copy), then
read finished strings:

```dart
@override
Widget build(BuildContext context, WidgetRef ref) {
  final langPref = ref.watch(appLanguageProvider);
  final s = AppStrings(resolveLanguageCode(langPref));
  // ...
  return Text(s.agentListTitle);   // finished string, no flag
}
```

If a child widget needs copy, **pass the finished `AppStrings s`** (or the
specific finished strings), never `isId`. A widget that needs three labels
takes three resolved `String`s or the `AppStrings` instance — not a bool.

### 1.3 No hardcoded natural-language strings

No Indonesian or English literal may appear in a widget tree. Every
user-facing string is a getter/method on `AppStrings`
(`lib/features/settings/data/app_language_provider.dart`). Add the getter there
**before** referencing it in UI.

Exempt (not translatable): brand name `MEOW AGENT`, route names, model-id
hints (`gpt-4o-mini`), emoji, and other non-language tokens.

### 1.4 New features add their strings first

Adding a screen/dialog/snackbar = add every label to `AppStrings` first, with
both `id` and `en` variants, then reference them. A getter that returns the
same text for both languages is fine when the term is a proper noun, but it
must still go through `AppStrings`.

### 1.5 Runtime (agent-facing) language is separate

The agent's spoken language is handled by the runtime via `DetectedLanguage` +
`ToolVerbalizer` / `NarrativeNarrator` / `LanguageRegistry` — NOT by
`AppStrings`. Never wire `AppStrings` into runtime prompt logic, and never wire
`DetectedLanguage` into UI copy. They are two different layers (see §3).

---

## 2. Prompt Rules (STRICT)

### 2.1 All prompt text lives in `prompt_*` files

Every LLM-facing string lives under `lib/services/agent_runtime/`:

| File | Owns |
|------|------|
| `prompt_constants.dart` | Central accessor (`PromptConstants.*`) + version + caching |
| `prompt_system.dart` | System rules, introduction gate |
| `prompt_analyze.dart` | Analyzer phase (intent, tool_groups, selectors) |
| `prompt_reflect.dart` | Reflector phase (strategy, impacts, slots) |
| `prompt_plan.dart` | Planner phase (goal tree) |
| `prompt_execute.dart` | Tool selector + reviewer |
| `prompt_context.dart` | Chat, compactor, repair, pending action, memory, workflow API context |
| `prompt_policy.dart` | Reusable policy blocks (Ask / Ground / Minimal / Recover / Voice) |
| `prompt_workflow.dart` | Workflow auto-execute prompts |
| `prompt_templates.dart` | Assembles the above into final prompts |

**Banned:** inline prompt strings in `runtime_engine.dart`, `workflow_runner.dart`,
module code, or any feature file. If you are writing a sentence the LLM will
read, it belongs in a `prompt_*` file and is exposed through `PromptConstants`.

### 2.2 Prompts are English-only

All prompt scaffolding and examples are authored in English. The LLM responds
in the user's language naturally via the separately-injected `DetectedLanguage`.

- No per-language example sets.
- Never enumerate language-specific words ("semua/setiap" / "all/every") as a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ibnuard/meow-agent](https://github.com/Ibnuard/meow-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
