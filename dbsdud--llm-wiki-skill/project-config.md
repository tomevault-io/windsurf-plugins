---
trigger: always_on
description: |
---


# /wiki

LLM Wiki maintainer for `~/vaults/`. Implements the Karpathy LLM Wiki pattern:

- **raw/** — immutable source materials
- **wiki/** — LLM-owned curated knowledge (concepts, entities, source summaries, comparisons)
- **CLAUDE.md** — schema and workflow rules

The vault is **consumer-only**. `graphify` is never run inside the vault.
Run `graphify` in each project; this skill brings the output into `raw/repos/` and synthesizes a wiki summary.

Reference: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

## Usage

```
/wiki                                       # show this help
/wiki init [path]                           # create vault structure (default ~/vaults)
/wiki ingest-project <project-path> [name]  # graphify project → raw/repos → wiki/sources
/wiki ingest-url <url>                      # fetch URL into raw/articles, then summarize
/wiki ingest-file <path>                    # copy local file into raw/, then summarize
/wiki query "<question>"                    # answer using wiki, file synthesis back
/wiki lint                                  # check orphans, frontmatter, sources, staleness
/wiki overview                              # rewrite wiki/overview.md from current pages
```

## Vault resolution

- Default root: `~/vaults`. Override with `WIKI_VAULT` env var or `init [path]`.
- For every non-`init` command: verify `$VAULT/CLAUDE.md` exists. If not, tell the user to run `/wiki init` first and stop.
- Conventions live in `$VAULT/CLAUDE.md`. Read it before any non-trivial operation — it is the source of truth for page types, frontmatter, and absolute rules.

## What you must do when invoked

If the user invoked `/wiki` or `/wiki -h` with no subcommand, print the Usage block verbatim and stop.

Otherwise, bind:

```bash
VAULT="${WIKI_VAULT:-$HOME/vaults}"
TODAY=$(date +%Y-%m-%d)
```

Then dispatch on the subcommand below.

## Common helpers

**Append a log entry** (always at the end of the operation, never mid-flight):

```bash
printf '\n## [%s] %s | %s\n' "$TODAY" "$OP" "$MSG" >> "$VAULT/wiki/log.md"
```

`$OP` is one of `init | sync | ingest | query | lint | synthesis`.
**Never edit existing log lines. Append only.**

**Read vault schema before non-init operations:** read `$VAULT/CLAUDE.md`. Defer to its rules over anything embedded in this skill.

---

### /wiki init [path]

Bootstrap a fresh LLM Wiki vault. Default path is `~/vaults`; if `[path]` is given, use it.

1. Verify state:
   ```bash
   TARGET="${1:-$HOME/vaults}"
   if [ -f "$TARGET/CLAUDE.md" ]; then
       echo "$TARGET/CLAUDE.md already exists. Refusing to overwrite."
       echo "Remove it first or pick another path."
       exit 1
   fi
   mkdir -p "$TARGET"
   ```

2. Create directory structure:
   ```bash
   mkdir -p "$TARGET"/raw/{articles,papers,repos,data,images,assets}
   mkdir -p "$TARGET"/wiki/{concepts,entities,sources,comparisons}
   ```

3. Write `$TARGET/CLAUDE.md` with the schema below (use the Write tool, do NOT cat-heredoc — the content has many backticks):

   ```markdown
   # LLM Wiki Schema (Karpathy Pattern)

   이 vault는 Andrej Karpathy의 LLM Wiki 패턴을 따른다.
   참조: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

   ## 3-Layer 아키텍처

   1. **Raw Sources (`raw/`)** — immutable. 외부에서 가져온 원본 자료. 절대 수정하지 말 것
   2. **Wiki (`wiki/`)** — LLM이 소유. raw/를 컴파일한 결과물
   3. **Schema (`CLAUDE.md`)** — 이 파일. 에이전트를 wiki maintainer로 변환하는 규칙

   ## 디렉토리 레이아웃

   - `raw/articles/` — 블로그·뉴스·클리핑 (`YYYY-MM-DD-slug.md`)
   - `raw/papers/` — 논문 PDF
   - `raw/repos/` — 외부 레포 README, graphify export 산출물
   - `raw/data/` — 벤치마크, CSV, JSON
   - `raw/images/` — 다이어그램, 스크린샷
   - `raw/assets/` — Obsidian 첨부파일 기본 경로
   - `wiki/index.md` — 전체 카탈로그 (카테고리별)
   - `wiki/log.md` — append-only 활동 로그
   - `wiki/overview.md` — 지식 베이스 전체 합성 요약
   - `wiki/concepts/` — 개념·이론·패턴
   - `wiki/entities/` — 인물·조직·제품
   - `wiki/sources/` — raw/ 항목별 요약 (1:1 매핑)
   - `wiki/comparisons/` — 비교·대조 페이지

   ## 페이지 프론트매터

   모든 wiki 페이지는 YAML frontmatter 필수:

   - `title` — Human-readable 제목
   - `type` — `concept | entity | source-summary | comparison`
   - `sources` — 참조한 raw/ 파일 경로 배열
   - `related` — 링크된 다른 wiki 페이지
   - `created` — `YYYY-MM-DD`
   - `updated` — `YYYY-MM-DD`
   - `confidence` — `high | medium | low`

   ## 페이지 유형별 규칙

   - **concept (`wiki/concepts/`)** — 개념·알고리즘·패턴. "무엇이고 왜 중요한가" 중심
   - **entity (`wiki/entities/`)** — 인물·조직·제품·모델. 핵심 사실 + 관련 작업 + 관계
   - **source-summary (`wiki/sources/`)** — raw/ 단일 항목의 압축 요약 (1:1 매핑)
   - **comparison (`wiki/comparisons/`)** — N개 항목을 같은 축에서 비교. 표 권장

   ## 특수 파일

   - `wiki/index.md` — 카테고리별 카탈로그. 신규·삭제 시 반드시 갱신
   - `wiki/log.md` — append-only. 절대 과거 항목 수정 금지. 포맷: `## [YYYY-MM-DD] <operation> | <description>` (operation: `init | sync | ingest | query | lint | synthesis`)
   - `wiki/overview.md` — 전체 high-level 합성. 주기적으로 다시 쓰기

   ## 워크플로

   주요 워크플로는 `/wiki` skill을 통해 실행:

   - `/wiki ingest-project <path>` — graphify 산출물을 raw/repos/ 로 가져와 wiki/sources/ 로 요약
   - `/wiki ingest-url <url>` — URL을 raw/articles/ 로 저장하고 요약
   - `/wiki ingest-file <path>` — 로컬 파일을 raw/ 적절한 위치에 저장하고 요약
   - `/wiki query "<question>"` — wiki를 통해 답변. 새 합성은 페이지로 저장
   - `/wiki lint` — 고아·frontmatter·sources 경로·stale 점검
   - `/wiki overview` — overview.md 재합성

   ## graphify 연동

   이 vault는 **consumer-only**다. graphify는 vault 안에서 절대 실행하지 않는다.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbsdud/llm-wiki-skill](https://github.com/dbsdud/llm-wiki-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
