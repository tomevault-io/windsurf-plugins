---
trigger: always_on
description: Guidance for Coding Agents working inside the `primefaces-integration-tests` module.
---

# primefaces-integration-tests

Guidance for Coding Agents working inside the `primefaces-integration-tests` module.
Read the root [`AGENTS.md`](../AGENTS.md) first for repository-wide conventions, and
[`primefaces-selenium/AGENTS.md`](../primefaces-selenium/AGENTS.md) for the test
**framework** this module builds on. This file focuses on **how to add a test and keep
component coverage healthy**.

## Purpose

This module is the **browser-based regression suite** for the PrimeFaces component
library. Each test boots a real Faces application (embedded Tomcat + Weld + Mojarra or
MyFaces), drives a page with Selenium, and asserts component behavior, rendered markup,
widget configuration, and the absence of JavaScript errors. It is the primary guard
against regressions in real component rendering and client-side behavior — unit tests in
`primefaces` cover server-side logic only.

It consumes `primefaces-selenium` (the reusable framework) as a `test`-scoped dependency.
This module supplies the **deployment** (`TomcatDeploymentAdapter`), the **views and
beans** under test, and the **tests** themselves.

## The test triad

Almost every test is three coordinated artifacts that share a numbered name
(`<Component><NNN>`). To exercise the `p:dataTable` "basic + paginator" scenario:

| Role | Location | Example |
|---|---|---|
| **View** (`.xhtml`) | `src/main/webapp/<component>/` | `datatable/dataTable001.xhtml` |
| **Backing bean** (CDI) | `src/main/java/.../integrationtests/<component>/` | `DataTable001.java` (`@Named @ViewScoped`) |
| **Test** (JUnit 5) | `src/test/java/.../integrationtests/<component>/` | `DataTable001Test.java` |

The view's `getLocation()` / `goTo()` path is **relative to the webapp root** and matches
the file under `src/main/webapp` (e.g. `datatable/dataTable001.xhtml`). The bean is wired
into the view by EL name (`#{dataTable001...}`). Backing beans use Lombok `@Data` and a
`@PostConstruct init()` to seed data, often via a shared `*Service` (e.g.
`ProgrammingLanguageService`).

> Not every test needs a bean — purely static markup scenarios can use a view alone, and
> a bean can back several numbered views. But keep the `NNN` number aligned across the
> three files so the triad is greppable.

## Numbering & naming conventions

- **`<Component><NNN>`** where `NNN` is zero-padded and increments per scenario:
  `DataTable001`, `DataTable002`, … A higher number is a *new scenario*, not a fix to an
  existing one — add a new triad rather than overloading an existing view.
- Optional suffix describing the angle: `DataTable005PagingTest`, `DataTable007LazyTest`,
  `DataTable007CellTest`.
- Test classes end in `Test` and are **package-private** (`class DataTable001Test`).
- Test **methods** carry `@Order(n)` and `@DisplayName("Component: human description")`.
  Reference the GitHub issue in the display name or a comment when a test pins a specific
  bug, e.g. `"DataTable: GitHub #7193 global filter..."`.
- Group reusable setup/data helpers in an `Abstract<Component>Test` per component package
  (see `AbstractDataTableTest`), and cross-component table helpers in the root
  `org.primefaces.integrationtests.AbstractTableTest`.

## Tagging tests by behavior

The `NNN` number is **insertion order, not feature** — the same behavior (filtering,
selection, …) is spread across many numbered classes, so the suite is hard to navigate by
*what* it covers. To make it navigable by behavior, `datatable` test classes carry JUnit 5
`@Tag`s at the **class level** (a class may carry several). Tags are prefixed with the
component name (`DataTable-`) so a multi-component run can target one component's behavior.
JUnit tags may not contain whitespace, so use a hyphen, not `" - "`. Vocabulary currently
in use:

| Tag | Covers |
|---|---|
| `DataTable-filter` | column / global / lazy filtering, filter operators (lt, in, between, NOT, …) |
| `DataTable-sort` | single / multi / custom sort |
| `DataTable-selection` | row & checkbox selection, (de)select-all, disabled selection |
| `DataTable-paginator` | paging behavior and paginator row-count |
| `DataTable-lazy` | `LazyDataModel`-backed tables |
| `DataTable-edit` | cell / row editing, add & delete row |
| `DataTable-rowgroup` | row grouping (header / summary / rowspan) |
| `DataTable-mvs` | MultiViewState |
| `DataTable-rowexpansion` | expandable rows / row toggler |
| `DataTable-frozencolumns` | frozen columns |
| `DataTable-scrolling` | sticky header, virtual scroll |
| `DataTable-rendering` | conditional render, null values, container-not-rendered, impl quirks |

Run one behavior across the whole suite with the JUnit tag filter (`-Dgroups`):

```bash
mvn verify -f primefaces-integration-tests/pom.xml \
    -Pintegration-tests,mojarra-4.0 -Dgroups=DataTable-filter
```

When adding a `datatable` test, tag it with the behavior(s) it exercises. Keep the
vocabulary small — reuse an existing tag rather than coining a near-synonym. (Currently
only the `datatable` package is tagged; extend the same `<Component>-<behavior>` scheme to
other components as needed.)

## Anatomy of a test

Tests extend `AbstractPrimePageTest` (from the framework). Two equally valid styles exist
in the codebase — match the surrounding package:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [primefaces/primefaces](https://github.com/primefaces/primefaces) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
