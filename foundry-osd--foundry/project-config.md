---
trigger: always_on
description: You must speak and write code exclusively in English.
---

You must speak and write code exclusively in English.

General behavior:
- Be concise, direct, and pragmatic
- Prefer implementation over long explanations
- Do not explain obvious things
- Avoid overengineering
- Follow the existing project structure and conventions

Code rules:
- Write production-ready code
- Use clear and meaningful names
- Keep methods small and focused
- Favor readability and maintainability
- Do not introduce unnecessary dependencies
- Do not rewrite unrelated code
- Minimize the scope of changes

Cleanup rules:
- After an implementation, check whether replaced code, unused files, obsolete helpers, dead configuration, or outdated documentation became unnecessary
- Remove obsolete code only when it is clearly made redundant by the current change and is within the task scope
- Do not remove legacy or compatibility code unless the task explicitly replaces it or the user asks for that cleanup

Unit testing rules:
- Add unit tests only when they provide clear business value
- Prioritize business logic, validation, selection, transformation, and fallback rules
- Do not add UI tests unless explicitly requested
- Do not test WPF views, WinUI views, code-behind, bindings, or framework behavior by default
- Avoid superfluous tests and duplicated coverage
- Keep tests small, deterministic, and easy to read
- Use clear test names that describe the behavior being verified
- Cover critical paths and edge cases, not trivial getters/setters
- Reuse existing test patterns and project structure when available
- When adding new functionality, add or update the smallest relevant set of tests
- Keep test projects aligned with the main project naming and solution structure

Format and verification rules:
- Before opening or updating a pull request, run `scripts\Test-FoundryFormat.ps1` from the repository root
- If the format check fails, run `scripts\Format-Foundry.ps1`, review the resulting diff, and rerun `scripts\Test-FoundryFormat.ps1`
- Treat `scripts\Test-FoundryFormat.ps1` as the source of truth for formatting checks
- Include the format check result in pull request testing notes

Git rules:
- Follow Conventional Commits for all commit messages
- Prefer Conventional Commit scopes when the change has a clear area, for example `feat(winpe): ...`, `fix(packaging): ...`, or `docs(readme): ...`
- Write commit messages in English
- Keep commits atomic and focused

Worktree / branch / PR rules:
- Use a dedicated git worktree for implementation work when the task changes code
- Create worktrees outside the main repository folder
- Sync the base branch before creating a worktree
- Create a focused branch for each implementation task
- Keep branch names short, descriptive, and aligned with the task scope
- Do not mix unrelated changes in the same branch
- Push the branch and open a pull request when implementation and verification are complete
- Check CI status before merging a pull request
- Treat x64 and ARM64 CI checks as blocking for Foundry changes
- Ignore submit-nuget failures unless the user explicitly asks to investigate that check
- Prefer squash merge when merging Foundry pull requests
- Delete merged feature branches and clean up worktrees after PR merge
- Do not remove a worktree before merge unless the user explicitly approves it

Pull request rules:
- Write pull request titles in English using Conventional Commits
- Prefer scoped pull request titles when the change has a clear area, for example `feat(winpe): ...`, `fix(packaging): ...`, or `docs(readme): ...`
- Write concise pull request descriptions in English
- Include: summary, reason, main changes, and testing notes

.NET / WPF / WinUI 3 rules:
- Handle this Visual Studio solution as a mixed WPF and WinUI 3 solution
- Foundry is a WinUI 3 desktop application
- Foundry.Connect and Foundry.Deploy are WPF runtime applications
- Keep WPF-specific logic and WinUI 3-specific logic separated when their UI frameworks differ
- Respect DevWinUI shell and navigation patterns when present
- Follow MVVM when applicable
- Keep business logic out of code-behind whenever possible
- Keep code-behind limited to UI-specific event wiring or framework glue
- Use XAML cleanly and keep UI structure readable
- Prefer bindings, commands, and view models over direct UI manipulation
- Reuse existing services, models, and patterns before creating new ones
- Respect nullable reference types and existing analyzer warnings if enabled

Configuration schema rules:
- Treat Foundry authoring configuration, Foundry.Deploy runtime configuration, and Foundry.Connect runtime configuration as separate schema contracts
- Keep schema versions monotonic within each contract
- Bump a schema version only when the persisted or generated configuration contract changes in a way that affects runtime behavior or compatibility
- Bump Foundry authoring schema when the user-facing persisted Foundry configuration shape, defaults, migration behavior, or semantic meaning changes
- Bump Foundry.Deploy runtime schema when Deploy consumes new generated configuration, requires new boot media assets, changes the meaning of an existing Deploy field, or needs older boot media to show a compatibility warning

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [foundry-osd/foundry](https://github.com/foundry-osd/foundry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
