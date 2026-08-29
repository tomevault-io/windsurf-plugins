---
trigger: always_on
description: JvmGuard is a production JVM-profiling server: a **Spring Boot** application (Spring DI, embedded Tomcat)
---

# JvmGuard — agent guide

JvmGuard is a production JVM-profiling server: a **Spring Boot** application (Spring DI, embedded Tomcat)
that serves the Vaadin 25 web UI and hosts the backend collaborators as Spring beans, reusing the
in-process `Server`/`ServerConnection` from `:backend:connector`. Build is **Gradle (Kotlin DSL)**,
rooted at this directory; modules use the standard **Maven source layout**
(`modules/<m>/src/main/{java,kotlin,resources}`, `src/test/...`) with build files named
`modules/<m>/build.gradle.kts`. Java baseline is **25** (provisioned by the
[foojay toolchain resolver](https://github.com/gradle/foojay-toolchains); the system `java`
may be older). The Gradle build is **fully self-contained**: the build logic lives in the local
`buildSrc` (under the `dev.jvmguard.build.*` packages), versions are declared once in
`gradle/libs.versions.toml`, and module registration is explicit in `settings.gradle.kts`.
**Never commit or push** — the user reviews and commits all changes.

## Backend — `modules/server` (`:server`, the Spring Boot app)

`dev.jvmguard.server.ServerMain` (`@Component`, `SmartLifecycle`) is the `main()` entry (no args); it builds a
`SpringApplication(JvmGuardApplication.class)`, registers a `bootstrap` initializer that applies a pending
`jvmguard.bak` restore before the auto-configured `DataSource` opens the H2 files (and inits agent-side logging),
selects the `integrationTest` profile when applicable, and runs it. `JvmGuardApplication` is `@SpringBootConfiguration
@EnableAutoConfiguration @EnableVaadin("dev.jvmguard.ui") @Import(SpringConfiguration.class)`;
`SpringConfiguration` component-scans the backend packages (`dev.jvmguard.common/data/collector/
database/rest/connector`) and `@Import`s the server-module beans. Backend collaborators are Spring
beans with **constructor injection**; the agent `ConnectionServer`, collector, telemetry, REST, and the
embedded web server are all in the one context. Notes:
- The in-process `Server` is published through `ServerFactory.setLocalServer(...)` (a static holder, so
  the agent-facing `connector` classes can reach it without the Spring context); inside the context,
  collaborators use Spring constructor injection.
- **Config** is standard Spring: the `jvmguard:` section of `application.yaml` binds to
  `dev.jvmguard.common.JvmGuardProperties` (`@ConfigurationProperties("jvmguard")`, `-Djvmguard.*` overrides),
  constructor-injected. Defaults ship in `modules/server/src/main/resources/application.yaml`; installs
  override via `<install>/config/application.yaml`. `JvmGuardEnvironmentPostProcessor` (in
  `META-INF/spring.factories`) runs before refresh: it binds the properties, resolves the install-layout
  directories (`JvmGuardDirectories` + the jar-vs-dev `LoadingDescriptor`, exposed as a bean), and publishes
  the early bootstrap keys (`server.port` from `httpPort`, `logging.config`). Static callers
  (`PasswordHelper`, `NetworkView`) read the bound properties via the
  `JvmGuardConfig` holder. `integrationTest` is the Spring profile, selected in `ServerMain.main()`.
- **HTTPS / reverse-proxy** are applied to Boot's embedded Tomcat by `WebServerCustomizer` (a
  `WebServerFactoryCustomizer<TomcatServletWebServerFactory>`: `factory.setSsl(...)` from the keystore
  resolved by `WebServerSupport`, a `connector.setProperty("server","")` customizer that suppresses the
  Server header, and a `RemoteIpValve` engine valve when `reverseProxy` is set).
- **Security is Spring Security** (`SecurityConfiguration`): an `@Order(0)` stateless HTTP-Basic chain for
  the REST API (`/api/**`, opt-in via `restApiEnabled`, `RestApiKeyAuthenticationProvider`) and an
  `@Order(1)` Vaadin chain (`VaadinSecurityConfigurer` + `loginView`). Web views carry JSR-250
  `@AnonymousAllowed`/`@PermitAll`/`@RolesAllowed(Roles.*)` (`Roles` in `dev.jvmguard.data.user`); the
  principal is `JvmGuardUserDetails` (authorities expand the `AccessLevel` hierarchy). Login flows
  `LoginView` → `SecurityBridge` (a `ServerFactory`-style holder, since views aren't beans) →
  `AuthenticationManager` → `JvmGuardAuthenticationProvider` → `Server.authenticate` then `Server.connect`.
  Failed logins are throttled by `LoginThrottle` (`dev.jvmguard.common.helper`, exponential backoff after
  3 free attempts), shared by the Vaadin login and the REST chain.
- **Backend authorization is Spring method security** (`@EnableMethodSecurity`): the gated
  `ServerConnectionImpl` / `AbstractServerConnectionImpl` methods carry the meta-annotations
  **`@RequireAdmin` / `@RequireProfiler` / `@RequireViewer`** (in `dev.jvmguard.data.user`, each composing
  `@PreAuthorize("hasRole(...)")` over a `Roles` constant, never hand-written), enforced via an AOP proxy of
  the prototype connection. This is the authoritative access-level boundary; route annotations are UX +
  defense-in-depth. The **REST** resources use the same `@Require*` markers (the chain only authenticates),
  with a 403 handler in `RestExceptionHandler` and `RestInterface` constructor-injected (no static holder).
  REST endpoints produce `text/plain`, so a client must `Accept` it (default is JSON). **Group-scoping** and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jvmguard/jvmguard](https://github.com/jvmguard/jvmguard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
