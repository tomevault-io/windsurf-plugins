---
trigger: always_on
description: Apply these instructions to all work in this repository. They are ranked by
---

# Vernet Copilot Instructions

Apply these instructions to all work in this repository. They are ranked by
recurrence and practical impact across the indexed chat history and repository
guidance. Keep changes focused and preserve existing project conventions.

## Workflow and Scope

1. Start from the concrete error, failing test, named file, or requested behavior.
2. Inspect the smallest nearby code path that controls the behavior before editing.
3. State one falsifiable hypothesis about the failure before making a change.
4. Identify one cheap validation check that could disprove the hypothesis.
5. Keep each change narrowly scoped to the requested behavior.
6. Preserve existing public APIs unless the request requires a contract change.
7. Follow the repository's existing architecture and local conventions.
8. Do not refactor unrelated code while fixing a targeted problem.
9. Treat user edits and dirty-worktree changes as intentional; never discard them.
10. Do not commit or create branches unless explicitly requested.
11. Prefer fixing the root cause over masking the symptom.
12. Make the smallest reversible edit that can test the current hypothesis.
13. After the first substantive edit, run a focused validation immediately.
14. If validation fails, repair the same slice before expanding the investigation.
15. If validation disproves the hypothesis, move one hop toward the controlling code.
16. Use a neighboring test or call site to resolve ambiguous ownership.
17. Keep the user informed with concise progress updates during longer work.
18. Finish the task end to end when the environment permits it.
19. Report blockers plainly instead of implying that unrun checks passed.
20. Summarize changed files, verification, and remaining limitations at the end.

## Flutter Architecture

21. Keep primary application logic under `lib/`.
22. Keep UI in `lib/pages/` and reusable UI in `lib/widgets/`.
23. Keep network operations in services, never directly in widgets.
24. Route feature actions through the UI, provider/controller, and service layers.
25. Keep structured data in `lib/models/`.
26. Keep reusable helpers in `lib/utils/`.
27. Keep feature state local whenever possible.
28. Prefer Provider, scoped state, or simple `StatefulWidget` state already used nearby.
29. Avoid introducing global mutable state.
30. Keep services independent of UI dependencies.
31. Make services reusable across screens and platforms.
32. Return structured models from services instead of presentation strings.
33. Keep functions small and focused.
34. Prefer composition over duplication.
35. Add a model when a result has multiple related fields or lifecycle states.
36. Keep platform-specific changes isolated to the relevant platform directory.
37. Avoid editing Android, iOS, Linux, macOS, or Windows glue unless necessary.
38. Use plugins and platform channels through their established abstraction.
39. Read `ARCHITECTURE.md` before structural changes.
40. Keep new network tools organized as page, service, model, and tests.

## Errors and Dependencies

41. Read the complete stack trace and locate the first project-owned frame.
42. Verify dependency APIs against the installed package version before changing callers.
43. Use the dependency's expected types and serialization formats exactly.
44. Do not infer a package object's identifier from `toString()` when a typed property exists.
45. Handle plugin initialization explicitly on platforms where it may be unavailable.
46. Guard platform plugin calls in tests and unsupported environments.
47. Provide a documented, deterministic fallback when an external service is unavailable.
48. Use UTC or another explicit fallback for missing timezone data.
49. Do not hide a real initialization error behind a broad catch without logging or rationale.
50. Keep fallback behavior observable and testable.
51. Prefer lightweight dependencies already present in `pubspec.yaml`.
52. Avoid adding a package when the existing SDK or repository helper is sufficient.
53. Check cross-platform compatibility before adding or changing a dependency.
54. Keep dependency lockfile changes limited to dependency changes.
55. Never edit generated files by hand unless the project workflow requires it.
56. Regenerate generated artifacts with the repository's prescribed command.
57. Treat SQLite locking as a lifecycle and concurrency problem first.
58. Ensure database handles, statements, and transactions are closed or awaited correctly.
59. Avoid overlapping database initialization and test cleanup operations.
60. Reproduce database failures with the smallest relevant test or command.

## Testing

61. Add or update a focused regression test for every behavior fix when practical.
62. Run the narrowest relevant test before the full suite.
63. Use `flutter test path/to/test.dart` for a focused unit or widget test.
64. Use the repository's actual integration-test command rather than a guessed command.
65. Separate unit, widget, and integration-test claims in the final report.
66. Make tests deterministic and independent of live DNS, network timing, or host state.
67. Use an explicit testing flag or injected fake for external network behavior.
68. Do not make production behavior depend on test-only assumptions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [osociety/vernet](https://github.com/osociety/vernet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
