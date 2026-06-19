---
trigger: always_on
description: >
---


# Galaxio Gatling Pro

## Core Rules

Use Gatling `3.11.x` and Scala `2.13.x` as the Galaxio baseline.

Supported JVM combinations:

- Scala + sbt: Scala DSL, `src/test/scala`, optional `src/it/scala`.
- Scala + Maven: Scala DSL, `src/test/scala`, `scala-maven-plugin`.
- Scala + Gradle: Scala DSL, `src/gatling/scala`, Gradle `gatling` source set.
- Java + Maven: Java DSL, `src/test/java`.
- Java + Gradle: Java DSL, `src/gatling/java`, Gradle `gatling` source set.
- Kotlin + Maven: Java DSL from Kotlin, `src/test/kotlin`, `kotlin-maven-plugin`.
- Kotlin + Gradle: Java DSL from Kotlin, `src/gatling/kotlin`, Gradle `gatling` source set.

Prefer `org.galaxio.gatling-picatinny` helpers when the repo has the dependency.
Picatinny examples in this skill are Scala-first; for Java/Kotlin, keep the same
architecture and use small local config/feeder helpers if Picatinny is not exposed
through the project's chosen DSL.

Generated Scala code in sbt projects must pass:

```bash
sbt scalafmtAll scalafmtSbt
```

When changing existing repo, follow local style first. If no style, use this skill.

## Build Tool Matrix

Use the build tool's conventional Gatling source roots. Do not move everything into
`src/test/scala` just because the Galaxio template started as sbt.

| Build tool | Languages | Simulation source root | Resource root | Run one simulation |
| --- | --- | --- | --- | --- |
| sbt | Scala | `src/test/scala` or `src/it/scala` | `src/test/resources` | `sbt 'Gatling/testOnly <fqcn>'` |
| Maven | Scala | `src/test/scala` | `src/test/resources` | `./mvnw gatling:test -Dgatling.simulationClass=<fqcn>` |
| Maven | Java | `src/test/java` | `src/test/resources` | `./mvnw gatling:test -Dgatling.simulationClass=<fqcn>` |
| Maven | Kotlin | `src/test/kotlin` | `src/test/resources` | `./mvnw gatling:test -Dgatling.simulationClass=<fqcn>` |
| Gradle | Scala | `src/gatling/scala` | `src/gatling/resources` | `./gradlew gatlingRun --simulation <fqcn>` |
| Gradle | Java | `src/gatling/java` | `src/gatling/resources` | `./gradlew gatlingRun --simulation <fqcn>` |
| Gradle | Kotlin | `src/gatling/kotlin` | `src/gatling/resources` | `./gradlew gatlingRun --simulation <fqcn>` |

Compile/pre-flight commands:

```bash
sbt Gatling/compile
./mvnw test-compile
./gradlew testClasses
```

## Project Layout

Keep the Galaxio boundaries regardless of language or build tool. Only the source
root changes.

Canonical Scala/sbt layout:

```text
src/test/scala/org/galaxio/performance/
  performance.scala # package object with protocols
  cases/            # atomic actions: HTTP, Kafka, JDBC, AMQP, JMS
  feeders/          # custom feeders; prefer Picatinny feeders
  scenarios/        # flows built from cases
  *Simulation.scala # simulations only
src/test/resources/
  simulation.conf
  gatling.conf
  logback.xml
```

Same layout adapted to each build tool:

```text
# sbt or Maven + Scala
src/test/scala/org/galaxio/performance/{performance.scala,cases,feeders,scenarios,*Simulation.scala}
src/test/resources/{simulation.conf,gatling.conf,logback.xml}

# Maven + Java
src/test/java/org/galaxio/performance/{Performance.java,cases,feeders,scenarios,*Simulation.java}
src/test/resources/{simulation.conf,gatling.conf,logback.xml}

# Maven + Kotlin
src/test/kotlin/org/galaxio/performance/{Performance.kt,cases,feeders,scenarios,*Simulation.kt}
src/test/resources/{simulation.conf,gatling.conf,logback.xml}

# Gradle + Scala
src/gatling/scala/org/galaxio/performance/{performance.scala,cases,feeders,scenarios,*Simulation.scala}
src/gatling/resources/{simulation.conf,gatling.conf,logback.xml}

# Gradle + Java
src/gatling/java/org/galaxio/performance/{Performance.java,cases,feeders,scenarios,*Simulation.java}
src/gatling/resources/{simulation.conf,gatling.conf,logback.xml}

# Gradle + Kotlin
src/gatling/kotlin/org/galaxio/performance/{Performance.kt,cases,feeders,scenarios,*Simulation.kt}
src/gatling/resources/{simulation.conf,gatling.conf,logback.xml}
```

Keep boundaries strict:

- `cases`: request/action only. No workload. No scenario.
- `feeders`: data only. No requests.
- `scenarios`: business flow only. No injection profile.
- `simulations`: injection, protocols, max duration. No request definitions.
- `performance.scala`, `Performance.java`, or `Performance.kt`: shared protocols only.

Do not use Gradle's `src/test/*` for Gatling simulations unless the project already
customizes the `gatling` source set. Do not use Maven's `src/gatling/*` unless the
project explicitly customizes plugin/source directories.

## Imports

Scala DSL:

Base:

```scala
import io.gatling.core.Predef._
import io.gatling.http.Predef._
import org.galaxio.gatling.config.SimulationConfig._
```

Picatinny helpers:

```scala
import org.galaxio.gatling.feeders._
import org.galaxio.gatling.utils.IntensityConverter._
```

Protocol imports, only when needed:

```scala
import io.gatling.jms.Predef._
import org.apache.kafka.clients.consumer.ConsumerConfig
import org.apache.kafka.clients.producer.ProducerConfig
import org.galaxio.gatling.amqp.Predef._
import org.galaxio.gatling.jdbc.Predef._
import org.galaxio.gatling.kafka.Predef._

import scala.concurrent.duration.DurationInt
```

Only add assertion imports when user explicitly asks for NFR/assertions.

Java DSL:

```java
import static io.gatling.javaapi.core.CoreDsl.*;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [galax-io/galaxio-gatling-pro](https://github.com/galax-io/galaxio-gatling-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
