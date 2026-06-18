---
trigger: always_on
description: Signal-Driven Navigator for indie hackers. Guides users through problem discovery, customer interviews, spec writing, and tool comparison to reach their first paid signal in 30 days. Use when the user asks to start, discover problems, interview customers, write specs, compare tools, or check progress.
---


# agnt

30일 안에 첫 유료 시그널을 만드는 Signal-Driven Navigator.

## When To Use

다음 요청에서 이 스킬을 사용합니다.

- "agnt 시작", "프로젝트 시작", "시작하기"
- "진단", "audit", "프로젝트 진단", "매출 준비도", "Revenue Readiness"
- "다음에 뭐 해야 해?", "다음 행동", "추천"
- "문제 찾기", "ICP 정의", "아이디어 검증"
- "ICP 문서", "ICP 구체화", "타겟 고객 문서", "deep-interview로 ICP"
- "디자인 시스템", "브랜드 가이드", "design-system", "docs/design-system.md"
- "고객 인터뷰", "Mom Test", "인터뷰 가이드"
- "SPEC 작성", "제품 스펙"
- "도구 비교", "결제 솔루션", "분석 도구"
- "진행 상태", "대시보드"
- "MVP 범위", "빌드 가이드", "뭘 먼저 만들어야 해?"
- "랜딩 전략", "랜딩에 뭘 써야 해?", "랜딩페이지 가이드"
- "채널 추천", "어디에 올려야 해?", "첫 포스트"
- "경쟁 분석", "비슷한 제품이 있어?", "차별화"
- "콘텐츠 전략", "뭘 올려야 해?", "첫 포스트"
- "오퍼 설계", "얼마에 팔아야 해?", "가격 설정"
- "론칭 계획", "론칭 어떻게 해?", "7일 계획"
- "성과 분석", "잘 되고 있는 거야?", "전환율 판정"
- "회고", "뭘 배웠지?", "다음 루프"
- "연결", "MCP 연결", "동기화", "agentic30 연결"
- "SEO 점검", "SEO 체크", "사이트 점검"
- "사업자 등록", "사업자 판단", "사업자 필요"
- "분석 세팅", "PostHog 설치", "이벤트 추적"
- "런칭 카피", "채널 카피", "카피 생성"
- "광고 소재", "광고 카피", "Meta 광고", "Google 광고"
- "Meta 광고 세팅", "Threads 광고 세팅", "Ads Manager 세팅", "광고 세트 타게팅"
- "수익 모델", "어떻게 돈 벌어", "첫 매출"
- "구독 전략", "앱 구독", "hard paywall", "freemium", "trial", "연간 플랜", "intro offer"
- Codex 명시형 호출: `$agnt-start`, `$agnt-audit`, `$agnt-next`, `$agnt-discover`, `$agnt-icp`, `$agnt-design-system`, `$agnt-interview`, `$agnt-compete`, `$agnt-spec`, `$agnt-build`, `$agnt-landing`, `$agnt-channel`, `$agnt-content`, `$agnt-offer`, `$agnt-launch`, `$agnt-analyze`, `$agnt-retro`, `$agnt-tools`, `$agnt-status`, `$agnt-connect`, `$agnt-seo-audit`, `$agnt-biz-setup`, `$agnt-analytics-setup`, `$agnt-launch-copy`, `$agnt-ad-creative`, `$agnt-meta-ads-setup`, `$agnt-revenue`, `$agnt-subscription`

## Runtime Setup

### 0) 에이전트 환경 분기

실행 환경을 먼저 확인합니다.

- Claude Code 환경: `/mcp` 명령이 사용 가능하거나 사용자가 Claude Code를 사용 중임
- Codex 환경: `codex` CLI 사용 중임

이후 MCP 연결 절차를 환경별로 따릅니다.

### 1) MCP 연결 확인 (환경별)

#### A. Claude Code

`agentic30` MCP 서버가 없으면 아래 순서로 안내합니다.

1. Claude Code 실행
2. 채팅 입력창에 `/mcp` 입력
3. 목록에서 `agentic30` 또는 `plugin:agnt:agentic30` 선택
4. `Authenticate` 선택 후 브라우저에서 Google 로그인
5. `/mcp`에서 connected 상태 확인

서버가 목록에 없으면 프로젝트 `.mcp.json`에 아래 항목을 추가합니다.

```json
{
  "mcpServers": {
    "agentic30": {
      "type": "http",
      "url": "https://mcp.agentic30.app/mcp"
    }
  }
}
```

