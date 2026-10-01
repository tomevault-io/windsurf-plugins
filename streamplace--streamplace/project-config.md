---
trigger: always_on
description: Streamplace is a Go application. The web and mobile frontend code lives under
---

# Streamplace

Streamplace is a Go application. The web and mobile frontend code lives under
`js/`, the media and API packages under `pkg/`, and the command entry points
under `cmd/`. The repo also contains generated lexicon bindings and end-to-end
test infrastructure.

This document is meant to be easy to parse for everyone, but agents should note that they must follow all guidelines, especially the ones marked as such.

Agents: you will want to keep your changes focused on the task. Read the relevant code before you edit it.
Use the repository's existing abstractions and scripts instead of recreating
their behaviour by hand.

## Repository layout

Useful areas of the repository:

- `pkg/`: Go packages, including API, media, RPC, and application logic.
- `cmd/`: Go command entry points.
- `js/app/`: Expo application.
- `js/web/`: web frontend.
- `js/components/`: shared frontend components.
- `js/e2e-web/`: Playwright end-to-end tests.
- `lexicons/`: source lexicons used to generate Go, JS, documentation, and API
  artifacts.
- `hack/`: development, provisioning, and test scripts.
- `.maestro/`: mobile end-to-end flows.

When you work in one of these areas, check its local documentation and
configuration first.

## Development environment

For agents: Inspect the current machine before you make decisions that depend on the
environment.

Use the repository's Make targets and scripts. Do not reproduce their underlying
commands by hand.

For the containerized build, scratch-node, and browser-test workflow, use the
[streamplace-docker skill](.claude/skills/streamplace-docker/SKILL.md). It covers
isolated sibling checkouts and does not require host-native build tools.

## Building

A fresh checkout needs the native build artifacts and the generated frontend
bundles before ordinary Go builds will work.

To set up a fresh checkout, run:

```sh
make dev-setup
```

This sets up Meson and the C and Rust dependencies, and runs an initial frontend
build.

For normal development, run:

```sh
make dev
```

After you change frontend code, rebuild the frontend. Do not assume an existing
`dist` directory reflects the current source:

```sh
make app
```

To build the Go packages that most closely match the CI build, run:

```sh
go build ./pkg/... ./cmd/...
```

For the code you change, run targeted checks where practical:

```sh
go test -count=1 ./pkg/<package>/...
```

`go vet ./pkg/<package>/...` is a fast sanity check, but it is much weaker than
the lint in the commit gate (golangci-lint with staticcheck). A change that
passes `go vet` can still fail CI tests! See [Checking your work](#checking-your-work).

Do not run expensive repository-wide suites when a targeted test gives the same
coverage.

## Frontend

The Go application embeds the built frontend artifacts. A frontend bundle can be
stale after you change source or switch branches.

When you change code under `js/`, make sure the bundle used by builds or
end-to-end tests was generated from the current source.

For UI and component styling, follow the
[streamplace-design skill](.claude/skills/streamplace-design/SKILL.md). Every
visual value comes from a theme token. Do not hardcode raw literals.
Useful checks:

```sh
cd js/app && npx tsc -p . --noEmit
pnpm run check
```

Use the workspace's existing package scripts. Do not add parallel tooling for
formatting, type checking, or dependency analysis.

## Lexicons and generated files

After you change files under `lexicons/`, regenerate the bindings:

```sh
make lexicons
```

Inspect the resulting diff and commit the generated outputs that the change
requires.

Lexicon generation can affect Go code, JS types, documentation, and API
artifacts. Do not assume a generated change is irrelevant just because it is
outside the directory you edited.

If you remove a lexicon, check whether its generated artifacts also need to be
removed.

Do not hand-edit generated output. Change the generator instead.

## Tests

Run the tests relevant to the code you changed.

For Go packages:

```sh
go test -count=1 ./pkg/<package>/...
```

When a package contains slow integration or media tests, use `-run` to run only
the tests you need.

The repository has a local web end-to-end harness and Playwright suite. The
normal entry point is:

```sh
hack/e2e-web-local.sh
```

The harness starts the services the browser tests need and creates temporary test
state. For agents: Use the harness. Do not reproduce its process topology by hand.

When you debug an e2e failure, a good first thing to do is to work out which of these caused it:

- application behaviour
- a stale frontend bundle
- a test harness failure
- an environment or networking failure

For agents: Do not call a failure pre-existing or environment-only until you have verified
it.

## Media and native dependencies

Parts of Streamplace depend on native media libraries and cgo.

If a Go build fails because native libraries or pkg-config metadata are missing,
you will want to inspect the repository's build environment and provisioning scripts.
For agents: Do not install arbitrary host dependencies or invent paths.

Use the repository's configured build environment to compile and test
media-related code.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [streamplace/streamplace](https://github.com/streamplace/streamplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
