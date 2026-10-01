---
trigger: always_on
description: 이 문서는 AI 에이전트(Gemini, Claude 등)가 이 저장소(Repository)에서 작업할 때 지켜야 할 규칙과 프로젝트 컨텍스트를 정의합니다. `GEMINI.md`, `CLAUDE.md` 등에서 이 파일을 참조합니다.
---

# doksam-skills - AI Agent Guidelines

이 문서는 AI 에이전트(Gemini, Claude 등)가 이 저장소(Repository)에서 작업할 때 지켜야 할 규칙과 프로젝트 컨텍스트를 정의합니다. `GEMINI.md`, `CLAUDE.md` 등에서 이 파일을 참조합니다.

1장~3장은 스킬 개수와 무관한 **저장소 공통 규약**이고, 4장부터는 **스킬별 작업 지침**입니다.

## 1. 프로젝트 개요 (Project Context)

- **성격**: 이 저장소는 Agent Skill 모음(doksam-skills)입니다. `skills/` 아래 스킬 하나가 배포 단위이며, `install.sh` 가 전부를 Claude Code / Codex / Antigravity 경로에 심링크로 노출합니다.
- **소유 원칙**: **스킬 하나가 자기 자산을 전부 소유합니다.** 행동 계약(`SKILL.md`), 리소스, 스크립트, 테스트, 세 런타임 Agent Adapter 가 모두 `skills/<skill>/` 안에 있습니다. 저장소 루트에는 설치기와 공통 규약 검증만 둡니다. 특정 스킬에만 쓰이는 파일을 루트 `scripts/` 나 `tests/` 에 두지 않습니다.
- **기술 스택**: 스킬 정의는 Markdown, 템플릿은 HTML5 + Vanilla CSS, 검증/테스트 스크립트는 Python 3 stdlib 와 bash 입니다.
- **언어 정책 (한국어 우선)**: 이 저장소는 한국인 사용자를 위한 스킬 모음입니다. `SKILL.md` 본문·frontmatter `description`·Agent Adapter·문서·커밋/이슈/PR 본문은 **한국어**로 작성합니다. 단, 영어가 더 정확하거나 필수인 것은 영어를 유지합니다 — 코드 식별자·클래스명·명령어·파일/키 이름(`name`, `description` 키 등), 고유 기술용어(IA, Storyboard, SSOT, frontmatter 등), 외부 도구가 요구하는 형식. 트리거 매칭에 쓰이는 description 은 한국어 문장으로 쓰되 핵심 키워드는 한/영을 병기할 수 있습니다.

## 2. 스킬 레이아웃 규약

```text
skills/<skill>/
  SKILL.md            필수. frontmatter 는 name·description 두 개뿐이고 name 은 디렉터리명과 같다
  agents/             선택. 아래 파일명만 허용한다
    claude.md         Claude Code Agent Adapter
    codex.toml        Codex custom agent (name 은 스킬명의 - 를 _ 로 바꾼 것)
    antigravity.md    Gemini API Managed Agent 등록용 역할 정의 원본
    openai.yaml       Codex UI 용 스킬 메타데이터
  resources/          템플릿 등
  scripts/            이 스킬 전용 스크립트
  tests/              이 스킬 전용 테스트
```

루트 `.claude/agents/<skill>.md` 와 `.codex/agents/<skill_>.toml` 은 위 원본을 가리키는 **심링크**입니다. 원본을 그 자리에 직접 두지 않습니다.

Agent Adapter 에 스킬의 행동 규칙을 복제하지 않습니다. 공통 행동 계약은 `SKILL.md` 한 곳에서 관리하고 Adapter 는 역할·호출·완료 조건만 정의합니다.

