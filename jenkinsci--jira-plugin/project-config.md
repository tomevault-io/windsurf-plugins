---
trigger: always_on
description: Guidance for coding agents working in this repository. See `CONTRIBUTING.md` for the
---

# AGENTS.md

Guidance for coding agents working in this repository. See `CONTRIBUTING.md` for the
human-oriented contribution workflow (pre-commit hooks, local Jenkins via docker-compose, the
Jira Cloud test instance) — this file focuses on what an agent needs to build, test, and stay
consistent with existing conventions.

## Project overview

A Jenkins plugin (`org.jenkins-ci.plugins:jira`) that integrates Jenkins with Atlassian Jira
(Cloud and Server), via `jira-rest-java-client` (pinned at `${jira-rest-client.version}` in
`pom.xml`, currently `6.0.2`). Packaging is `hpi`; the project uses the standard
`org.jenkins-ci.plugins:plugin` parent POM.

- `hudson.plugins.jira.JiraSite` — per-Jenkins-config representation of a Jira instance (URL,
  credentials, timeouts). `getSession(Item)` resolves credentials and hands back a `JiraSession`.
- `hudson.plugins.jira.JiraSession` — thin facade over `JiraRestService` (`public final
  JiraRestService service` field) with some business logic on top (status-id caching,
  fixVersion regex replace, etc.). This is what build steps/notifiers actually call through.
- `hudson.plugins.jira.JiraRestService` — wraps the Atlassian `ExtendedJiraRestClient` for most
  operations, plus a few raw Apache `fluent.Request` HTTP calls (`getVersions`, `getComponents`)
  for endpoints the Atlassian client doesn't expose. `BASE_API_PATH = "rest/api/2"` is used by
  those raw calls and by explicitly-built URIs (`addComment`, `releaseVersion`) — but the
  Atlassian client's *own* internal calls (`getIssueClient()`, `getMetadataClient()`, etc.)
  always route through `/rest/api/latest` regardless of that constant. Don't assume one base
  path applies everywhere; check which call path you're in.
- `hudson.plugins.jira.extension.*` — this plugin's own extensions on top of
  `jira-rest-java-client` for operations it doesn't support natively (extended version/
  mypermissions REST clients). `JiraSite.ExtendedAsynchronousJiraRestClientFactory` (nested in
  `JiraSite.java`) is the actual `JiraRestClientFactory` in use.
- `hudson.plugins.jira.JiraSessionFactory` — builds the `ExtendedJiraRestClient` +
  `JiraRestService` pair from a `JiraSite`, `URI`, and credentials; picks Basic vs. Bearer auth.
- Build steps / notifiers (`JiraIssueUpdater`, `JiraCreateIssueNotifier`,
  `JiraVersionCreatorBuilder`, `JiraReleaseVersionUpdateBuilder`, etc.) all go through
  `JiraSite.getSession(item)` — don't bypass it to call `JiraRestService` directly from plugin
  code (test code doing so intentionally, to validate the wire format, is the one exception —
  see Testing instructions below).

## Dev environment tips

- Requires Java 17+ (the Jenkins baseline in `pom.xml`, currently `2.492`, needs it) and Maven.
- If `mise` is configured (see `mise.toml`), prefix commands with `mise exec --`.
- `jira-rest-java-client-core` is built on Jersey 2 / Apache HttpClient 4, provided via the
  Jenkins-bundled `jersey2-api` / `apache-httpcomponents-client-4-api` plugins rather than
  bundled directly (see the exclusions block around the `jira-rest-java-client-*` dependencies
  in `pom.xml`). Check any new HTTP-related dependency against this before assuming it'll just
  work — prefer shaded/standalone artifacts (as done for `wiremock-standalone`) when there's a
  real risk of a Jersey/Jetty/Jackson version collision.
- `io.jenkins.plugins:jackson2-api` already provides Jackson (currently `2.18.3`) — don't add a
  competing direct Jackson dependency; check `mvn dependency:tree -Dincludes='com.fasterxml.jackson*'`
  if unsure.

## Setup commands

```sh
mvn clean install -DskipTests   # build the .hpi without running the test suite
mvn spotless:apply              # auto-fix formatting (run before every commit)
```

## Testing instructions

```sh
mvn clean test                                                        # full build + test suite
mvn test -Dtest=SomeTest                                              # single class
mvn test -Dtest=SomeTest#someMethod                                   # single method
mvn test -Dtest='JiraRestServiceWireMockTest,LiveJiraCloudE2ETest' -DfailIfNoTests=false
```

- Tests use `jenkins-test-harness` (`@WithJenkins` + a `JenkinsRule` parameter), which boots a
  real embedded Jenkins instance per test — this is normal, not a bug, and it's why individual
  test runs take a few seconds each.
- `surefire` is configured with `reuseForks=false`; don't remove that without checking why (see
  the comment above it in `pom.xml` — mock serialization issues otherwise).
- `src/test/resources/logging.properties` quiets jenkins-test-harness's own INFO-level boot
  logging so real test failures aren't buried in console noise. If a test failure seems to be
  missing its actual stack trace, check `target/surefire-reports/` directly — it always has the
  full detail regardless of console verbosity.
- `-Dtest=SomeClass` (or `#someMethod`) can make a passing `@WithJenkins` test fail with
  `class ... is missing its descriptor` — an extension-indexing artifact of filtering, not a real
  failure. Before treating that as a bug, confirm with a full `mvn test` (no `-Dtest`), which is
  what CI actually runs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jenkinsci/jira-plugin](https://github.com/jenkinsci/jira-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
