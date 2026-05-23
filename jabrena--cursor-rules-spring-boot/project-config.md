---
trigger: always_on
description: Guidelines for building Spring Boot applications as GraalVM native images for improved startup time, reduced memory footprint, and enhanced performance characteristics.
---

# Spring Boot Native Compilation

Guidelines for building Spring Boot applications as GraalVM native images for improved startup time, reduced memory footprint, and enhanced performance characteristics.

## Implementing These Principles

These guidelines are built upon the following core principles:

- **Compile-time analysis**: All code paths must be discoverable at compile time
- **Minimal reflection usage**: Avoid runtime reflection and dynamic class loading
- **Explicit configuration**: Configure native hints for reflection, resources, and proxies
- **Profile-guided optimization**: Use AOT processing and build-time optimizations
- **Resource efficiency**: Optimize for reduced memory usage and faster startup

## Table of contents

- Rule 1: Configure Native Build Tools and Dependencies
- Rule 2: Manage Reflection and Dynamic Features
- Rule 3: Handle Resources and Configuration Files
- Rule 4: Optimize Application Profiles for Native
- Rule 5: Test Native Image Compatibility
- Rule 6: Use AOT Processing and Build-time Hints

## Rule 1: Configure Native Build Tools and Dependencies

**Title**: Proper Native Build Configuration
**Description**: Configure Maven or Gradle with the Spring Boot Native plugin and GraalVM dependencies. Ensure all necessary build tools are properly set up for native compilation.

**Good example:**

```xml
<!-- pom.xml -->
<properties>
    <native.maven.plugin.version>0.9.28</native.maven.plugin.version>
    <spring-boot.version>3.5.0</spring-boot.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>

<profiles>
    <profile>
        <id>native</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <executions>
                        <execution>
                            <id>process-aot</id>
                            <goals>
                                <goal>process-aot</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
                <plugin>
                    <groupId>org.graalvm.buildtools</groupId>
                    <artifactId>native-maven-plugin</artifactId>
                    <executions>
                        <execution>
                            <id>add-reachability-metadata</id>
                            <goals>
                                <goal>add-reachability-metadata</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

**Bad Example:**

```xml
<!-- Missing native plugin configuration -->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <!-- No native image configuration -->
        </plugin>
    </plugins>
</build>
```

## Rule 2: Manage Reflection and Dynamic Features

**Title**: Minimize and Configure Reflection Usage
**Description**: Avoid runtime reflection and dynamic class loading. When reflection is necessary, provide explicit native hints or use Spring's AOT processing to register classes at build time.

**Good example:**

```java
// Use @RegisterReflectionForBinding for data classes
@RegisterReflectionForBinding({Person.class, Address.class})
@RestController
public class PersonController {
    
    @GetMapping("/person")
    public Person getPerson() {
        return new Person("John", "Doe");
    }
}

// For explicit reflection hints
@Component
public class ReflectionHints implements RuntimeHintsRegistrar {
    
    @Override
    public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
        hints.reflection()
            .registerType(MyClass.class, MemberCategory.INVOKE_DECLARED_CONSTRUCTORS)
            .registerType(AnotherClass.class, MemberCategory.DECLARED_FIELDS);
    }
}

// Use @ImportRuntimeHints to register hints
@ImportRuntimeHints(ReflectionHints.class)
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**Bad Example:**

```java
// Avoid runtime class loading and reflection
@RestController
public class BadController {
    
    @GetMapping("/dynamic")
    public Object getDynamic() throws Exception {
        // This will fail in native image
        Class<?> clazz = Class.forName("com.example.DynamicClass");
        return clazz.getDeclaredConstructor().newInstance();
    }
}
```

## Rule 3: Handle Resources and Configuration Files

**Title**: Explicit Resource Configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jabrena/cursor-rules-spring-boot](https://github.com/jabrena/cursor-rules-spring-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
