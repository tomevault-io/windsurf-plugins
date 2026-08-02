---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

`methodTraceLog` is a Spring Boot starter that wraps application code with a method-tracing AOP aspect. For every intercepted method it emits a `ServiceCallInfo` (traceid / spanid / pspanid, class, signature, args, return value or exception, action enum, timestamp) which is then fanned out to a list of `ICallService` beans. Two are built in: one writes a structured log line, the other records Micrometer `Timer` samples and builds a parent/child `MethodTraceInfo` tree that the web panel renders. A second feature, gated by `method-trace-log.file.enable`, reads/parses log files via `NIO WatchService` and tail-pushes new lines over STOMP to `/topic/log-monitor`. The decompiler (`DecompilerUtils`, CFR-backed) is exposed as an HTTP endpoint at `GET /methodTraceLog/decompile`, gated by an optional `X-Api-Key` filter.

A companion module `methodTraceLog-mcp` is a separate, standalone Spring Boot process that speaks Model Context Protocol over stdio and forwards `@Tool` calls to one or more hosts (each host is a separate app that has the starter on the classpath) over HTTP. This is how AI agents get access to the trace / log / decompile data without bringing in any AI framework themselves.

The published artifact is `com.gitee.wb04307201.methodTraceLog:methodTraceLog-spring-boot-starter`. The working tree builds `1.0-SNAPSHOT`.

## Build, test, run

Maven multi-module project, Java 17. Run from the repo root.

- Build everything and install into the local repo: `mvn install`
- Build/test a single module and its dependencies: `mvn -pl methodTraceLog-test -am package`
- Run all tests: `mvn test`
- Run a single test class: `mvn -pl methodTraceLog-test test -Dtest=AbstractCallServiceTest`
- Run a single test method: `mvn -pl methodTraceLog-test test -Dtest=AbstractCallServiceTest#transContext_withArray_shouldConvertToList`
- Launch the sample app: `mvn -pl methodTraceLog-test spring-boot:run` (the test module does not declare `spring-boot-maven-plugin`; run via `java -cp ...` or by adding the plugin to the module).
- Launch the MCP server: `java -jar methodTraceLog-mcp/target/methodTraceLog-mcp-1.0-SNAPSHOT.jar` (talks over stdio).
- The Maven `mvn` command on this machine must be invoked via `/c/developer/apache-maven-3.9.16/bin/mvn` (the default `mvn` shim is broken).

Module layout:
- `methodTraceLog` — core library (aspect, strategy, properties, services, utils, file + decompiler, panel.html + static assets). No AI dependencies.
- `methodTraceLog-spring-boot-autoconfigure` — `@AutoConfiguration` classes registered through `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (`LogConfig`, `LogFileConfig`, `ApiKeyFilter`).
- `methodTraceLog-spring-boot-starter` — empty wrapper that depends on the autoconfigure module.
- `methodTraceLog-mcp` — standalone Spring Boot MCP server (separate process, stdio transport, talks to hosts via HTTP).
- `methodTraceLog-test` — runnable sample + JUnit 5 tests for `AbstractCallService`, `LogActionEnum`, `LogQueryRequest`, `ValidationUtils`, `DecompilerUtils`. No test covers the AOP wiring end-to-end; verify changes by running the sample app and hitting the endpoints.

## Configuration

`method-trace-log.*` is bound by `MethodTraceLogProperties` (`@ConfigurationProperties`). Four top-level groups, each switchable independently:

- `method-trace-log.log.enable` — gates `LogConfig` and the whole aspect/AOP machinery.
- `method-trace-log.file.enable` — gates `LogFileConfig`, STOMP broker, file reading, and the WebSocket controller.
- `method-trace-log.security.api-key` — when non-empty, the `ApiKeyFilter` requires an `X-Api-Key` header on every `/methodTraceLog/**` request except the `/panel` HTML page itself. If empty, the filter is a no-op (development only).
- `method-trace-log.decompile.timeout-seconds` — per-call CFR timeout for the `DecompilerUtils` and the `/methodTraceLog/decompile` endpoint.
- `method-trace-log.log.serviceCalls` — per-`ICallService` enable flags applied at startup. `CallServiceStrategy.setCallServiceEnable(name, enable)` can flip them at runtime via `GET /methodTraceLog/view/callService?name=...&enable=...`.
- `method-trace-log.file.*` — log path, allowed extensions, `maxLines`, `maxFileSize`, and the regex `logPattern` used by `LogLineInfo.parse`. The default pattern matches `%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n`.
- `management.endpoints.web.exposure.include=methodtrace` — exposes the `MethodTraceLogEndPoint` actuator endpoint used by the panel.

MCP server config (`methodTraceLog-mcp/src/main/resources/application.yml`):
- `method-trace-log.mcp.hosts[].{name,url,description,api-key}` — each host is a `RouterFunction`-serving Spring Boot app. `api-key` (if set) is forwarded as `X-Api-Key` to the host.

Sample app config lives in `methodTraceLog-test/src/main/resources/application.yml`; logback uses `${LOG_DIR}/${APP_NAME}.log` and a daily rolling file.

## Big-picture architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wb04307201/methodTraceLog](https://github.com/wb04307201/methodTraceLog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
