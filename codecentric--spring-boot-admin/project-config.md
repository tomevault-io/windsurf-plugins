---
trigger: always_on
description: Spring Boot Admin is a multi-module Maven project providing an admin interface for Spring Boot applications that expose actuator endpoints. It consists of a Vue.js frontend and Java backend components with 19 Maven modules.
---

# Spring Boot Admin
Spring Boot Admin is a multi-module Maven project providing an admin interface for Spring Boot applications that expose actuator endpoints. It consists of a Vue.js frontend and Java backend components with 19 Maven modules.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Environment Setup
- Install Java 17 (OpenJDK Temurin 17.0.16+ recommended) - project requires exactly Java 17
- Install Node.js 22.18.0 exactly (project specifies this in .nvmrc and package.json)
- Download Node.js 22.18.0: `curl -fsSL https://nodejs.org/dist/v22.18.0/node-v22.18.0-linux-x64.tar.xz -o /tmp/node.tar.xz`
- Extract and configure PATH: `cd /tmp && tar -xf node.tar.xz && export PATH="/tmp/node-v22.18.0-linux-x64/bin:$PATH"`
- Verify versions: `java -version` (should show 17.x) and `node --version` (should show v22.18.0)

### Building the Project
- **NEVER CANCEL builds - they take time but will complete successfully**
- **CRITICAL**: Set timeout to 20+ minutes for builds, 60+ minutes for tests
- Clean compile: `export PATH="/tmp/node-v22.18.0-linux-x64/bin:$PATH" && ./mvnw clean compile -B --no-transfer-progress -DskipTests` -- takes ~10.5 minutes
- Full package: `export PATH="/tmp/node-v22.18.0-linux-x64/bin:$PATH" && ./mvnw package -B --no-transfer-progress -DskipTests` -- takes ~4 minutes  
- Install to local repository: `export PATH="/tmp/node-v22.18.0-linux-x64/bin:$PATH" && ./mvnw install -B --no-transfer-progress -DskipTests` -- takes ~1.5 minutes

### Testing
- **NEVER CANCEL test runs - set 60+ minute timeouts**
- Full test suite: `export PATH="/tmp/node-v22.18.0-linux-x64/bin:$PATH" && ./mvnw test -B --no-transfer-progress` -- takes 20-40 minutes
- UI tests only: `cd spring-boot-admin-server-ui && export PATH="/tmp/node-v22.18.0-linux-x64/bin:$PATH" && npm run test` -- takes ~44 seconds
- UI build only: `cd spring-boot-admin-server-ui && export PATH="/tmp/node-v22.18.0-linux-x64/bin:$PATH" && npm run build` -- takes ~16 seconds

### Code Quality and Formatting
- Format Java code: `./mvnw spring-javaformat:apply`
- Check Java formatting: `./mvnw spring-javaformat:validate`
- Lint UI code: `cd spring-boot-admin-server-ui && npm run lint`
- Format UI code: `cd spring-boot-admin-server-ui && npm run format:fix`
- **ALWAYS** run `./mvnw spring-javaformat:apply` and UI formatting before committing
- **ALWAYS** run `./mvnw checkstyle:check` to verify compliance

### Running Sample Applications
- Build and install all modules first: `export PATH="/tmp/node-v22.18.0-linux-x64/bin:$PATH" && ./mvnw install -B --no-transfer-progress -DskipTests`
- Run servlet sample: `cd spring-boot-admin-samples/spring-boot-admin-sample-servlet && export PATH="/tmp/node-v22.18.0-linux-x64/bin:$PATH" && ../../mvnw spring-boot:run -Dspring-boot.run.profiles=dev,insecure`
- Access UI at: `http://localhost:8080` (username: user, password shown in logs or use insecure profile)
- Application starts in ~3 seconds and shows "all up" status with registered instance

### UI Development Mode
- For UI development: `cd spring-boot-admin-server-ui && npm run build:watch` (builds on file changes)
- Configure Spring Boot Admin Server with: 
  ```
  spring.boot.admin.ui.cache.no-cache: true
  spring.boot.admin.ui.resource-locations: file:../../spring-boot-admin-server-ui/target/dist/
  spring.boot.admin.ui.template-location: file:../../spring-boot-admin-server-ui/target/dist/
  spring.boot.admin.ui.cache-templates: false
  ```

## Validation Scenarios
- **MANUAL VALIDATION REQUIRED**: After building and running, always test actual functionality
- Launch sample application and verify Spring Boot Admin UI loads at `http://localhost:8080`
- Verify application registration: Should show "spring-boot-admin-sample-servlet" with UP status
- Test navigation: Click on Applications, Wallboard, and instance details
- Check endpoints: Verify actuator endpoints are detected and accessible
- Test monitoring features: Instance details should show health, metrics, loggers, etc.

## Key Modules and Locations

### Primary Components
- **Root**: `/` - Main Maven reactor project (pom.xml)
- **Server Backend**: `spring-boot-admin-server/` - Core Spring Boot Admin server functionality
- **UI Frontend**: `spring-boot-admin-server-ui/` - Vue.js 3 web interface with Vite build
- **Client**: `spring-boot-admin-client/` - Client library for connecting applications
- **Documentation**: `spring-boot-admin-docs/` - Asciidoc documentation

### Starter Modules
- **Server Starter**: `spring-boot-admin-starter-server/` - Auto-configuration for servers
- **Client Starter**: `spring-boot-admin-starter-client/` - Auto-configuration for clients

### Sample Applications (in spring-boot-admin-samples/)
- **servlet**: Standard servlet-based sample (recommended for testing)
- **reactive**: WebFlux reactive sample
- **eureka**: Eureka discovery integration
- **consul**: Consul discovery integration
- **hazelcast**: Hazelcast session management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codecentric/spring-boot-admin](https://github.com/codecentric/spring-boot-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
