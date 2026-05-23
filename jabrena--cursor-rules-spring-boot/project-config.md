---
trigger: always_on
description: HikariCP is the default connection pool for Spring Boot and is known for being the fastest, most reliable connection pool available for Java applications. This guide will help you configure HikariCP optimally for your Spring Boot applications.
---

# Spring Boot HikariCP Connection Pool Configuration

HikariCP is the default connection pool for Spring Boot and is known for being the fastest, most reliable connection pool available for Java applications. This guide will help you configure HikariCP optimally for your Spring Boot applications.

## Implementing These Principles

These guidelines are built upon the following core principles:

- **Performance First**: Configure pool sizes based on your application's actual database concurrency needs
- **Resource Efficiency**: Balance connection availability with memory and database server resources
- **Monitoring & Observability**: Enable metrics and logging to understand pool behavior
- **Environment-Specific**: Adjust configurations based on development, testing, and production environments
- **Fail-Fast**: Configure appropriate timeouts to detect issues quickly

## Table of contents

- Rule 1: Essential Pool Sizing Configuration
- Rule 2: Connection Timeout and Lifecycle Management
- Rule 3: Health Check and Validation Configuration
- Rule 4: Performance Monitoring and Metrics
- Rule 5: Environment-Specific Configuration Strategies

## Rule 1: Essential Pool Sizing Configuration

**Title**: Right-size your connection pool based on application needs

**Description**: The most critical aspect of HikariCP configuration is determining the optimal pool size. Ask yourself: "How many concurrent database operations does my application actually need?" Most applications need far fewer connections than developers initially think.

**Key Questions to Ask:**
- How many concurrent users will access my application?
- How many database operations happen per user request?
- What's my database server's connection limit?
- Am I running multiple application instances?

**Good example:**

```yaml
# application.yml
spring:
  datasource:
    hikari:
      # Start with this formula: connections = ((core_count * 2) + effective_spindle_count)
      # For most web apps: 10-15 connections is often sufficient
      maximum-pool-size: 10
      minimum-idle: 5
      # Allow pool to shrink during low activity
      idle-timeout: 300000  # 5 minutes
```

```java
// For programmatic configuration
@Configuration
public class DatabaseConfig {
    
    @Bean
    @ConfigurationProperties("spring.datasource.hikari")
    public HikariConfig hikariConfig() {
        HikariConfig config = new HikariConfig();
        // Conservative pool sizing for most applications
        config.setMaximumPoolSize(10);
        config.setMinimumIdle(5);
        config.setIdleTimeout(300_000);
        return config;
    }
}
```

**Bad Example:**

```yaml
# application.yml - DON'T DO THIS
spring:
  datasource:
    hikari:
      # Too many connections - wastes resources and can overwhelm DB
      maximum-pool-size: 100
      minimum-idle: 50
      # Never let connections be idle - keeps unnecessary connections
      idle-timeout: 0
```

## Rule 2: Connection Timeout and Lifecycle Management

**Title**: Configure appropriate timeouts for reliable connection handling

**Description**: Proper timeout configuration ensures your application fails fast when database issues occur and doesn't hold onto stale connections. Ask yourself: "How long should my application wait for a database connection before giving up?"

**Key Questions to Ask:**
- What's an acceptable wait time for users when the database is under load?
- How quickly should I detect database connectivity issues?
- What's my application's typical query execution time?

**Good example:**

```yaml
# application.yml
spring:
  datasource:
    hikari:
      # Fast failure for connection acquisition
      connection-timeout: 20000      # 20 seconds - adjust based on your needs
      # Detect stale connections quickly
      max-lifetime: 1800000         # 30 minutes - less than DB connection timeout
      # Quick validation of connections
      validation-timeout: 5000       # 5 seconds
      # Test connections when borrowed from pool
      connection-test-query: SELECT 1
```

```java
// Programmatic configuration with monitoring
@Configuration
public class DatabaseConfig {
    
    @Bean
    @ConfigurationProperties("spring.datasource.hikari")
    public HikariConfig hikariConfig() {
        HikariConfig config = new HikariConfig();
        config.setConnectionTimeout(20_000);
        config.setMaxLifetime(1_800_000);
        config.setValidationTimeout(5_000);
        
        // Enable connection testing
        config.setConnectionTestQuery("SELECT 1");
        return config;
    }
}
```

**Bad Example:**

```yaml
# application.yml - DON'T DO THIS
spring:
  datasource:
    hikari:
      # Too long - users will think app is frozen
      connection-timeout: 120000
      # Too long - may exceed DB server timeout
      max-lifetime: 7200000
      # No validation - stale connections may be used
      # connection-test-query: # missing
```

## Rule 3: Health Check and Validation Configuration

**Title**: Implement robust connection health checking


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jabrena/cursor-rules-spring-boot](https://github.com/jabrena/cursor-rules-spring-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
