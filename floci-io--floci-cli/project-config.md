---
trigger: always_on
description: Guidance for AI coding agents working in the `floci-cli` repository.
---

# AGENTS.md

Guidance for AI coding agents working in the `floci-cli` repository.

This file defines repository-specific operating rules for autonomous or semi-autonomous coding agents. Follow these instructions unless a maintainer explicitly tells you otherwise.

---

## Project Overview

`floci-cli` is the official command-line interface for [Floci](https://floci.io) — the free, open-source local cloud emulator for AWS, GCP, and Azure. It manages emulator lifecycle (start/stop/status/logs), configuration profiles, diagnostics (`doctor`), and state snapshots.

- Stack: Java 25, Picocli 4.7.x, Jackson (JSON + YAML), JUnit 5
- Distribution: GraalVM native binaries (linux/amd64, linux/arm64, darwin/amd64, darwin/arm64, windows/amd64) + fat-JAR fallback
- No frameworks beyond Picocli — plain `java.net.http.HttpClient` and `docker` CLI subprocesses

---

## Build and Test Commands

```sh
mvn compile                          # compile only
mvn test                             # run all unit tests
mvn test -Dtest=WaitCommandParseTest # run a single test class
mvn test -Dtest=DockerVersionCheckTest#testVersionParsing  # run a single test method
mvn package                          # compile + test + produce target/floci.jar (fat JAR)
mvn package -DskipTests              # skip tests
mvn package -Pnative -DskipTests     # build native binary → target/floci  (requires GraalVM)
```

Running the fat JAR (requires Java 25+):

```sh
java -jar target/floci.jar <command>
```

---

## Architecture

### Entry point and command wiring

`FlociCli` is the Picocli root command. Every subcommand is registered in its `subcommands = {}` array. All commands implement `Callable<Integer>` and return an exit code.

Bare commands (`floci start`) route to the configured default product (`aws` unless changed via `floci config default-product aws|gcp|az|oci`). Explicit product groups are `floci aws` (same classes as the root tree), `floci gcp`, `floci az`, and `floci oci`.

### The unified product tree — CRITICAL

There is ONE implementation of every shared command, in `commands/`, parameterized by `ProductProfile` (`io.floci.cli.ProductProfile`) — the single source of per-product config:

| Profile | Default endpoint | Container | Env prefix | Control prefix |
|---------|------------------|-----------|------------|----------------|
| `AWS` (root tree) | `http://localhost:4566` | `floci` | `FLOCI_*` | `/_floci` |
| `GCP` | `http://localhost:4588` | `floci-gcp` | `FLOCI_GCP_*` | `/_floci-gcp` |
| `AZ` | `http://localhost:4577` | `floci-az` | `FLOCI_AZ_*` | `/_floci` (same as AWS — intentional) |
| `OCI` | `http://localhost:4599` | `floci-oci` | `FLOCI_OCI_*` | `/_floci-oci` |

The `commands/gcp/`, `commands/az/`, and `commands/oci/` leaf classes are **~8-line shims**: `GcpStartCommand extends StartCommand { super(ProductProfile.GCP); }` with their own `@Command` annotation carrying the per-product description. `ProductProfileTest` pins every profile field; `ProductTreesParsingTest` pins each tree's defaults.

Rules for the unified tree:

- **Every parameterized command pre-initializes its mixin in the constructor**: `this.global = new GlobalOptions(profile)`. Picocli uses a non-null `@Mixin` field instance as-is; a command that forgets gets AWS defaults in every tree (the per-tree parse tests catch this).
- **Per-product option defaults are constructor-set field values**, not annotation `defaultValue`s (annotation strings are compile-time constants). Use `(default: ${DEFAULT-VALUE})` in the option description to render them in help.
- **Never subclass a group command** (`GcpCommand`, `*ConfigCommand`, `*SnapshotCommand` groups): picocli registers the `subcommands` arrays of both super and subclass — instant duplicate-name crash. Groups stay standalone registration-only classes.
- Product-varying strings come from the profile: `displayName()` for banners, `commandPrefix()` for hint strings, `envVar("SERVICES")` for container env vars, `controlPrefix()` for `FlociHttpClient`, `serverRepo()` for issue-tracker links.
- Only `EnvCommand`/`GcpEnvCommand`/`AzEnvCommand`/`OciEnvCommand` are genuinely product-specific (not shims), plus the OCI-only `OciSetupCommand` (the OCI CLI/SDKs require a config file + signing key, so `floci oci setup` scaffolds them; no other tree needs an equivalent).

Commands call `global.printer()` at the start of `call()` — never store a `Printer` as a field.

### Two I/O boundaries

**Docker:** `DockerClient` wraps `docker` CLI subprocesses — no docker-java library. This is a deliberate native-image trade-off. `DockerClient` returns plain records (`ContainerInfo`, `ImageInfo`) and throws `DockerException` on non-zero exit codes.

**Floci server:** `FlociHttpClient` wraps `java.net.http.HttpClient` against the Floci REST API. All methods throw `FlociException`. The control-plane prefix is constructor-configurable: AWS and Azure use `/_floci`, GCP uses `/_floci-gcp`. Known live endpoints: `/_floci/health`, `/_floci/info`, `/_floci/init`. Snapshot endpoints (`/_floci/snapshots/*`) do not exist on the server yet.

### Output pipeline

Every command checks `printer.format()` to decide between text and structured output:

```java
if (printer.format() != OutputFormat.text) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [floci-io/floci-cli](https://github.com/floci-io/floci-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
