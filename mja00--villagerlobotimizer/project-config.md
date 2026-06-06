---
trigger: always_on
description: VillagerLobotimizer is a Minecraft Paper plugin written in Java 21 that optimizes server performance by disabling villager AI when they're trapped in trading halls. The project uses Gradle for building and includes sophisticated CI/CD for publishing to Hangar and Modrinth.
---

# VillagerLobotimizer Development Instructions

VillagerLobotimizer is a Minecraft Paper plugin written in Java 21 that optimizes server performance by disabling villager AI when they're trapped in trading halls. The project uses Gradle for building and includes sophisticated CI/CD for publishing to Hangar and Modrinth.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Prerequisites and Setup
- Set up Java 21 (required):
  ```bash
  export JAVA_HOME=/usr/lib/jvm/temurin-21-jdk-amd64
  export PATH=$JAVA_HOME/bin:$PATH
  java -version  # Should show OpenJDK 21
  ```
- Make Gradle wrapper executable: `chmod +x ./gradlew`
- Verify Gradle setup: `./gradlew --version`

### Building the Plugin
- **CRITICAL**: Build takes 2-10 minutes on first run due to dependency downloads. NEVER CANCEL. Set timeout to 30+ minutes.
- `./gradlew build --no-daemon` -- downloads dependencies and compiles. NEVER CANCEL.
- **Network Dependency**: Requires access to `repo.papermc.io` for Paper development bundle
- **Known Issue**: Build fails in sandboxed environments due to `repo.papermc.io` being blocked with error "No address associated with hostname". This is expected in restricted environments.
- **Error Signature**: `Could not resolve io.papermc.paper:dev-bundle:1.21.6-R0.1-SNAPSHOT` indicates network restriction
- **Workaround**: If PaperMC repository is blocked, the build cannot complete. Document this limitation rather than attempting fixes.
- Built plugin JAR will be in `build/libs/VillagerLobotimizer-<version>.jar`
- Uses shadow plugin, so the actual artifact is the shaded JAR (no classifier)

### Running Test Server
- **CRITICAL**: Server download and startup takes 10-20 minutes. NEVER CANCEL. Set timeout to 60+ minutes.
- `./gradlew runServer` -- downloads Paper server for Minecraft 1.21.8 and starts with plugin installed
- **Note**: This requires network access to download Paper server
- Server runs in interactive mode - you can issue Minecraft commands
- **Limitation**: Cannot interact with Minecraft GUI in headless environments

### Publishing (for maintainers)
- `./gradlew publishAll` -- publishes to both Hangar and Modrinth
- Requires `HANGAR_API_KEY` and `MODRINTH_TOKEN` environment variables
- Auto-detects release vs snapshot based on git tags

## Validation

### Manual Testing Scenarios
After making changes, ALWAYS run through these validation steps:

1. **Build Validation**:
   - `./gradlew build --no-daemon`
   - Verify build/libs/ contains the plugin JAR
   - Check that no compilation errors occurred

2. **Plugin Loading Validation**:
   - `./gradlew runServer`
   - Wait for server startup (shows "Done" message)
   - Type `plugins` to verify VillagerLobotimizer is loaded
   - Type `stop` to shutdown server cleanly

3. **Command Testing** (if server is running):
   - `/lobotomy info` -- should show plugin statistics
   - `/lobotomy debug toggle` -- should enable debug mode
   - Create test villagers and verify plugin behavior

4. **CI Validation**:
   - Always check that GitHub Actions workflows pass
   - PR builds must complete successfully before merging

### Performance Testing
- Test with confined villagers in trading halls
- Verify AI is disabled for trapped villagers
- Confirm trading functionality still works
- Monitor console for debug messages when debug mode enabled

## Common Tasks

### Repository Structure
```
.
├── .github/workflows/     # CI/CD pipelines (pr-build.yml, test.yml, publish.yml)
├── build.gradle.kts       # Kotlin DSL build configuration
├── gradle/               # Gradle wrapper files
├── gradlew              # Gradle wrapper script (Unix)
├── gradlew.bat          # Gradle wrapper script (Windows)
├── settings.gradle.kts   # Gradle settings
├── src/main/
│   ├── java/dev/mja00/villagerLobotomizer/  # Java source code
│   └── resources/        # Plugin resources (config.yml, plugin.yml)
├── README.md            # Project documentation
└── LICENSE              # MIT license
```

### Key Source Files
- `VillagerLobotimizer.java` -- Main plugin class
- `LobotomizeCommand.java` -- Command handling (/lobotomy commands)
- `LobotomizeStorage.java` -- Data storage and management
- `listeners/EntityListener.java` -- Entity event handling
- `utils/VillagerUtils.java` -- Villager-specific utilities

### Configuration Files
- `src/main/resources/plugin.yml` -- Plugin metadata for Paper
- `src/main/resources/config.yml` -- Default plugin configuration
- `build.gradle.kts` -- Build configuration with plugins:
  - `io.papermc.paperweight.userdev` -- Paper development
  - `xyz.jpenilla.run-paper` -- Test server runner
  - `com.gradleup.shadow` -- JAR shading
  - `io.papermc.hangar-publish-plugin` -- Hangar publishing
  - `com.modrinth.minotaur` -- Modrinth publishing

### Dependencies and Versions
- **Minecraft**: 1.21.6, 1.21.7, 1.21.8 (Paper API)
- **Java**: 21 (required)
- **Gradle**: 8.14.3

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mja00/VillagerLobotimizer](https://github.com/mja00/VillagerLobotimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-06 -->
