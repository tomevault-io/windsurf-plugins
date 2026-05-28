---
trigger: always_on
description: **Agent Acknowledgment**: At the start of each response, identify yourself using your actual model name and confirm you understand and will follow these instructions.
---

# GitHub Copilot Instructions for Image-Description-Toolkit

**Agent Acknowledgment**: At the start of each response, identify yourself using your actual model name and confirm you understand and will follow these instructions.

# ⚠️ CRITICAL REMINDERS

**NEVER claim code is fixed without actually RUNNING it**
**PyInstaller frozen executables are NOT the same as Python scripts**
**`from scripts.X` ALWAYS fails in frozen mode - use module-level imports**
**Production code that's been working for months should be changed with EXTREME caution**
**INCOMPLETE REFACTORS caused 23% of commits to be fixes - ALWAYS check ALL callers**
**Changing function signatures REQUIRES updating ALL callers - search first, change later**
**wx event handlers SILENTLY SWALLOW exceptions - "does nothing" means exception on line 1**
**REGRESSION? Run `git log` to find breaking commit FIRST. Dev mode SECOND. Frozen exe LAST.**

## 🚨 MANDATORY READING

**Before making ANY code changes**, review:
- **[Pre-Commit Verification Checklist](docs/worktracking/PRE_COMMIT_VERIFICATION_CHECKLIST.md)** - MANDATORY checklist to complete BEFORE any code changes
- **[AI Comprehensive Review Protocol](docs/worktracking/AI_COMPREHENSIVE_REVIEW_PROTOCOL.md)** - Required checklists for all changes
- **[Migration Audit](docs/worktracking/2026-01-20-MIGRATION-AUDIT.md)** - Known issues and patterns to avoid

**Failure to follow these protocols has caused critical production failures.**

## ⚠️ PRE-COMMIT VERIFICATION PROTOCOL

**BEFORE making ANY code change:**
1. ✅ Complete Phase 1: Read actual code, verify names exist
2. ✅ Complete Phase 2: Search for impacts, check all callers
3. ✅ Complete Phase 3: Make minimal changes only
4. ✅ Complete Phase 4: BUILD and TEST the executable
5. ✅ Complete Phase 5: Document actual results (not "should work")

**See [PRE_COMMIT_VERIFICATION_CHECKLIST.md](docs/worktracking/PRE_COMMIT_VERIFICATION_CHECKLIST.md) for full details.**

**DO NOT claim something is "fixed" or "done" without completing ALL phases.**

Follow these guidelines for all coding work on this project.

## Code Quality Standards

1. **Professional Quality:** Code should be professional quality and tested before calling something complete and ready for submission.

2. **Planning and Tracking:** Planning and tracking documents should go in the `docs/worktracking/` directory and kept current as work is completed.

3. **Session Summaries:** Create and maintain a session summary document in `docs/worktracking/` with format `YYYY-MM-DD-session-summary.md`. Update it throughout the session with:
   - Changes made (files modified, features added)
   - Technical decisions and rationale
   - Testing results
   - User-friendly summaries of what was accomplished
   - Keep it updated until explicitly told to stop

4. **Accessibility:** All coding should be WCAG 2.2 AA conformant. This should be a given for this project, so being accessible doesn't need to be highlighted in documentation unless it is something especially unique.

5. **Dual Support:** Work needs to keep in mind that we want to keep support for both the script-based approach for image descriptions and the GUI ImageDescriber app.

6. **No Shortcuts:** Avoid the typical behavior of AI of taking shortcuts, not scanning files completely, and more. This has already resulted in duplicate code at times and extensive time debugging. The project has become too large for constant testing.

7. **Repository Hygiene:** Keep the repository clean and organized. Remove unused files, fix broken links, and ensure that documentation is up to date.

8. **Finished includes tested now and forever:** Ensure that all code is thoroughly tested before considering it finished. This includes unit tests, integration tests, and any other relevant testing methodologies completed along with the change.

9. **Test Before Claiming Complete:** Before stating that code is fixed or a feature is implemented:
   - Actually BUILD the executable if changes affect compiled code
   - RUN the code with realistic test scenarios (not just theoretical analysis)
   - VERIFY the fix works end-to-end, not just that syntax is correct
   - Do NOT ask the user to test - YOU test first, then report results
   - If you cannot test (missing environment/data), explicitly state what you CAN'T verify
   
   **Example from config system debugging:** Agent made 7 fixes across multiple files but kept saying "it should work now" without testing. Each fix broke something new because related code wasn't checked. The correct approach: After each fix, rebuild idt.exe, run with test data, verify the actual error is gone, check for NEW errors, repeat until genuinely working.

10. **Comprehensive Impact Analysis:** When making changes:
    - Search for ALL usages of the function/variable/pattern being changed
    - Check for duplicate implementations that need the same fix
    - Look for related code that depends on the changed behavior
    - Verify argument parsers don't have conflicting flags (e.g., two `-c` arguments)
    - Check if frozen executable vs dev mode have different code paths

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kellylford/Image-Description-Toolkit](https://github.com/kellylford/Image-Description-Toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
