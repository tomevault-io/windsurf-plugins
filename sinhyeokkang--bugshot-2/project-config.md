---
trigger: always_on
description: > **이 파일은 자동 생성물이다.** 원본은 [CLAUDE.md](./CLAUDE.md)이고 `pnpm sync:agents`가 아래 본문을 그대로 복제한다.
---

# AGENTS.md

> **이 파일은 자동 생성물이다.** 원본은 [CLAUDE.md](./CLAUDE.md)이고 `pnpm sync:agents`가 아래 본문을 그대로 복제한다.
> 고칠 내용이 있으면 **CLAUDE.md를 고치고** `pnpm sync:agents`를 돌려라 — 이 파일을 직접 편집하면 다음 sync에서 덮어써진다.
> 같은 규칙이 `.agents/skills/`(= `.claude/commands/` 미러)에도 적용된다. 이 프리앰블만 예외로 `.agents/PREAMBLE.md`에서 손으로 관리한다.
> 본문이 `CLAUDE.md`·`.claude/commands/`를 가리키면 **그 원본 경로가 맞다** — 치환 없이 복제하므로 그대로 읽으면 된다.

## Codex 런타임 차이 (이 프리앰블 전용)

Claude Code에만 있는 자동 안전망이 Codex 세션에는 없다. 아래는 **직접** 챙긴다.

- **스킬 호출 매핑** — 본문이 `/<name>`으로 부르는 스킬은 Codex에선 `source-command-<name>` 스킬로 로드한다.
- **미제공 스킬 (역할 분담)** — `/push`·`/merge`·`/deploy`·`/sync`는 미러하지 않는다. **Codex는 작업 → 커밋까지, 원격으로 나가는 건 Claude Code**가 단일 창구로 맡는다 — 릴리스 파이프라인 게이트(`/merge`의 원격 CI 결론 조회·버전 bump, `/deploy`의 tag)가 두 창구에서 경쟁하면 깨지기 때문이다. 이 스킬들이 필요해지면 사용자에게 Claude Code 세션에서 실행하라고 안내하고 멈춘다.
- **`/ship`은 11단계까지** — `source-command-ship`은 미러돼 있고 `/tdd`~커밋 #4(11단계)까지 전부 돈다. 12·13단계(`/push`·`/build`)는 **수행하지 않고** "push 대기 — Claude Code에서 `/push` 실행"을 리포트에 남기고 종료한다. e2e 차단 게이트는 push 이후 CI(`e2e-gate`)가 맡으므로 Codex 쪽에서 미리 돌려둘 게이트가 없다 — 필요하면 `/e2e-run`을 수동 호출할 수는 있지만 게이트는 아니다. 상세는 스킬 본문의 "push 권한 / 런타임별 종착점".
- **i18n ko/en 대칭 훅 없음** — Claude Code는 `.claude/settings.json`의 PostToolUse 훅이 `src/i18n/` 편집 시 대칭 검사를 자동 실행해 불일치를 차단한다. Codex엔 이 훅이 없으니 `src/i18n/` 또는 `src/log-viewer/i18n.ts`(복제 사전)를 건드렸으면 손으로 돌린다:
  `pnpm test --run src/i18n/__tests__/locales.test.ts src/log-viewer/__tests__/i18n.test.ts`
- **미러 sync 훅 없음** — Claude Code는 `CLAUDE.md`·`.claude/commands/*.md` 편집 시 훅이 `sync:agents`를 자동 실행한다. Codex엔 없다. 애초에 **Codex는 원본을 편집하지 않는 게 규칙**이고, 부득이 고쳤으면 `pnpm sync:agents`를 직접 돌려 미러를 함께 커밋한다.
- **개인 메모리 없음** — 본문 말미의 `~/.claude/projects/.../memory/`는 Claude Code 전용 저장소다. Codex는 이 경로를 읽지 않는다.
- **커밋 트레일러** — Codex 세션에서 만든 커밋은 마지막 줄에 `Co-Authored-By: Codex <noreply@openai.com>`를 붙인다(Claude Code의 `Co-Authored-By: Claude ...`와 대칭 — 어느 에이전트가 만든 커밋인지 히스토리에서 구분되게). 커밋 메시지의 scope는 **바뀐 파일 기준**이라 그대로다 — CLAUDE.md를 고쳤으면 Codex가 커밋해도 `docs(CLAUDE): ...`다.

