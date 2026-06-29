---
trigger: always_on
description: Retromod transforms older Minecraft mod bytecode so old mods work on newer MC versions. It supports Fabric, NeoForge, and Forge with ~120 version shims and ~35 polyfill providers (counts drift; run `grep -cv '^#' META-INF/services/...` for the current numbers).
---

# Retromod - Claude Development Guide

Retromod transforms older Minecraft mod bytecode so old mods work on newer MC versions. It supports Fabric, NeoForge, and Forge with ~120 version shims and ~35 polyfill providers (counts drift; run `grep -cv '^#' META-INF/services/...` for the current numbers).

**Repository:** https://github.com/Bownlux/Retromod.git

## Writing style

**Do not use em-dashes (the long dash, Unicode U+2014) anywhere:** not in chat replies, not in code comments, docs, CHANGELOG/ROADMAP entries, or commit messages. Also avoid en-dashes (U+2013). They are annoying to copy out of responses. Use a comma, parentheses, a colon, or two separate sentences instead. (Ordinary hyphens `-`, e.g. in version ranges like `1.20-26.2`, are fine.)

## Critical Context

- **Target MC version:** 26.1 (Mojang removed ALL code obfuscation); 26.2 supported since 1.1.0-snapshot.4 (shims/aliases/Fabric build target in place, verified on 26.2-rc-1)
- **Java:** Built WITH Java 25 (we need the modern compiler to use ASM 9.8 features that read MC 26.1's class file format), but bytecode targets `--release 17` so the SAME JAR runs on Java 17, 21, and 25. Broad runtime compat is the entire point. `build-all.sh` sets the per-MC `"java"` constraint in fabric.mod.json based on what each MC version itself needs: `>=17` for MC 1.20-1.20.4, `>=21` for MC 1.20.5-1.21.x, **`>=25` for MC 26.x** (MC 26.1's own class files are Java 25 bytecode, so a Java 21 JVM can't load them). Don't accidentally bump `<release>` higher, or that locks out MC 1.20.x users on Java 17. ASM 9.8 itself runs on Java 8+; it just READS class files up to v69 (Java 25), which has nothing to do with what bytecode WE emit.
- **Intermediary names are dead in 26.1+.** All `class_XXXX`, `method_XXXX`, `field_XXXX` must map to Mojang official names.
- **NeoForge already uses Mojang names** since 1.17. NeoForge mods mainly need metadata patching, not name remapping.
- **Fabric mods use intermediary names**, so they need full intermediary→Mojang remapping for 26.1+.
- **ALL old shims (including pre-1.20.1) must stay.** People still translate 1.16.5, 1.14.4 mods. Shims are NOT separate build targets; they're all part of one build.
- **`Retromod.TARGET_MC_VERSION`** is auto-detected at runtime from the mod loader. NEVER hardcode version strings like `"1.21.11"`. Always use `Retromod.TARGET_MC_VERSION`.

## Architecture

```
src/main/java/com/retromod/
├── core/           Core runtime: Retromod, RetromodTransformer, version detectors, mod transformers
├── cli/            CLI tool: RetromodCli (analyze, batch, aot, shims, etc.)
├── aot/            AOT compiler: AotCompiler, HybridCompiler, FullAotCompiler
├── shim/           Version shims by loader (fabric/, neoforge/, forge/, api/)
│   └── ShimRegistry.java   BFS-based shim chain finder with version aliases
├── mapping/        IntermediaryToMojangMapper, MappingComposer (26.1 core feature)
├── mixin/          Mixin compatibility: MixinCompatibilityTransformer, MixinTargetRedirector
├── polyfill/       Removed API reimplementations (72+ polyfills across 10 providers)
├── embedder/       API embedding into mod JARs, ModVersionInfo record
├── resources/      Resource/data pack transforms
├── gui/            In-game GUI (title screen button, file picker, restart popup)
├── agent/          Java Agent mode (premain/agentmain)
├── legacy/         Legacy version support utilities
├── compat/         Compatibility layer
├── archive/        Archive handling
├── util/           Utilities
└── virtual/        Virtual filesystem/classes
```

## Build Commands

```bash
# Quick build (skip tests)
mvn package -q -DskipTests -Dexec.skip=true

# Full build with tests
mvn package -Dexec.skip=true

# Run tests only
mvn test -Dexec.skip=true

# Lite build (1.20+ only, smaller JAR, no legacy/third-party polyfills)
mvn package -P lite -DskipTests -Dexec.skip=true

# Run CLI command (deps not bundled in JAR, must use mvn exec)
mvn exec:java -Dexec.mainClass="com.retromod.cli.RetromodCli" -Dexec.args="<command>" -q
```

**Important:** Always pass `-Dexec.skip=true` during build to prevent Maven from running the CLI entrypoint.

Output JAR: `target/retromod-1.1.0.jar`

## Release integrity (self-hash)

Official builds embed a SHA-256 of Retromod's own classes in `SignatureVerifier.EXPECTED_SELF_HASH`. At startup the verifier re-hashes the running jar's `com/retromod/**` classes (excluding the relocated `com/retromod/shaded/**` deps and the verifier class itself) and reports `VERIFIED` on a match, otherwise it fires a fork notice. It's an integrity / modification check, **not** cryptographic anti-tamper: there's no secret key, so a determined attacker can recompute it; for real verification users compare the jar's SHA-256 against the value published on the releases page.

**Embed the hash as the LAST release step** (any source change shifts it):
```bash
mvn clean package -Dexec.skip=true                          # build the final jars
python3 scripts/compute-self-hash.py target/retromod-1.1.0-all.jar

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bownlux/Retromod](https://github.com/Bownlux/Retromod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
