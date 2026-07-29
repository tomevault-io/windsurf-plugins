---
trigger: always_on
description: Copyright (c) 2024 Snowflake Inc.
---

<!--
 Copyright (c) 2024 Snowflake Inc.

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

 http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
 -->

# Code Conventions


## SQL safety

Never interpolate user-supplied values or config values directly into SQL strings.
Use the appropriate helper depending on whether the value is an **object identifier**
or a **string value**.

### Object identifiers (table names, stage names, schema names, …)

Use `FQN` and its `.sql_identifier` property, which wraps the identifier in
`IDENTIFIER('...')` so Snowflake resolves it correctly.

When the components come from separate variables, use the constructor directly —
never f-string them together before passing to `FQN`:

```python
# WRONG — f-string composed before any sanitisation
cursor.execute(f"CREATE TABLE {database}.{schema}.{table_name} ...")

# ALSO WRONG — f-string still composes before FQN sees the parts
from snowflake.cli.api.identifiers import FQN
fqn = FQN.from_string(f"{database}.{schema}.{table_name}")
cursor.execute(f"CREATE TABLE {fqn.sql_identifier} ...")

# CORRECT — pass each component individually
from snowflake.cli.api.identifiers import FQN
fqn = FQN(database=database, schema=schema, name=table_name)
cursor.execute(f"CREATE TABLE {fqn.sql_identifier} ...")
```

`FQN.from_string()` is safe when the full dotted name comes from a single
trusted source (e.g. already-validated CLI argument). Use the constructor
whenever the components arrive separately.

### String values (LIKE patterns, CALL arguments, stage paths, string parameters)

Use `to_string_literal()` from `snowflake.cli.api.project.util`, which produces
a properly escaped single-quoted string literal:

```python
# WRONG
cursor.execute(f"SHOW DATABASES LIKE {database}")

# CORRECT
from snowflake.cli.api.project.util import to_string_literal
cursor.execute(f"SHOW DATABASES LIKE {to_string_literal(database)}")
```

When doing SHOW … LIKE to look up a **specific object by name** (not a user
search pattern), use `identifier_to_show_like_pattern()` instead. It also
escapes `%` and `_` wildcard characters so that `'my_db'` does not accidentally
match `'myXdb'`:

```python
from snowflake.cli.api.project.util import identifier_to_show_like_pattern
cursor.execute(f"SHOW DATABASES LIKE {identifier_to_show_like_pattern(name)}")
```

### Bind parameters

For scalar values in queries and write statements (`SELECT`, `INSERT`, `UPDATE`,
`DELETE`), prefer bind parameters — they are the safest option and require no
escaping:

```python
cursor.execute("SELECT * FROM my_table WHERE id = %s", (user_id,))
```

Bind parameters are not supported for DDL or `SHOW` statements. Use
`to_string_literal` or `identifier_to_show_like_pattern` there instead.

## Handling sensitive values

When a command option accepts a secret (password, token, key, etc.), declare it
with `click_type=SecretTypeParser()` from `snowflake.cli.api.commands.flags`.
This wraps the raw string in a `SecretType` at parse time so the value is never
accidentally logged or printed (`__str__` returns `***`):

```python
from snowflake.cli.api.commands.flags import SecretTypeParser
from snowflake.cli.api.secret import SecretType  # annotation only — SecretTypeParser wraps the value at parse time

@app.command()
def my_command(
    password: Optional[SecretType] = typer.Option(
        None, click_type=SecretTypeParser(), hide_input=True,
    ),
):
    if password:
        do_something(password.value)            # access the real value explicitly
    log.debug("Called with password=%s", password)  # logs "***"
```

## File access

Use `SecurePath` from `snowflake.cli.api.secure_path` for **all** file reads and
writes. `SecurePath` provides two security guarantees:

1. Every file operation is logged at `INFO` level, creating an audit trail.
2. Files created by `SecurePath` get restrictive permissions (0600 on Unix).

```python
# WRONG
from pathlib import Path
content = Path("config.yml").read_text()

# CORRECT
from snowflake.cli.api.secure_path import SecurePath
content = SecurePath("config.yml").read_text(file_size_limit_mb=1)
```

`pathlib.Path` is still acceptable for path manipulation (joining segments,
checking suffixes, building paths).

## Terminal output safety

Never print values from Snowflake server responses or user config directly to the
terminal. They may contain ANSI escape sequences or terminal control characters.

```python
# WRONG
cc.step(f"Object name: {server_response['name']}")

# CORRECT
from snowflake.cli.api.sanitizers import sanitize_for_terminal
cc.step(f"Object name: {sanitize_for_terminal(server_response['name'])}")
```

## Error handling

The project is migrating from `ClickException` to typed `CliError` subclasses.
Existing `ClickException` usages are legacy — do not imitate them.

All new code must use `CliError` or one of its subclasses from
`snowflake.cli.api.exceptions`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snowflakedb/snowflake-cli](https://github.com/snowflakedb/snowflake-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