---

## 응답 스타일 (이 문서의 다른 모든 규칙보다 우선)

**한국어로, 간결하게.** 위반 시 답변을 다시 쓴다. 아래는 취향이 아니라 판정 기준이다.

- **첫 문장이 결론**: 서두·예고 금지 — "~해보겠습니다", "좋은 질문입니다", "확인해보니 다음과 같습니다" 류로 시작하지 않는다. 바로 답/결과부터.
- **꾸밈말 금지**: "완벽합니다", "훌륭한", "핵심적인", "말씀하신 대로" 같은 평가·동조 표현을 빼도 정보가 안 줄면 뺀다.
- **재진술 금지**: 방금 보여준 diff·명령 출력·파일 내용을 산문으로 다시 설명하지 않는다. 코드가 말하는 건 코드가 말하게 둔다.
- **길이 상한**: 단순 질문·확인 → 3줄 이내. 작업 완료 보고엔 줄 수 상한이 없다 — 필요한 정보를 줄이면서까지 짧게 만들지 않는다. 대신 위의 재진술·꾸밈말 금지로 군더더기만 덜어낸다.
- **미완·실패를 먼저**: 못 한 것·실패한 테스트·건너뛴 범위를 성공 요약보다 앞에 쓴다.
- **선택지 나열 금지**: 추천 하나를 고르고 그 이유 한 줄. 사용자 결정이 필요한 지점(작업 원칙의 "가정을 명시")만 예외.
- **예외**: 코드·커밋 메시지·PR title/body·GitHub Release notes는 영문(코드 컨벤션 참조). 문서(`docs/`·`guide/ko`)의 본문 톤은 각 문서 규칙을 따른다 — 이 섹션은 **대화 응답**에만 적용된다.

강제 장치는 2단이다: 이 섹션(두 런타임 공통 — Codex는 `AGENTS.md` 미러로 받는다)과, `.claude/settings.json`의 `UserPromptSubmit` 훅이 매 턴 같은 규칙 요약을 컨텍스트에 재주입하는 것(긴 세션에서 문서 앞쪽이 희석되는 걸 막는다). **훅은 Claude Code 전용이라 Codex 세션에선 이 섹션만 남는다.**

bugshot-2: Chrome MV3 Side Panel 버그 리포팅 확장. 웹 페이지의 버그를 요소 스타일 편집(before/after 비교)·스크린샷(영역/화면/페이지 전체/요소, 어노테이션)·영상 녹화(탭/화면, 30초 리플레이) 중 원하는 방식으로 캡처하고, 콘솔·네트워크·사용자 액션 로그를 자동 수집한다. 이렇게 만든 리포트를 Jira·GitHub·Linear·Notion·GitLab·Asana·ClickUp 이슈로 등록하거나 Slack 채널·DM으로 공유한다.

## 코어 밸류: Privacy (클라이언트 온리)

