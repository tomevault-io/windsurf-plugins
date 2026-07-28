---
trigger: always_on
description: YOU MUST FOLLOW EVERY RULE BELOW. These are not suggestions.
---


# LISA Code Review Instructions

YOU MUST FOLLOW EVERY RULE BELOW. These are not suggestions.

## CRITICAL REQUIREMENT — Test Suggestion Block

EVERY review you produce MUST end with a test validation block in EXACTLY this format. Do not skip it. Do not rephrase it. Do not place it inside a summary section. Place it as the LAST thing in your review output.

**Key Test Cases:**
<method_names separated by | — use exact method names, no class prefixes>

**Impacted LISA Features:**
<feature class names — e.g. NetworkInterface, StartStop, Gpu, Nvme, Resize, Hibernate>

**Tested Azure Marketplace Images:**
<exact image strings from Section 7 below, one per line with - prefix>

Example:

**Key Test Cases:**
verify_reboot_in_platform|verify_stop_start_in_platform|smoke_test

**Impacted LISA Features:**
NetworkInterface, StartStop

**Tested Azure Marketplace Images:**
- canonical 0001-com-ubuntu-server-jammy 22_04-lts-gen2 latest
- redhat rhel 9_5 latest

Rules:
- Test method names: exact names only, no class prefix, separated by `|`.
- Feature classes: exact class names from `lisa/features/`.
- Images: use exact strings from Section 7. Choose by affected distro, Gen1/Gen2 if generation-sensitive, x64/ARM64 if architecture-sensitive.
- Only list integration tests. Unit tests run automatically.

---

## Severity Levels — Use These Exact Labels

| Level | When | Action |
|-------|------|--------|
| Critical | Security issue, data loss, breaks existing tests | Request Changes |
| Major | Coverage reduction without justification, missing cleanup, wrong logic | Request Changes |
| Minor | Missing comments on magic numbers, inconsistent naming | Comment |
| Nit | Style preference, formatting | Comment |

Do NOT use "High", "Medium", "Low". Use "Critical", "Major", "Minor", "Nit" only.

Request Changes if any Critical or Major issue exists. Approve only if none remain.

---

## PR Hygiene

- Empty PR description → comment: "Please add a description explaining what this PR does and why."
- No linked issue for a bug fix → comment: "Consider linking the related issue for traceability."
- PR touches >10 unrelated files → suggest splitting.

---

## Code Quality

- Magic numbers controlling test behavior (loop counts, timeouts, thresholds) MUST have inline comments. Flag any uncommented magic number.
- If the PR reduces iteration counts, removes tests, lowers retries, or narrows scope → flag: "This change reduces test coverage. Please provide justification."
- Catch specific exceptions, not bare `except:` or `except Exception`.
- Exception messages must include **what happened** AND **how to resolve/investigate**. Bad: `"VM size not found"`. Good: `"VM size [X] not found in location [Y]. Verify the size is available in this region."`.
- Use `assert_that()` from assertpy, not bare `assert`.
- Tests creating Azure resources must clean them up.
- Flag `keep_environment: yes` without justification.
- Use `node.log` or `log`, not `print()`.
- **No `time.sleep()`**. Use bounded waits: `check_till_timeout()`, `retry_without_exceptions()`, or the `retry` decorator. Bare `sleep()` causes flaky or slow tests.
- **Type hints mandatory** on all test method parameters and return types. At minimum `node: Node` and `-> None`. Flag missing type hints.
- **Prefer f-strings** over `%` formatting or `.format()`.
- **Path handling**: Use `node.get_pure_path()` for cross-OS path compatibility. Flag hardcoded `/` or `\\` path separators in node commands.

---

## Assertions

- Put the **actual value** in `assert_that(actual)`, not the expected value.
- Add `.described_as("business context")` to every assertion that isn't self-explanatory. The description should explain **why** this check matters, not repeat the code.
- Prefer **native matchers** over manual manipulation: `assert_that(items).is_length(6)` not `assert_that(len(items)).is_equal_to(6)`.
- Use **collection assertions**: `.contains()`, `.is_subset_of()`, `.does_not_contain()` instead of manual loops.
- Do not repeat already-checked conditions. One precise assertion is better than redundant chains.

---

## LISA Conventions

### Test Structure
- **One test class per file.** Class name PascalCase, describes the feature (not a scenario). Inherits `TestSuite`.
- **Method naming**: Prefix with `verify_` or `test_`. Name describes the scenario being validated.
- **File location**: `lisa/microsoft/testsuites/<feature_area>/<test_name>.py`. Filename in snake_case.
- Tests follow **AAA pattern**: Arrange → Act → Assert. Keep sections clearly separated.

### Metadata
- Every test class has `@TestSuiteMetadata` with `area`, `category`, `description`. Flag if any field is missing.
- Every test method has `@TestCaseMetadata` with `description`, `priority`, `requirement`. Flag if any field is missing.
- `requirement` should use `simple_requirement(supported_os=..., unsupported_os=..., supported_features=..., supported_platform_type=...)`. Flag custom selection logic that duplicates what `simple_requirement` already provides.

### Tools, Features, and Extensions
- Use LISA tools from `lisa/tools/` instead of raw `node.execute()` when a tool exists.
- Use LISA features from `lisa/features/` for platform capabilities.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/lisa](https://github.com/microsoft/lisa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
