---
trigger: always_on
description: 이 파일은 Claude Code가 이 저장소에서 작업할 때 참고하는 컨텍스트입니다.
---

# CLAUDE.md

이 파일은 Claude Code가 이 저장소에서 작업할 때 참고하는 컨텍스트입니다.

## 프로젝트 개요

`@us-all/android-mcp` — ADB 기반 Android 디바이스 관리 MCP 서버. **75 도구**로 디바이스/앱/UI/logcat/에뮬레이터/파일/시스템/디버그 영역 커버. Pure ADB (Appium / uiautomator2 의존성 없음).

- **타겟**: Android 디바이스 또는 에뮬레이터 (`adb` 필요)
- **런타임**: Node 20+, stdio transport, TypeScript strict
- **포지셔닝**: Android-only specialist. 크로스플랫폼은 [mobile-next/mobile-mcp](https://github.com/mobile-next/mobile-mcp)
- **표준**: [@us-all MCP Standard](https://github.com/us-all/mcp-toolkit/blob/main/STANDARD.md) 준수

## 디렉토리

```
src/
├── index.ts            # MCP 서버 + tool() 헬퍼 + 카테고리별 등록
├── config.ts           # ENV 로딩 (ANDROID_TOOLS / ANDROID_DISABLE 토글)
├── adb.ts              # execFile 기반 ADB wrapper
├── tool-registry.ts    # CATEGORIES + @us-all/mcp-toolkit
├── resources.ts        # MCP Resources (android:// URI)
└── tools/
    ├── utils.ts        # wrapToolHandler, shellEscape, validation helpers
    ├── extract-fields.ts   # toolkit re-export
    ├── device.ts apps.ts ui.ts logcat.ts emulator.ts
    ├── files.ts system.ts debug.ts shell.ts
    └── aggregations.ts # analyze-app
fixtures/test-app/      # Android Gradle 테스트 앱 (스모크용)
```

## Build & Run

```bash
pnpm install
pnpm build              # tsc → dist/
pnpm test               # vitest unit tests
pnpm smoke              # 실 디바이스 대상 스모크 (ADB 필요)
pnpm token-stats        # tools/list 토큰 측정
```

## 카테고리 (10)

`ANDROID_TOOLS=device,ui,apps,logcat` 같이 ENV로 일부만 로드.

| 카테고리 | 포함 도구 |
|---------|----------|
| `device` | list-devices, get-device-info |
| `apps` | install, uninstall, list, clear, permissions, analyze-app |
| `ui` | tap, swipe, screenshot, dump-hierarchy, find, scroll, annotated-screenshot |
| `logcat` | logcat capture, filter, crash extraction |
| `emulator` | AVD start/stop/snapshot |
| `files` | push, pull, exists, ls |
| `system` | settings, network, battery, display, locale, broadcast |
| `debug` | dumpsys, getprop, processes, doctor, mem/gfx/cpu info |
| `shell` | raw shell exec (별도 `ANDROID_MCP_ALLOW_SHELL` 게이트) |
| `meta` | search-tools (항상 활성) |

## MCP Resources (android://)

| URI | 설명 |
|-----|------|
| `android://devices` | 연결된 모든 디바이스 |
| `android://device/{serial}` | 디바이스 상세 (model/brand/version/display) |
| `android://app/{packageName}/activities` | 앱이 노출하는 액티비티 목록 (exported/launchable 플래그) |
| `android://device/{serial}/processes` | 디바이스 실행 중 프로세스 목록 |

## 설계 원칙

- **2-tier security gating**: 쓰기는 `ANDROID_MCP_ALLOW_WRITE=true`, shell exec는 `ANDROID_MCP_ALLOW_SHELL=true` 별도. 의도적 분리.
- **Pure ADB**: Appium/scrcpy/uiautomator2 같은 외부 자동화 프레임워크 의존성 0. ADB CLI만 사용.
- **shell 인자 escaping**: `shellEscape` 헬퍼로 single-quote 기반 안전 escape. injection 방지.
- **input validation**: setting key/value, package name, permission, component, action, broadcast extras 등 정규식 화이트리스트.
- **device path traversal 방지**: 절대 경로 강제 + `..` 차단 + shell metachar 차단.

## 최근 변경사항

- **v1.14.4** (2026-06-19): MCP tool annotations 적용 — `@us-all/mcp-toolkit ^1.3.0`의 `inferToolAnnotations`를 중앙 `tool()` 헬퍼에 추가, 전 도구에 readOnlyHint/destructiveHint/openWorldHint 자동 부여 (override 0). 비파괴 패치. 94/94 test.
- **v1.14.3** (2026-06-19): `take-screenshot` 이미지 변환 백엔드를 **jimp → sharp(0.35.1, libvips 8.18)** 교체. JPEG/PNG 인코딩·resize가 libvips 네이티브로 수행되어 더 빠르고 압축률이 좋음(스크린샷 base64 payload 절감). 도구 스키마·동작 동일(`format`/`quality`/`maxWidth`, png+no-resize fast-path 보존, `withoutEnlargement`로 no-upscale 유지). `jimp` dep 제거. **네이티브 dep 관리**: `pnpm.supportedArchitectures`(linux/darwin × x64/arm64 × glibc/musl)로 lockfile에 전 플랫폼 prebuilt 바이너리 기록 → Docker(`node:22-alpine`, musl) `--frozen-lockfile` 빌드가 `@img/sharp-linuxmusl-x64` 해석 가능. CI에 **`sharp-musl-smoke` 잡 추가**(Alpine 컨테이너에서 sharp 로드 + 스크린샷 테스트) — 발행 전 musl 검증 게이트. 94/94 test (darwin-arm64 로컬 검증, musl은 CI). 라이브 디바이스 미연결로 실 screencap 검증은 보류(transform 경로는 fixture PNG로 커버).
- **v1.13.5** (2026-05-17): 보안 — `pnpm.overrides`에 `fast-xml-builder ^1.1.7` 추가 (GHSA-5wm8-gmm8-39j9, high/CVSS 6.1: attribute injection bypass via quote characters). `fast-xml-parser ^5.7.x`의 transitive. 5/15 wave 미커버 신규 alert로 직접 패치. 코드 변경 0줄, 90/90 test.
- **v1.13.4** (2026-05-15): 보안 — `pnpm.overrides`에 fast-uri ^3.1.2 / hono ^4.12.18 / ip-address ^10.1.1 추가 (CVE-2026 transitive). 기존 vite/@hono/node-server overrides 보존. toolkit ^1.2.2 → ^1.2.3.
- **v1.13.3** (2026-05-15): `@us-all/mcp-toolkit ^1.2.2` 핀 업데이트 — 자동 cascade. 코드 변경 0줄.
- **v1.13.1** (2026-05-06): MCP Server Registry 발행 — `mcpName: "io.github.us-all/android"` 추가 + 루트 `server.json` (npm 패키지 + stdio transport + 7개 환경변수 메타데이터, ANDROID_MCP_ALLOW_SHELL/ALLOW_WRITE 게이트 명시). 코드 변경 0줄.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [us-all/android-mcp-server](https://github.com/us-all/android-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
