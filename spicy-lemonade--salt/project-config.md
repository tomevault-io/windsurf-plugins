---
trigger: always_on
description: - **Source data is always the truth.** Salt must never modify, repair or convert the files it backs up; it encrypts them and restores them exactly as they were given (contents, dates and permissions).
---

# CLAUDE.md

## Important Rules

- **Source data is always the truth.** Salt must never modify, repair or convert the files it backs up; it encrypts them and restores them exactly as they were given (contents, dates and permissions).
- **Build for any agent memory, not one tool.** Hermes, Mnemosyne, Honcho, Hindsight and the others are examples. Salt's code, messages and defaults must work for any files, SQLite or Postgres database, whatever tool made them, with no tool-specific paths, names or special cases.
- *Never* run git commands without asking for user permission, even if 'auto-accept' is selected during a Claude Command session.
- *Never* run files which use an LLM API without asking for user permission,  even if 'auto-accept' is selected during a Claude Command session.
- *Never* attempt to re-engineer the code or alter data without asking for user permission.
- *Never* make assumptions. Ask for more information and wait for the user response.
- Do *not* use numerical prefixes when writing comments.
- Do *not* use newline characters in print statements.
- Use double quotation marks instead of single quotation marks when possible.
- Use type hinting.
- Favor modular, resuable code.
- Favor vectorised code.
- Use concurrency when processing data with an LLM API.
- Read existing files before writing any output.
- Do not re-read files unless they have been changed.
- Use gitmoji when commiting, by running `gitmoji -c` and selecting an appropriate emoji.

## Project Overview

Salt is an open-source Go CLI that encrypts AI-agent memory backups before they are pushed to Git. Examples are Hermes with Mnemosyne SQLite databases and Markdown files such as USER.md, MEMORY.md, SOUL.md and SKILL.md. It also covers Postgres databases such as Honcho and Hindsight (Postgres + pgvector), and will later cover OpenViking. An example usage is a nightly backup script which saves a snapshot of the agent files, but runs `salt seal` to encrypt them before they reach the git repo. A pre-commit hook (`salt check`) then refuses any commit containing a file that is not encrypted. Salt is distributed through a Homebrew tap. The full design is in `docs/design.md`.

Salt is a general-purpose open-source tool for public release. Write code, defaults, messages and docs for any user and any setup.

**Key Technologies:**
- Go 1.26+
- age (`filippo.io/age`) for encryption: X25519 keys, and scrypt for passphrase-wrapped keys
- zstd (`github.com/klauspost/compress/zstd`) for compression before encryption
- BIP39 12-word recovery phrases; the age key is derived from the phrase with HKDF-SHA256
- OS keychain via `github.com/zalando/go-keyring`, with a 0600 file fallback (`SALT_KEYSTORE=file`)

**Commands:** `init`, `seal [--sqlite DB] [--postgres CONN] [--postgres-env VAR] [--name NAME]`, `prune`, `check`, `restore [--allow-unsigned]`, `verify [--allow-unsigned]`, `doctor`, `trust`, `recovery test|show`, `hook install`, `version`.

**Package layout:**
- `cmd/salt`: CLI entry point and flag parsing
- `internal/app`: commands; talks to the person only through `UI` and to git only through `GitOps`
- `internal/seal`: streaming seal, restore and verify; encrypted index; change-detection cache
- `internal/keys`: recovery phrase, key derivation, passphrase wrapping, key stores
- `internal/repo`: backup repo layout, format file, recipients, public-file allowlist
- `internal/check`: pre-commit plaintext detection
- `internal/hook`: pre-commit hook script and installation
- `internal/gitx`: the only way salt runs git (hooks always disabled)
- `internal/source`: safe copies of live databases (runs `sqlite3` and `pg_dump`); each kind is a `Database` listed in `Kinds`; with `gitx`, the only packages that start programs
- `internal/guard`: refuses nested salt processes; sets a soft memory limit
- `internal/prune`: keeps only the backups from the last N days with a change (counted for the whole repo, not per file) by rewriting the branch's history
- `internal/trust`: this machine's approved copy of each repo's keys and settings; seal refuses if the repo differs
- `internal/rules`: source-scan test enforcing the process-safety rules

### Data Architecture

**Backup repository layout** (a git repo owned by salt):
- `.salt/format.json`: public; layout version, `encrypt_paths`, recovery method
- `.salt/recipients.txt`: public; the age public keys every file is encrypted to
- `.salt/key.age`: passphrase-wrapped private key (passphrase recovery only)
- `index.age`: encrypted JSON index holding real paths, SHA-256 of the plaintext, sizes, modes, last-modified times and symlinks, signed with the signing key
- `objects/xx/<random>.age`: file contents when paths are encrypted (the default), and parts after the first of a file over 99 MiB
- `files/<path>.age`: file contents with `--plain-paths`
- `README.md`, `LICENSE`, `.gitignore`, `.gitattributes`: the only other files allowed unencrypted


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spicy-lemonade/salt](https://github.com/spicy-lemonade/salt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