**BugShot의 코어 밸류이자 경쟁 우위 축.** 버그 리포트에는 프로덕션 세션의 가장 민감한 단면이 담긴다 — 스크린샷 속 고객 데이터, network 로그의 토큰과 페이로드, console에 찍힌 내부 식별자. 그래서 BugShot은 그걸 **가져가지 않는 쪽**을 택했다. 캡처 데이터(스크린샷·영상·console/network/action 로그·CSS diff·리포트 본문)는 BugShot 서버를 거치지 않고 **사용자 브라우저 → 사용자의 이슈 트래커/Slack으로 직행**한다. 사용자가 AI 기능을 실행하면 필요한 프롬프트·로그 요약·캡처/인라인 이미지는 **사용자가 선택한 LLM endpoint로 직접 전송**된다. BugShot 서버를 지나는 건 **OAuth 토큰 교환 프록시**(`VITE_OAUTH_PROXY_URL`)뿐이고, 익명 PostHog 집계는 설정된 분석 host로 직접 전송된다(`in.bug-shot.com` — BugShot 도메인이지만 CNAME이 PostHog Cloud를 직접 가리키는 리버스 프록시라 BugShot 서버는 이 경로에도 없다) — 어느 경로에도 캡처 데이터가 BugShot 서버를 거치지 않는다.

이건 정책이 아니라 구조다. "안 보겠다"는 약속이 아니라 **물리적으로 볼 수 없게** 만들어둔 것 — 규제·보안 민감 조직에게 약속과 구조의 차이는 검증 가능성의 차이다. 호스팅 저장소·워크스페이스를 두는 SaaS 모델은 필연적으로 이 구조를 깬다. 편의를 좇아 무서버·데이터 직행을 포기하는 건 기능 추가가 아니라 **제품 정체성 변경**으로 취급한다. 절대적 제약은 아니지만, 새 기능이 캡처 데이터를 외부 서버로 보내야 하면 이 밸류와 충돌하는지 먼저 따진다.

## 작업 원칙

- **가정을 명시**: 해석이 여러 개면 조용히 하나 고르지 말고 선택지를 제시. 불확실하면 물어라.
- **더 단순한 방법이 있으면 제안**: 200줄을 50줄로 줄일 수 있으면 줄여라. 요청하지 않은 유연성·설정 가능성·추상화 추가 금지.
- **외과적 변경**: 요청과 직접 관련 없는 인접 코드 개선·리팩터 금지. 기존 스타일 따르기. 기존 dead code는 언급만 하고 삭제하지 않는다 — 내 변경이 만든 고아만 제거.
- **검증 가능한 목표로 전환**: "버그 고쳐" → "재현 테스트 작성 후 통과시켜". 멀티스텝 작업은 단계별 검증 체크를 포함한 플랜을 먼저 제시.
- **테스트 우선**: 신규 인터페이스(함수·헬퍼·어댑터) 추가 시 테스트를 먼저 작성하고 구현한다. 기존 로직 변경 시에도 관련 순수 함수의 단위 테스트를 작성/갱신하고 `pnpm test` 통과를 확인한 뒤 작업을 마친다. 테스트 없이 코드만 변경하지 않는다.

## 스택

- React 18 + TypeScript + Vite (via `@crxjs/vite-plugin`)
- Tailwind CSS v3 + shadcn/ui + `@tailwindcss/container-queries` (디자인 시스템·색상 토큰·UI 컨벤션 상세는 [DESIGN.md](./docs/DESIGN.md))
- Zustand + `chrome.storage` (session/local 혼용)
- Tiptap (ProseMirror) WYSIWYG 에디터 + `tiptap-markdown` 양방향 변환 + `markdown-it` (HTML/ADF/Notion 변환용 파서)
- 스크린샷 어노테이션: Konva + react-konva 캔버스(사이드패널 lazy 청크). 도형은 natural 좌표, 표시 배율은 CSS transform. 줌·팬 계산은 `sidepanel/components/annotation/viewport.ts` 순수 함수 단일 출처(fit-width 진입 / 전체 조망 / 사용자 배율을 `ZoomLevel`로 **의도 저장**). 드래그는 **window 리스너**로 구동 — pointer capture를 쓰지 않는다(상세·함정은 [ARCHITECTURE.md](./docs/ARCHITECTURE.md))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SinhyeokKang/bugshot-2](https://github.com/SinhyeokKang/bugshot-2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
