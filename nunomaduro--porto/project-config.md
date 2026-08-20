---
trigger: always_on
description: `porto` is a dependency audit ledger for PHP. `porto` records the bytes of each installed package that the user trusts, shows the reviewable delta between the version that the user trusted and the version that the user installed, and exits non-zero when a package in `composer.lock` is ungranted or when its bytes changed. Two users read the output: a PHP developer who reviews the dependencies of a project, and the CI job of that project.
---

# The project

`porto` is a dependency audit ledger for PHP. `porto` records the bytes of each installed package that the user trusts, shows the reviewable delta between the version that the user trusted and the version that the user installed, and exits non-zero when a package in `composer.lock` is ungranted or when its bytes changed. Two users read the output: a PHP developer who reviews the dependencies of a project, and the CI job of that project.

Install the dependencies with `composer install`. Run the tests with `composer test`. Run the program with `./porto`.

`porto` is a [Laravel Zero](https://laravel-zero.com) application. `App\Commands\AuditCommand` is the default command in `config/commands.php`, thus `./porto` with no argument shows what is unaudited.

| Path | Function |
|---|---|
| `app/Commands` | the commands of the CLI: the input, the output and the flags |
| `app/Providers` | the service registration of Laravel Zero |
| `app/Identity` | the content identity of an installed tree |
| `app/Lock` | the readers of `composer.lock` and of `vendor/composer/installed.json` |
| `app/Registry` | the metadata of Packagist, the fetch of an archive and the extraction of an archive |
| `app/Delta` | the difference between two trees, the classifier of a change and the unified diff |
| `app/Ledger` | the read of `porto.json`, the write of `porto.json`, the entry and the audit decision |
| `app/Exceptions` | the exceptions of the audit engine |
| `app/Support` | the cache, the HTTP client, the JSON codec, the paths, the version constraints and the helpers of a command |
| `bootstrap` and `config` | the configuration of Laravel Zero |
| `tests` | the tests, which Pest runs |
| `porto.json` | the ledger of this project |

Audit this project with `./porto`, and record the result in `porto.json`.

---

## 1. The reader of this file

This file is for a coding agent. No user reads it. Write an instruction that an agent obeys during its work, then stop. Do not write an introduction, a conclusion, an argument for a decision that the project made, or a sentence that no agent can obey. Give a reason only when the reason changes the next decision of the agent.

Start each rule with a verb in the imperative. Put the condition before the instruction. Give the exact path, the exact command and the exact name that the agent must use. If a rule needs a test, give the command that does the test.

The public files are different. `README.md`, the website, the release notes and the `description` in each manifest are for a user. Section 3 controls them.

---

## 2. How to write this file

Write in [ASD-STE100](https://www.asd-ste100.org) Simplified Technical English: one instruction in one sentence, the imperative, the active voice, one meaning for one word, no contraction, and no synonym for variety. The standard holds the full rules. Do not copy them here. Four items keep their exact form: a quotation from a standard, an identifier in the code, a path or a command, and a name from a different supplier.

Write GitHub flavored Markdown. Put one `#` heading in a file. Write one paragraph on one line, thus a change stays small in `git diff`. Number the sections of a long file, thus a different file can point to "section 3". Put a path, a command and a name from the code in `code font`.

Write the intention of the project in this file. Write a decision about one directory in the `AGENTS.md` file of that directory. If this file and the `AGENTS.md` file of a directory disagree, obey this file and correct the other file.

Each task teaches you a fact that these files do not hold. Write the fact in the same change, or the session ends and you lose it. A correction from the user is a rule: write it in a file before you continue the work. Replace an old rule. Do not add a second rule near it. Delete a rule that the project does not obey. Do not keep a record of what the project stopped doing, in a file or in a directory. Git holds the history.

---

## 3. How to write the public text

These rules control `README.md`, the website, the release notes, the announcement and the `description` in each manifest. Sections 1 and 2 do not control these files.

The voice is calm and certain. Say what the project does, then stop. Do not say that the project is fast, simple, powerful or intelligent. Show the command and let the reader form that opinion. Cut each superlative. Cut each sentence that a competitor can also write about itself.

Write for one person and call that person "you". Put the result first and the mechanism second. Keep the sentences short, and let one sentence stand alone when it carries the idea. A demonstration is the strongest argument: one command in a terminal is worth one paragraph of adjectives.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nunomaduro/porto](https://github.com/nunomaduro/porto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
