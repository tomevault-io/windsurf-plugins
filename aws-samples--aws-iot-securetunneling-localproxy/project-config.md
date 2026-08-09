---
trigger: always_on
description: Guidance for agents (and humans) making commits to
---

# AGENTS.md

Guidance for agents (and humans) making commits to
**aws-iot-securetunneling-localproxy**. This repo is a C++14 / CMake project
(not a Brazil package); it uses Boost.Asio/Beast, protobuf and OpenSSL, and a
Nix flake provides the formatter. See `CONTRIBUTING.md` for the project's
overall contribution policy (issues, PRs, CLA).

---

## TL;DR commit checklist

Before every commit:

1. Make one focused, logical change.
2. Update the unit tests and UATs the change affects (see below).
3. `nix fmt` — format the tree (must end clean / idempotent).
4. Run the spelling check (see Spelling); add legitimate technical terms to
   `misc/dictionary.txt` if flagged.
5. Build and run the unit tests (see below); ensure they pass.
6. Stage only the files for this change; write a clear message (see below).
7. Never commit to `main`; never force-push a shared branch without
   coordinating.

---

## Branching

- Work on a feature branch named `dev/<short-kebab-name>` (e.g.
  `dev/security-audit-findings`).
- Per `CONTRIBUTING.md`, work against the latest `main` and keep a change
  focused on one thing — don't reformat unrelated code.
- History may be reworded/rebased **only while the branch has not been pushed**.
  Once shared, prefer new commits; rewriting then requires a coordinated
  force-push.

## One change per commit

- Each commit should be a single, self-contained change (one fix / one feature).
  Don't mix unrelated changes.
- Keep the **tests** and **docs** for a change in the same commit as the change
  (e.g. URL-parsing tests live with the URL-parsing fix).
- With each feature or fix, update the **unit tests** (`test/`, Catch2) and the
  **UATs** (`uat/`) as needed so behavior stays covered: add cases for new
  behavior and adjust existing ones instead of deleting them.
- Keep **formatting** in the same commit as the code it applies to. Normally
  this is automatic: just run `nix fmt` _before_ committing. (Redistributing
  formatting across already-made commits is only needed if it was applied after
  the fact — not part of the normal flow.)

## Commit message style

Match the existing history: a short, imperative, capitalized subject with **no
trailing period**, under ~70 characters.

```
<Imperative subject, e.g. "Fix double callback and buffer leak in WebProxyAdapter">

<One or two lines: the problem, and why it matters.>

- Short bullet per distinct point: the reason (why) and the idea (how).
```

- Subject says _what_ changes; body says _why_ and _the idea_. Keep it terse.
- **Be concise.** Two or three bullets is usually the whole body; if you are
  writing a fourth, check whether it restates the subject, narrates the process,
  or describes something the diff already shows. A one-line body, or a single
  `Fix:` line, is fine for a simple change.
- Don't add `Tests:`/`Docs:` trailers, or any other line that just names the
  kind of change — the subject and the diff already say that. Mention tests or
  docs only when the reader needs a fact that is not evident from the diff.
- Reference symbols/files in `code font` where it aids clarity.
- Describe the final state, not how you got there. No "first tried X", no
  "renamed again", no apologies for earlier commits on the branch. While the
  branch is unpushed, squash follow-up fixes into the commit they belong to and
  write the message as if it had been one commit all along.

Examples from history:

- `Add AF_UNIX support for destination (-d) mode`
- `Fix editor-config errors`
- `Split the README into topic-based docs under docs/`

## Build & test

The project builds with CMake + make (see `docs/BUILD.md` for the full guide).
From the repo root:

```bash
mkdir -p build && cd build
cmake ../ -DBUILD_TESTS=ON        # add -DLINK_STATIC_OPENSSL=OFF for dynamic OpenSSL
make -j"$(nproc)"
```

- The proxy binary is produced at `build/bin/localproxy`.
- With `-DBUILD_TESTS=ON`, the Catch2 test binary is `build/bin/localproxytest`;
  run it to execute the unit suite. Ensure all tests pass before committing.
  `ctest --test-dir build` runs the same binary; CI invokes the path directly.
- Build directories matching `*build*/` are git-ignored.
- The end-to-end UATs live in `uat/` (see `uat/README.md`); run the relevant
  scripts when a change affects tunnel/proxy behavior, and keep them current
  with the feature.

Boost, Protobuf and Catch2 are resolved from the system when present and
otherwise built from the versions pinned in `fc_deps.json` — that file is the
**only** place to bump a dependency version. When you do bump one, also refresh
`.github/docker-images/oss-compliance/build-from-source-packages/build-from-source-package-licenses.txt`
and `THIRD_PARTY_LICENSES`, or the license manifests silently go stale. OpenSSL
and zlib always come from the platform and are deliberately absent from the
manifest.

Relevant CMake options (top of `CMakeLists.txt`; full table in `docs/BUILD.md`):

- `BUILD_TESTS` (default OFF) — build the Catch2 unit tests under `test/`.
- `LINK_STATIC_OPENSSL` (default ON) — statically link OpenSSL.
- `LOCALPROXY_RELEASE` (default OFF) — omit the git hash; the `version` file at
  the repo root is the single source of truth.
- `DISABLE_SSL_HOST_VERIFY_OPT` (default OFF) — production builds may drop the
  `--no-ssl-host-verify` option.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/aws-iot-securetunneling-localproxy](https://github.com/aws-samples/aws-iot-securetunneling-localproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