**`claude.md` 와 `antigravity.md` 는 YAML frontmatter(`name`·`description`)가 필수입니다.** 특히 agy 는 frontmatter 가 없는 agent md 를 **오류도 경고도 없이 무시합니다** — 플러그인에는 파일이 복사돼 있는데 `agy agents` 에는 안 나오는 상태가 됩니다 (2026-08-11 agy 1.1.11 실측, 이슈 #110). 등록 여부는 파일 존재가 아니라 `agy agents` 출력으로 확인하세요.

**어댑터는 넷을 한 세트로 둡니다.** `agents/` 를 만든다는 것은 그 스킬을 위임형 에이전트로 노출하겠다는 결정이고, 그 결정이 런타임마다 다를 이유는 없습니다. 하나만 빠지면 그 런타임에서만 조용히 안 보이는 상태가 됩니다 — 실제로 `openai.yaml` 이 10개 중 1개에만 있었습니다 (이슈 #131). 어댑터를 **아예 두지 않는 것**은 별개의 선택이며, `session-recording` 이 그 경우입니다 (5장 참조).

이 규약 위반은 `tests/test_skill_layout.py` 가 스킬을 순회하며 잡습니다.

### 런타임별 발견 경로 (2026-08-11 실측)

| 런타임 | 스킬 | Agent Adapter | 확인 방법 |
|---|---|---|---|
| Claude Code | `~/.claude/skills/` | `~/.claude/agents/<skill>.md` | 세션 로드 |
| Codex | `~/.agents/skills/` · 프로젝트 `.agents/skills/` | `~/.codex/agents/<skill_>.toml` | `codex debug prompt-input` |
| Antigravity | `~/.gemini/config/skills/` | 플러그인 `agents/*.md` (`agy plugin install`) | `agy agents` |

Codex 는 `~/.codex/skills/` 도 읽지만 그쪽은 시스템 스킬 자리이므로 쓰지 않습니다. Antigravity 는 `~/.gemini/config/agents/` 를 탐색하지 않으므로 에이전트는 반드시 플러그인으로 등록합니다.

세 런타임을 한 번에 대조하려면 `./install.sh --verify`(에이전트까지 보려면 `--with-agent` 를 함께) 를 씁니다. 위 "확인 방법" 열의 명령을 실행해 그 출력과 저장소의 스킬 목록을 맞춰 보고, 하나라도 없으면 exit 1 합니다. **파일 존재를 등록의 증거로 삼지 마세요** — 이슈 #110 이 정확히 파일은 제자리에 있는데 런타임이 조용히 무시한 경우였습니다. 질의할 CLI 가 없는 자리(Claude Code 의 스킬, Codex 의 커스텀 에이전트)는 경로 존재로 판정하며, 출력에 근거가 함께 찍힙니다.

## 3. 저장소 공통 작업

### 새 스킬 추가

```bash
./scripts/new_skill.sh my-skill "한 줄 설명"
```

뼈대와 세 런타임 Adapter, 루트 심링크까지 만들어집니다. `install.sh` 와 루트 `tests/` 는 스킬을 순회하므로 스킬을 추가할 때 손댈 필요가 없습니다. 반대로 **거기에 스킬명을 하드코딩하면 테스트가 실패합니다.**

### 테스트

Python 은 **stdlib 만** 씁니다. 이 환경의 Homebrew Python 3.14 는 외부 라이브러리 import 가 깨져 있습니다.

```bash
# 루트 규약 + 모든 스킬 테스트 + 설치기 테스트를 한 번에
./scripts/run_tests.sh

# 개별 실행
python3 -m unittest discover -s tests -t tests -v
python3 -m unittest discover -s skills/<skill>/tests -t skills/<skill>/tests -v
./tests/test_install.sh
```

이 저장소는 생성 산출물(HTML)을 커밋하지 않습니다. 산출물 검증은 사용자가 생성한 파일을 인자로 넘겨 수행합니다.

## 4. 스킬별 작업 지침: mobile-web-planner

사용자의 요청(예: "쇼핑몰 기획해줘")에 따라 IA 와 화면 설계서(HTML 기반 스토리보드)를 생성하는 스킬입니다.

- `skills/mobile-web-planner/SKILL.md`: 기획자 페르소나, 워크플로우, 클래스 Quick Reference, 마크업 예시가 정의된 핵심 파일.
- `skills/mobile-web-planner/resources/template.html`: 기획서 결과물의 HTML/CSS 스켈레톤. **CSS 클래스의 유일한 정의처**.
- `skills/mobile-web-planner/scripts/scaffold.py`: 템플릿 head 를 복사한 빈 산출물 뼈대 생성기. 에이전트가 430줄 CSS 를 손으로 옮겨 적지 않게 한다.
- `skills/mobile-web-planner/scripts/validate_storyboard.py`: 산출물 구조 검증기.
- `skills/mobile-web-planner/scripts/check_badge_overflow.py`: 배지 좌표가 목업 밖으로 나가는지 점검하는 보조 스크립트.
- `skills/mobile-web-planner/scripts/check_badge_alignment.py`: 배지 겹침과 라벨-좌표 순서 역전을 점검하는 보조 스크립트.
- `skills/mobile-web-planner/scripts/apply_badge_audit.py`: badge-audit 실측 JSON 을 받아 인라인 `top` 을 일괄 반영하고 정적 검증기를 재실행하는 스크립트.
- `skills/mobile-web-planner/resources/badge-audit.js`: 브라우저에서 실행해 배지가 실제로 무엇을 가리키는지 실측하는 스니펫. 목업이 0.9배로 축소되어 인라인 `top` 만으로는 정렬을 알 수 없다.
- `skills/mobile-web-planner/scripts/export_deck.py`: 산출물 HTML 에서 PDF 와 PPTX 를 함께 만드는 내보내기 스크립트.
- `skills/mobile-web-planner/scripts/check_layout_runtime.py`: Chrome headless 로 렌더해 레이아웃 회귀(슬라이드 overflow · 배지 이탈/겹침 · 설명 패널 잘림)를 잡는 검사기.
- `skills/mobile-web-planner/resources/layout-probe.js`: 위 검사기가 주입하는 좌표 수집 스니펫. **판정은 하지 않는다** — 임계값은 파이썬 한 곳에만 둔다.

### 클래스 계약 (가장 중요)

`resources/template.html` 에 CSS 로 정의된 클래스만 사용한다. 유일한 예외는 `mermaid` (mermaid.js 가 렌더하므로 CSS 불필요).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeeYudok/doksam-skills](https://github.com/LeeYudok/doksam-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
