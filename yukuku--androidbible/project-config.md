---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Bible for Android** (Alkitab / Quick Bible) — a 100% free, open-source Bible reader app for Android. Published on Google Play as "Alkitab" (Indonesian) and "Quick Bible" (non-Indonesian). The codebase supports multiple product flavors, 100+ downloadable Bible versions, song books, devotions, reading plans, cloud sync, and a daily verse widget.

- Official site: https://alkitab.app
- Developer docs: https://alkitab.app/developer

## Build Commands

```bash
# Debug APK (open-source build, works out of the box)
./gradlew assemblePlainDebug

# Debug App Bundle
./gradlew bundlePlainDebug

# Unit tests (same as CI)
./gradlew testPlainDebugUnitTest testPlainReleaseUnitTest

# Run a single test class
./gradlew testPlainDebugUnitTest --tests "yuku.alkitab.base.util.QueryTokenizerTest"

# Run a single test method
./gradlew testPlainDebugUnitTest --tests "yuku.alkitab.base.util.QueryTokenizerTest.testQuotedPhrases"
```

**Requirements**: JDK 17 (Zulu recommended), Android SDK with compile SDK 36, NDK 28.2.13676358.

The `plain` flavor is the open-source development build and works out of the box with the placeholder `Alkitab/google-services.json` checked into the repo (Firebase features won't function at runtime, but the app builds and runs). Production flavors (`yuku_alkitab`, `yuku_quick_bible`, `sabda_alkitab`) require:
- `$ALKITAB_PROPRIETARY_DIR/overlay/<applicationId>/text_raw/` — proprietary Bible text
- `$ALKITAB_PROPRIETARY_DIR/google-services.json` — real Firebase config (one file with client entries for all production applicationIds)
- Signing-key env vars: `SIGN_KEYSTORE`, `SIGN_ALIAS`, `SIGN_PASSWORD`

With those set, build with a plain `./gradlew assembleYuku_alkitabRelease` (or any other production flavor).

### Building in the Claude Code sandbox (one-time setup)

The Claude Code VM does not ship with a compatible JDK or the Android SDK. You must provision them yourself before running Gradle — do not rely on GitHub Actions for verification. Prefer `plainDebug` since it needs no proprietary overlay or signing secrets.

Install paths used below (pick any, but keep them consistent):

- JDK 17: `/home/user/tools/zulu17.64.17-ca-jdk17.0.18-linux_x64`
- Android SDK: `/home/user/android-sdk`

**Disk footprint:** expect ~6 GB across all of these combined — NDK r28c alone is ~2 GB unpacked, the rest of the SDK is ~1 GB, and `~/.gradle` grows to ~2 GB after the first build. Check free space before starting.

**Environment.** Every step after the JDK install needs the same env vars. Write them once and source them each time:

```bash
cat > /home/user/tools/android-env.sh <<'EOF'
export JAVA_HOME=/home/user/tools/zulu17.64.17-ca-jdk17.0.18-linux_x64
export ANDROID_HOME=/home/user/android-sdk
export PATH="$JAVA_HOME/bin:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$PATH"
unset JAVA_TOOL_OPTIONS  # strip the sandbox's -Dhttp.proxyHost that would poison Gradle downloads
EOF
```

Then `source /home/user/tools/android-env.sh` at the start of any shell that runs `sdkmanager` or Gradle.

1. **Install Zulu JDK 17** (the preinstalled JDK is 21, which the Android Gradle Plugin rejects for the `jvmToolchain(17)` used across modules):

   ```bash
   mkdir -p /home/user/tools && cd /home/user/tools
   curl -fsSL -o zulu17.tar.gz \
     https://cdn.azul.com/zulu/bin/zulu17.64.17-ca-jdk17.0.18-linux_x64.tar.gz
   echo "819e3f09ea628901a21b2104ed8f5256e17ae91a4145b272b2eb2131f832af1d  zulu17.tar.gz" | sha256sum -c -
   tar xzf zulu17.tar.gz && rm zulu17.tar.gz
   ```

2. **Trust the sandbox egress CA in the JDK truststore.** Outbound HTTPS in the Claude Code sandbox goes through an Anthropic TLS-inspection proxy (`sandbox-egress-production TLS Inspection CA`). `curl` trusts it via `/etc/ssl/certs`, but the JDK keeps its own `cacerts`, so `sdkmanager` and Gradle will fail with `PKIX path building failed` until you import the system CAs:

   ```bash
   JAVA_HOME=/home/user/tools/zulu17.64.17-ca-jdk17.0.18-linux_x64
   for crt in /usr/local/share/ca-certificates/*.crt; do
     "$JAVA_HOME/bin/keytool" -importcert -noprompt -trustcacerts \
       -keystore "$JAVA_HOME/lib/security/cacerts" -storepass changeit \
       -alias "$(basename "$crt" .crt)" -file "$crt"
   done
   ```

3. **Install the Android SDK command-line tools**, then use `sdkmanager` to fetch the exact packages Gradle expects (`compileSdk 36`, `build-tools 36.0.0`, `ndk 28.2.13676358` — the NDK is required because the `Snappy` module has JNI C++; note that `sdkmanager` resolves that NDK coordinate to `android-ndk-r28c` on disk, which is expected):

   ```bash
   mkdir -p /home/user/android-sdk/cmdline-tools && cd /home/user/android-sdk/cmdline-tools
   curl -fsSL -o cmdline-tools.zip \
     https://dl.google.com/android/repository/commandlinetools-linux-13114758_latest.zip
   echo "7ec965280a073311c339e571cd5de778b9975026cfcbe79f2b1cdcb1e15317ee  cmdline-tools.zip" | sha256sum -c -
   unzip -q cmdline-tools.zip && mv cmdline-tools latest && rm cmdline-tools.zip

   source /home/user/tools/android-env.sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yukuku/androidbible](https://github.com/yukuku/androidbible) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
