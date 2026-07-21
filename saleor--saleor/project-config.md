---
trigger: always_on
description: - Check last git tag to find what version we are branched from (e.g. 3.22 tag). Based on that, add description to new fields with ADDED_IN_{VERSION} clause
---

# Saleor

## Graphql

###  API versioning

- Check last git tag to find what version we are branched from (e.g. 3.22 tag). Based on that, add description to new fields with ADDED_IN_{VERSION} clause


### GraphQL permissions
- Use PermissionsField to describe field restrictions

# Architecture

- Do not use Django signals (e.g. `post_save`, `pre_delete`, `@receiver`). Call the relevant logic explicitly from the code that triggers it instead of wiring it through signal handlers. (The one exception is the `post_migrate` signal used to trigger data-migration tasks — see Data Migrations below.)

# Code style

- Use global import statements. Place all imports at the top of the file rather than inside functions, methods, or other local scopes.

# Testing

## Running in a git worktree

When Saleor is checked out in a git worktree, prefer running operations (tests, management
commands, etc.) inside that worktree's containers rather than on the host machine. The host's
services may belong to a different worktree, so running directly on the host can hit the wrong
database/cache or collide with another worktree's stack.

- Use the `.worktree-container/compose.sh` wrapper, which targets the current worktree's
  isolated stack. Run commands inside the `saleor` service, e.g.:
  ```sh
  .worktree-container/compose.sh up -d                       # start this worktree's stack
  .worktree-container/compose.sh exec saleor pytest --reuse-db saleor/path/to/test_file.py
  .worktree-container/compose.sh exec saleor python manage.py migrate
  ```
- Do not run these operations directly on the host machine when working in a worktree.

## Running tests

- Run tests using `pytest`
- Attach `--reuse-db` argument to speed up tests by reusing the test database
- Select tests to run by passing test file path as an argument
- Inside the dev container, dependencies are installed system-wide, so run `pytest` directly — no virtual environment to activate
- On the host (outside the container), enter the virtual environment before executing tests

## Writing tests

- Use given/when/then structure for clarity
- Use `pytest` fixtures for setup and teardown
- Declare test suites flat in file. Do not wrapp in classes
- Prefer using fixtures over mocking. Fixtures are usually within directory "tests/fixtures" and are functions decorated with`@pytest.fixture`
- When you create an object for testing and fixture for it doesn't exist, create new one. You can use factory to pass arguments to the fixture.
- When writing assertions, prefer assertion on actual returned value instead of checking if it's not none. For example: `assert email is "a@b.com` instead if `assert email is not None`
- When asserting GraphQL errors, assert error message too
- When asserting to Enum, import enum in a test file and use `assert error.code == MyEnum.SOME_ERROR.name` instead plain string comparison
- Avoid assertion to plain values, if you already have references to existing value. For example, when you create entity in database and assert if entity is returned in response, compare response fields with entity fields, instead plain values.
- When building GraphQL query variables, use enum values instead of plain strings. For example: `{"field": TransactionSortField.CREATED_AT.name}` instead of `{"field": "CREATED_AT"}`
- Do not write unnecessary prefix to test names. If file name is `test_my_mutation.py`, do not write `test_my_mutation_with_x_y` but `test_with_x_y`
- When testing graphQL cases with expected errors ALWAYS assert expected length of errors list
- When setting up test data, extract values into variables and reuse them in assertions. Do not repeat literal values between setup and assertion — use the variable instead.
- When comparing JSON payloads in tests, use `json.loads()` to compare dicts instead of comparing serialized strings with `json.dumps()`. String comparison breaks when key order changes.
- When a test expects exactly one row, use `Model.objects.get()` instead of `.first()` followed by an `is not None` assertion. `get()` both fetches the object and asserts that exactly one exists, so drop the separate not-None check.
- For presence/absence checks prefer `qs.exists()` over `qs.count() == 0` / `!= 0`. `exists()` is cheaper than a `COUNT`. Example: `assert checkout.lines.exists() is False`.
- Use `@pytest.mark.parametrize` for variations of the same scenario instead of copy/pasting near-identical test bodies. Label each case with a leading `_case` string parameter rather than the `ids=` argument — it keeps the description next to the data and is easier to read and edit:
  ```python
  @pytest.mark.parametrize(
      ("_case", "value"),
      [
          ("omitted", {}),
          ("empty_list", {"lines": []}),
      ],
  )
  def test_something(_case, value): ...
  ```
- When a mutation input is optional/nullable, test the explicit `null` value in addition to omitting the field — they are distinct inputs and can be handled differently.


# Webhooks and Events

## Dispatching webhook events


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saleor/saleor](https://github.com/saleor/saleor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
