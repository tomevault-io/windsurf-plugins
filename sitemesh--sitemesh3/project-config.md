---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Scope

This is the **3.3.x / master** branch of SiteMesh 3. It targets **Java 17+**, **Jakarta EE 11 (Servlet 6.1)**, **Jetty 12**, **Tomcat 11**, and **Spring Boot 4.x**. The 3.1.x (Servlet 3–4, Java 8) and 3.2.x (Jakarta EE 9/10) lines live on separate branches — do not backport API changes to them from here without checking the compatibility matrix in `README.md`.

## Build & Test

Gradle multi-project build (`settings.gradle`). JDK 17 is enforced via toolchain in the root `build.gradle`.

- Full test suite: `./gradlew test`
- Build jars: `./gradlew jar`
- Javadoc: `./gradlew javadoc`
- Dependency update report: `./gradlew dependencyUpdates`
- Publish to Sonatype: `./gradlew publishToSonatype` (requires `signing.key` / `signing.password` / `sonatypeUsername` / `sonatypePassword` gradle properties)
- Run a single test class: `./gradlew :sitemesh:test --tests org.sitemesh.content.tagrules.html.SomeTest`
- Run a single test method: `./gradlew :sitemesh:test --tests 'org.sitemesh.content.tagrules.html.SomeTest.testMethodName'`

### Running the examples

Example project paths follow the directory layout (`examples/<name>` -> `:examples:<name>`):

- Tomcat 11 (default): `./gradlew :examples:hellowebapp:tomcatRun`
- Jetty 12: `./gradlew :examples:hellowebapp:jettyRun` (overrides gretty to jetty12)
- Spring Boot on Tomcat (default): `./gradlew :examples:springboot:bootRun`
- Spring Boot on Jetty: `./gradlew :examples:springboot:bootRun -Pcontainer=jetty`
- Javalin: `./gradlew :examples:javalin:run`
- Struts on Tomcat 11 / Jetty 12: `./gradlew :examples:struts:tomcatRun` / `:examples:struts:jettyRun`
- Micronaut (Netty; needs JDK 25): `./gradlew :examples:micronaut:run`

Whenever a change touches response buffering, dispatch, or content-type handling, **run both the Tomcat and Jetty variants** of `hellowebapp` and `springboot` before declaring success. The containers have diverged enough (see below) that one can mask bugs in the other.

### JFlex lexer generation

`sitemesh/src/main/java/org/sitemesh/tagprocessor/lexer.flex` is compiled by JFlex into `sitemesh/build/generated-sources/jflex/` as part of `compileJava` (see `sitemesh/build.gradle`). If you edit the HTML tokenizer, edit the `.flex` source — not the generated `Lexer.java`.

## Modules

- `sitemesh` — the core library. Content-free of servlet runtime deps (`jakarta.servlet-api` is `compileOnly`) so it can also be used offline from the command line / Ant task / Java API.
- `sitemesh-webfilter` — thin module that exposes the servlet filter integration. Depends on `:sitemesh` via `api`.
- `spring-webmvc-sitemesh` — plain Spring Web MVC integration (no Boot dependency): `SiteMeshViewResolver` wraps a `ViewResolver` and decorates resolved views with `SiteMeshView` inside MVC rendering, avoiding servlet-response buffering entirely.
- `spring-boot-starter-sitemesh` — auto-configuration starter for Spring Boot 4.x. **Default integration is `view-resolver`** (a single non-invasive delegating resolver via `wrapMode=delegate`); the classic servlet filter is opt-in via `sitemesh.integration=filter`. Spring Boot properties documented in `CONFIGURATION.md`.
- `examples/hellowebapp` — WAR built with the `gretty` plugin; exercises both Tomcat and Jetty against plain servlets + JSP + JSTL.
- `examples/springboot` — Spring Boot app with Thymeleaf + FreeMarker + JSP views; container swap via `-Pcontainer=jetty`.
- `examples/javalin` — Javalin + FreeMarker example.

## Architecture

SiteMesh's job is to intercept an HTML response, parse it into named properties (`title`, `head`, `body`, plus custom properties), and merge those into a decorator template. Understanding the flow means tracing a request through four layers:

1. **Filter / integration layer** (`org.sitemesh.webapp`, `org.sitemesh.config`)
   - `SiteMeshFilter` is the servlet filter entry point. It wraps the response in a buffering wrapper and delegates to the next filter.
   - `ConfigurableSiteMeshFilter` is the user-facing subclass; it reads `/WEB-INF/sitemesh3.xml`, then calls `applyCustomConfiguration(SiteMeshFilterBuilder)` for programmatic overrides. XML + Java can be combined.
   - `WebAppContext` performs the dispatch to the decorator template via a configurable `DispatchMode` (`include` / `forward` / `detect`, default `detect`). See "Dispatch gotcha" below.
   - Decorator selection is pluggable via `DecoratorSelector` implementations: `MetaTagBasedDecoratorSelector` (default), `PathBasedDecoratorSelector`, `RequestAttributeDecoratorSelector`.

2. **Response buffering layer** (`org.sitemesh.webapp.contentfilter`)
   - `HttpServletResponseBuffer` wraps the response, routes writes into a `Buffer`, and decides per-request whether to buffer based on content type. `RoutableServletOutputStream` swaps the destination between the buffer and the underlying stream.
   - Servlet 6.1 introduced `ServletOutputStream.write(ByteBuffer)` and `HttpServletResponse.setContentLengthLong(long)`. Both **must** be overridden in the wrappers, or content on newer containers silently bypasses the buffer. See `JAKARTA_UPGRADE.md` for full context.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sitemesh/sitemesh3](https://github.com/sitemesh/sitemesh3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
