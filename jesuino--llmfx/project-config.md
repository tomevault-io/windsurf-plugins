---
trigger: always_on
description: When a Java developer asks you to "setup jDeploy" or similar requests, follow these instructions:
---

# Claude Instructions for jDeploy Setup

When a Java developer asks you to "setup jDeploy" or similar requests, follow these instructions:

## Overview

jDeploy setup focuses on configuring the project to work with jDeploy, whether installed as a desktop app or npm package. No installation of jDeploy is required during setup.

## 1. Prerequisites Check

First, verify the project structure:
- Check for `pom.xml` (Maven) or `build.gradle` (Gradle)
- Ensure the project builds successfully
- Identify if it has a main class
- Check current JAR output configuration

## 2. Configure Executable JAR Build

### Preferred: JAR with Dependencies in lib/ Directory

**Maven (using maven-dependency-plugin):**
```xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-dependency-plugin</artifactId>
   <version>3.2.0</version>
   <executions>
      <execution>
         <id>copy-dependencies</id>
         <phase>package</phase>
         <goals>
            <goal>copy-dependencies</goal>
         </goals>
         <configuration>
            <outputDirectory>${project.build.directory}/lib</outputDirectory>
         </configuration>
      </execution>
   </executions>
</plugin>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-jar-plugin</artifactId>
<version>3.2.2</version>
<configuration>
   <archive>
      <manifest>
         <addClasspath>true</addClasspath>
         <classpathPrefix>lib/</classpathPrefix>
         <mainClass>com.example.MainClass</mainClass>
      </manifest>
   </archive>
</configuration>
</plugin>
```

**Gradle (using application plugin):**
```gradle
plugins {
    id 'application'
}

application {
    mainClass = 'com.example.MainClass'
}

task copyDependencies(type: Copy) {
    from configurations.runtimeClasspath
    into "$buildDir/libs/lib"
}

jar {
    dependsOn copyDependencies
    manifest {
        attributes(
            'Main-Class': application.mainClass,
            'Class-Path': configurations.runtimeClasspath.collect { "lib/" + it.getName() }.join(' ')
        )
    }
}
```

### Alternative: Shaded/Fat JAR (if already configured)

If the project already produces a shaded JAR, that's acceptable:

**Maven (maven-shade-plugin):** Keep existing configuration
**Gradle (shadow plugin):** Keep existing configuration

## 3. Configure package.json

Create or modify `package.json` with required jDeploy configuration:

```json
{
   "bin": {"{{ appName }}": "jdeploy-bundle/jdeploy.js"},
   "author": "",
   "description": "",
   "main": "index.js",
   "preferGlobal": true,
   "repository": "",
   "version": "1.0.0",
   "jdeploy": {
      "jdk": false,
      "javaVersion": "21",
      "jar": "build/libs/{{ artifactId }}-1.0.0.jar",
      "javafx": true,
      "title": "{{ appTitle }}",
      "buildCommand": [
         "./gradlew",
         "buildExecutableJar"
      ]
   },
   "dependencies": {
      "command-exists-promise": "^2.0.2",
      "node-fetch": "2.6.7",
      "tar": "^4.4.8",
      "yauzl": "^2.10.0",
      "shelljs": "^0.8.4"
   },
   "license": "ISC",
   "name": "{{ appName }}",
   "files": ["jdeploy-bundle"],
   "scripts": {"test": "echo \"Error: no test specified\" && exit 1"}
}
```

### Key Configuration for Different Build Types:

**JAR with lib/ directory:**
```json
"jdeploy": {
  "jar": "target/myapp-1.0.jar",
  "javaVersion": "11",
  "title": "My Application"
}
```

**Shaded JAR:**
```json
"jdeploy": {
  "jar": "target/myapp-1.0-jar-with-dependencies.jar",
  "javaVersion": "11", 
  "title": "My Application"
}
```

**JavaFX Application:**
```json
"jdeploy": {
  "jar": "target/myapp-1.0.jar",
  "javaVersion": "11",
  "javafx": true,
  "title": "My JavaFX App"
}
```

**Compose Multiplatform Desktop Application:**
```json
"jdeploy": {
  "jar": "compose-desktop/build/libs/compose-desktop-1.0-SNAPSHOT-all.jar",
  "javaVersion": "21",
  "javafx": false,
  "title": "My Compose App",
  "buildCommand": [
    "./gradlew",
    ":compose-desktop:buildExecutableJar"
  ]
}
```

### Required Fields:
- `name`: Unique NPM package name
- `bin`: Must include `"jdeploy-bundle/jdeploy.js"`
- `dependencies`: Must include `"shelljs": "^0.8.4"`
- `jdeploy.jar`: Path to executable JAR
- `jdeploy.javaVersion`: Java version required
- `jdeploy.title`: Human-readable name

### Optional Fields:
- `jdeploy.jdk`: Set to true if full JDK required (default: false)
- `jdeploy.javafx`: Set to true for JavaFX apps (default: false)
- `jdeploy.args`: Array of JVM arguments

## 4. Find and Configure Application Icon

jDeploy uses an `icon.png` file in the project root (same directory as `package.json`) for the application icon.

### Search for Existing Icons

Look for icon files in common locations:
- `src/main/resources/` (Maven)
- `src/main/resources/icons/`
- `src/resources/`
- `resources/`
- `assets/`
- `images/`
- Project root directory

### Icon Requirements:
- **Format**: PNG format
- **Dimensions**: Must be square (256x256, 512x512, or other square sizes)
- **Filename**: Must be named `icon.png` in project root

### Steps to Configure Icon:

1. **Search for candidate icons:**
   ```bash
   find . -name "*.png" -o -name "*.ico" -o -name "*.icns" | grep -i icon
   find . -name "*.png" | head -10  # Check first 10 PNG files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jesuino/LLMFX](https://github.com/jesuino/LLMFX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