그 뒤 Claude Code를 재시작하고 `/mcp` 인증을 다시 진행합니다.

#### B. Codex

`agentic30` MCP 서버가 없으면 아래 순서로 안내합니다.

```bash
codex mcp add agentic30 --url https://mcp.agentic30.app/mcp
codex mcp login agentic30
codex mcp list
```

이미 등록되어 있으면 `codex mcp login agentic30`만 다시 수행하면 됩니다.

### 2) 상태 파일 경로(AGNT_DIR)

아래 순서로 state 경로를 탐색합니다.

1. `.claude/agnt/state.json`
2. `~/.claude/agnt/state.json`
3. `.codex/agnt/state.json`
4. `~/.codex/agnt/state.json`
5. 둘 다 없으면 `.claude/agnt/state.json` 생성

### 3) references 경로(REFS_DIR)

`navigator-engine.md` 존재 여부로 탐색합니다.

1. `references/` (현재 스킬 디렉토리 기준)
2. `{AGNT_DIR}/references`
3. `~/.claude/plugins/marketplaces/agentic30/references`
4. `.agents/skills/agnt/references`
5. `~/.codex/skills/agnt/references`

## Skill Mapping

사용자 의도를 아래 스킬로 매핑해 실행합니다.

- 시작/온보딩: `skills/start/SKILL.md`
- Revenue Readiness Audit: `skills/audit/SKILL.md`
- 다음 행동: `skills/next/SKILL.md`
- 문제 발견: `skills/discover/SKILL.md`
- ICP 정의/구체화: `skills/icp/SKILL.md`
- 디자인 시스템: `skills/design-system/SKILL.md`
- 고객 인터뷰: `skills/interview/SKILL.md`
- 경쟁 분석: `skills/compete/SKILL.md`
- SPEC 작성: `skills/spec/SKILL.md`
- MVP 빌드: `skills/build/SKILL.md`
- 랜딩 전략: `skills/landing/SKILL.md`
- 채널 활성화: `skills/channel/SKILL.md`
- 콘텐츠 전략: `skills/content/SKILL.md`
- 오퍼 설계: `skills/offer/SKILL.md`
- 론칭 계획: `skills/launch/SKILL.md`
- 성과 분석: `skills/analyze/SKILL.md`
- 회고: `skills/retro/SKILL.md`
- 도구 비교: `skills/tools/SKILL.md`
- 상태 확인: `skills/status/SKILL.md`
- Agentic30 연결: `skills/connect/SKILL.md` *(disable-model-invocation)*
- SEO 점검: `skills/seo-audit/SKILL.md`
- 사업자 등록 판단: `skills/biz-setup/SKILL.md`
- 분석 환경 세팅: `skills/analytics-setup/SKILL.md`
- 런칭 카피: `skills/launch-copy/SKILL.md`
- 광고 소재: `skills/ad-creative/SKILL.md`
- Meta 광고 세팅: `skills/meta-ads-setup/SKILL.md`
- 수익 모델: `skills/revenue/SKILL.md`
- 구독 전략: `skills/subscription/SKILL.md`

각 스킬의 SKILL.md를 source of truth로 사용합니다.

### Codex Command Style (`$agnt-*`)

Codex에서는 아래 명령을 canonical로 사용합니다.

- `$agnt-start` → `skills/start/SKILL.md`
- `$agnt-audit` → `skills/audit/SKILL.md`
- `$agnt-next` → `skills/next/SKILL.md`
- `$agnt-discover` → `skills/discover/SKILL.md`
- `$agnt-icp` → `skills/icp/SKILL.md`
- `$agnt-design-system` → `skills/design-system/SKILL.md`
- `$agnt-interview` → `skills/interview/SKILL.md`
- `$agnt-compete` → `skills/compete/SKILL.md`
- `$agnt-spec` → `skills/spec/SKILL.md`
- `$agnt-build` → `skills/build/SKILL.md`
- `$agnt-landing` → `skills/landing/SKILL.md`
- `$agnt-channel` → `skills/channel/SKILL.md`
- `$agnt-content` → `skills/content/SKILL.md`
- `$agnt-offer` → `skills/offer/SKILL.md`
- `$agnt-launch` → `skills/launch/SKILL.md`
- `$agnt-analyze` → `skills/analyze/SKILL.md`
- `$agnt-retro` → `skills/retro/SKILL.md`
- `$agnt-tools` → `skills/tools/SKILL.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [october-academy/agnt](https://github.com/october-academy/agnt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
