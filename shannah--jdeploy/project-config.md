---
trigger: always_on
description: This guide captures the conventions and non-obvious patterns used across this codebase. It focuses on project-specific and uncommon conventions rather than generic Java best practices.
---

# Coding Style Guide (concise)

This guide captures the conventions and non-obvious patterns used across this codebase. It focuses on project-specific and uncommon conventions rather than generic Java best practices.

---

## Project layout & build systems
- Multi-module Java project:
  - `cli` is built with Maven (traditional `pom.xml`).
  - IntelliJ plugin uses Gradle Kotlin DSL (`build.gradle.kts`) and the `org.jetbrains.intellij` Gradle plugin.
- Gradle IntelliJ plugin configuration is declared in Kotlin DSL. Set JVM compatibility in `tasks.withType<JavaCompile>` and Kotlin target via `KotlinCompile.kotlinOptions.jvmTarget`.
- Use environment variables for secret values in Gradle tasks (e.g. `System.getenv("PUBLISH_TOKEN")`, `CERTIFICATE_CHAIN`, `PRIVATE_KEY`).

---

## Dependency Injection
- Use `javax.inject.Inject` + `javax.inject.Singleton` for constructor injection and singleton scopes.
- Central DI entrypoint: `DIContext.getInstance()` used to fetch singletons in places where constructor injection is not available (tests, builders).
- Constructors are used to declare dependencies; fields for injected dependencies are `private final`.

---

## Builders & Fluent Params pattern
- Builders implement an interface (e.g. `ProjectGeneratorRequest.Params`) and return that interface from setters for fluent chaining.
- Public getters on builders lazily derive values from multiple inputs (e.g. `magicArg`, `projectName`, `githubRepository`) rather than requiring callers to populate every field.
- Builders may embed CLI metadata as annotations on fields (see Command Line Parser section).

Example pattern:
- setX(...) returns `ProjectGeneratorRequest.Params` so callers can chain.
- getX() returns either explicitly-set value or an inferred default computed from other fields.

---

## Command-line metadata on fields
- Use a custom `CommandLineParser` annotation set on builder fields:
  - `@CommandLineParser.Alias("x")` for short flags
  - `@CommandLineParser.Help("...")` to document flags
  - `@CommandLineParser.PositionalArg(n)` for positional args
- These annotations are placed on private fields rather than on methods.

---

## Template + replacement conventions
- Template placeholders use Mustache-like double-brace tokens: `{{ foo }}` (e.g. `{{ packagePath }}`, `{{ mainClass }}`).
- Replacement occurs for many text file types (explicit list in code): .java, .properties, .xml, .gradle, .json, .yml, .yaml, .md, .adoc.
- File names are also processed for placeholder replacements (not only file contents).
- Placeholder substitution is performed by sequential String#replace calls (simple token replacement, not a template engine).
- `packagePath` placeholder is expected to be a `/`-separated path (derived from `packageName.replace(".", "/")`).

---

## File & directory handling
- Use Apache Commons IO `FileUtils` for copying, moving, reading/writing files and directories.
- When creating project directories:
  - Create with `mkdirs()` and then verify existence; throw `IOException` if creation failed.
- When iterating template files:
  - Copy directories with `FileUtils.copyDirectory(...)` and files with `FileUtils.copyFileToDirectory(...)`.
- When processing files recursively:
  - Recurse into directories, process files, then apply rename/move operations (so moved/renamed entries are returned by helper methods).

---

## Extension merging
- Project extensions are applied by a `ProjectDirectoryExtensionMerger` (merge extension directories into project directory). Treat extensions as additive overlays on a template.

---

## Naming & derived defaults
- Many values are computed from inputs in the following precedence order:
  - explicit value set on builder
  - derived from `githubRepository` if provided (e.g. `groupId` → `com.github.[user]`)
  - derived from `magicArg` when it contains `package.Class` form
  - fallbacks such as `my-app`, `My App`, `com.example.myapp`
- Use helper `StringUtils` for:
  - camelCase ↔ lower-case-with-separator conversions
  - `ucFirst`, `ucWords`, `lowerCaseWithSeparatorToCamelCase`
  - `isValidJavaClassName`, `countCharInstances`

Naming conventions used by helpers:
- artifactId/project-name → lower case with `-` separator
- class names → CamelCase; package names → dot-separated lowercase

---

## GitHub & git conventions
- GitHub repo operations:
  - Creating repository via REST `POST https://api.github.com/user/repos` using `HttpURLConnection` and a JSON body (`{"name": "...", "private": true/false}`).
  - Use a token from `GithubTokenService` in `Authorization: Bearer <token>`.
- Git operations use JGit:
  - Initialize repo with `Git.init().setDirectory(localPath).call()`.
  - Manage `origin` remote via JGit `RemoteConfig` and `URIish`, checking for existing URIs and adding if absent.
  - Commit with `git.add().addFilepattern(".").call()` and `git.commit().setMessage(...).call()`.
  - Push with `git.push().setRemote("origin").setCredentialsProvider(new UsernamePasswordCredentialsProvider(token, ""))`. (token passed as username and empty password)
- When dealing with releases for private repos:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shannah/jdeploy](https://github.com/shannah/jdeploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
