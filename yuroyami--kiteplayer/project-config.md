---
trigger: always_on
description: One Kotlin engine: an actor loop, worker lanes, a quiesce handshake, a sync law and a seek
---

# KitePlayer, for whoever works in this tree

One Kotlin engine: an actor loop, worker lanes, a quiesce handshake, a sync law and a seek
machine. Containers, codecs and platform output all arrive through the service interface in
`kiteplayer-core`'s `spi` package. The media library lives in the sibling checkout,
`../KiteFFmpeg`, and is its own repository with its own issue tracker.

`CONTRIBUTING.md` has the ground rules, the gate and the build prerequisites. This file has only
what reading the code or running the gate would not teach you.

## How work happens here

- Future audioviz pattern revamp specs, analyses and implementation plans live only in
  `audioviz-revamp/`; start with its `README.md`. That folder is not tracked, because its evidence
  is device video and stills, so it exists only on the owner's machine. Revamp patterns
  individually on the accepted technical base. Keep documentation of implemented public contracts
  in `docs/`.
- Work on `main`. Never create a branch without asking. Commit locally, never push. The owner
  pushes, publishes and cuts every release.
- Commit subject is one imperative sentence about the outcome. Short prose body. No trailers.
- Every commit is authored and committed as `yuroyami <youcefsidena@gmail.com>`, whatever git
  identity the machine came with. A cloud container can arrive set to Claude, so check
  `git config user.name` before the first commit. Never name Claude in a commit: not as author,
  not as committer, and no `Co-Authored-By` or session line.
- Every change starts with an issue, and the commit that closes one says `Fixes #n` in its body.
- Talk to the owner in plain words. No internal codes, no jargon walls. Say what a thing means,
  not what it is. A question must be answerable by someone who has read nothing.

## Gotchas

Each line is something that bit someone. Delete a line when it stops being true.

### The gate and the tools around it

- `run-c-tests.sh` never builds anything, so on its own it proves nothing about a source change;
  run the variant's build script first, every time.
- A Gradle compile task with no sources prints `NO-SOURCE` and exits zero, so "the target compiles
  now" can mean "there was never anything there to compile". Grep the log for that word against the
  exact task name, or check that the run reports a test count rather than a build result.
- A Kotlin/Native test report gives every test a time near zero, so a test that returned early
  because it found no fixture reads exactly like one that ran. To prove a native test reads its
  media, hide the fixture once and watch it fail.
- `./gradlew ... | tail` reports the exit code of `tail`. A background build once reported success
  with BUILD FAILED sitting in its own log.
- Moving or renaming the checkout breaks the prebuilt C test binaries: they carry an absolute path
  to their interpose library from link time, so every suite aborts naming the old path, which reads
  like a broken test and is a stale binary. Rebuild them; a directory move counts as a C change.
- Adding a dependency can poison Kotlin's incremental-compilation cache, and the failure names a
  standard library function and reads like a compiler bug in your own code. Delete the module's
  `build/kotlin` and build again. The same failure on the web target wants
  `build/classes/kotlin/wasmJs` deleted and a rerun with tasks forced. A large edit across many
  files in one module does the same in a second shape: the run fails with `NoClassDefFoundError`
  for one of our own classes, usually a companion, because that class file was never written.
  Delete `build/kotlin` and `build/classes/kotlin/jvm`.
- A Gradle test run that is killed part way leaves its results directory unusable, and the next run
  fails before any test with `NoSuchFileException ... in-progress-results-generic.bin`. Delete
  `build/test-results/<task>` and run again.
- The js browser tests cannot run in a checkout whose path contains `#`, such as one under `#Kite`.
  Kotlin/JS serves each test file as `/absolute/<path>`, the browser cuts that URL at the `#`, and
  the task fails with two 404 lines and no test result. The wasmJs browser tests and both Node
  runners are unaffected, and CI's path has no `#`.
- `audiovizSurvey` takes its classpath from `jvmTest`, so it depends on it: one red fast test stops
  the whole survey before it draws anything. The XML you then read is the previous run's, with the
  previous numbers. Compare the file's timestamp with the source you edited before you believe it.
- The survey takes tens of minutes and holds a test worker of its own. A second Gradle build in
  this checkout while it runs rewrites the classes under it, and the survey dies part way with a
  socket timeout or a missing class. Wait for it, or run the second build somewhere else.
- Scraping every Gradle configuration gives a load-dependent answer, because which configurations
  are realised depends on the rest of the task graph. The publication readiness check passed alone
  and failed inside a full gate run, reporting that a publishing module depended on the sample,
  which no build file says. Only `api`, `implementation`, `compileOnly` and `runtimeOnly` can reach
  a POM.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuroyami/KitePlayer](https://github.com/yuroyami/KitePlayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
