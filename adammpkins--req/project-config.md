---
trigger: always_on
description: This documents the language. It is a DSL that reads like a sentence and runs in a shell.
---

# .cursor/rules/grammar.mdc

# req grammar - v0.1 specification

This documents the language. It is a DSL that reads like a sentence and runs in a shell.

## Command shape

    req <verb> <url> [clauses...]

Clauses are key=value pairs where keys are semantic words, not config words.

Order of clauses is free.

Unknown clause keys are errors.

## Verbs

- read - GET, print to stdout
- save - GET, write to file via to=
- send - default GET, POST if with= present
- upload - POST when attach= or with= present, else error
- watch - GET with SSE or polling
- inspect - HEAD only
- authenticate - login and store session state
- session show, session clear, session use - session management

## Clauses

| Clause     | Meaning                                     | Repeatable | Example                                                                 |
|-----------|---------------------------------------------|------------|-------------------------------------------------------------------------|
| using=    | HTTP method override                         | no         | using=PUT                                                               |
| include=  | Add headers, params, cookies                 | yes        | include='header: Authorization: Bearer token; param: q=search query'     |
| with=     | Request body                                 | no         | with=@user.json or with='{"name":"Adam"}'                               |
| expect=   | Assertions on response                       | no         | expect=status:200, header:Content-Type=application/json, contains:"ok"  |
| as=       | Output format for stdout                     | no         | as=json                                                                 |
| to=       | Destination path                             | no         | to=out.json                                                             |
| retry=    | Retry attempts for transient errors          | no         | retry=3                                                                 |
| under=    | Timeout or size limit                        | no         | under=30s or under=10MB                                                 |
| via=      | Proxy URL                                    | no         | via=http://proxy:8080                                                   |
| attach=   | Multipart parts for upload or send           | yes        | attach='part: name=avatar, file=@me.png; part: name=meta, value=xyz'    |
| follow=   | Redirect policy for write verbs              | no         | follow=smart                                                            |
| insecure= | Disable TLS verification for this request    | no         | insecure=true                                                           |

### include= value grammar

- One include clause may contain multiple items separated by semicolons that are not inside quotes.
- Allowed item tags:
  - header: Name: Value
  - param: key=value
  - cookie: key=value
  - basic: username:password

Merging
- Headers: keep all values for multi valued headers, else last wins.
- Params: repeated keys become repeated pairs in insertion order.
- Cookies: last value wins per cookie name.
- Basic Auth: sets Authorization header, overrides any existing Authorization header.

Quoting
- If an item payload contains a semicolon, quote the value.
- Backslash escapes allowed inside quoted values for the quote char and backslash.

Errors
- Unknown tag before the first colon.
- Header without Name colon Value.
- Param or cookie missing equals.
- Basic item missing colon (must be username:password format).
- Unquoted semicolon inside an item payload.

Examples

    include='header: Accept: application/json, application/problem+json; q=0.9'
    include='param: q=search query; param: tag=ai; param: tag=ml'
    include='cookie: session=abc; cookie: prefs="a=1; b=2"'
    include='basic: user:pass; header: Accept: application/json'

### attach= value grammar

- One attach may contain multiple items separated by semicolons that are not inside quotes.
- Allowed item tags:
  - part: name=..., file=@path  or  part: name=..., value=...
  - Optional filename=...
  - Optional type=media/type
  - boundary: TOKEN optional

Validation
- name is required.
- exactly one of file or value is required.
- for file parts, @path must exist at execution time.

Header behavior
- Any attach= forces multipart Content-Type with a generated boundary.
- If user included a manual Content-Type, override it and print a one line note.

Examples

    attach='part: name=avatar, file=@./me.png, filename=me.png, type=image/png'
    attach='part: name=meta, value={"name":"adam"}'
    attach='part: name=file, file=@./a.png; part: name=meta, value=xyz'

### with= body modes

- Inline text or JSON.
- @file to read from file.
- @- to read from stdin.

Content type
- If Content-Type is not set and inline begins with "{" or "[", infer application/json and note on stderr.
- An explicit Content-Type header always overrides inference.

### expect= assertions

Single clause with comma separated checks. All must pass.

Supported checks
- status:200
- header:Content-Type=application/json
- contains:"text"
- jsonpath:"$.items[0].id"
- matches:"^OK\\b"

Exit codes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adammpkins) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
