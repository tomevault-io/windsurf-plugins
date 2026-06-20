---
trigger: always_on
description: name: mv-clip-factory
---

---
name: mv-clip-factory
version: 3.0.0
description: "음악 + 레퍼런스 영상 → 가사 작성 → 스토리보드 → 웹툰 이미지 생성 → AI 영상 클립 생성 → 최종 MV 편집까지의 전체 뮤직비디오 제작 파이프라인. 실행 스크립트 포함 (Flux 2 Pro, Qwen, Wan, Veo 3). 뮤직비디오, MV, 영상제작, AI영상, 클립생성, 스토리보드, Veo, 이미지투비디오 시 자동 활성화."
preamble-tier: 2
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - WebSearch
  - WebFetch
hooks:
  post-phase4: "이미지 생성 완료 후 /qa로 품질 검수 실행 가능"
  post-phase7: "최종 MV 완성 후 /ship으로 배포 실행 가능"
shared-modules:
  # 점진적 분리 예정 (하이브리드 아키텍처):
  # /ai-image    — Imagen 4.0 + fal.ai Flux Schnell 통합 인터페이스
  # /ai-video    — Veo 3.0 + fal.ai Wan2.2 폴백 체인
  # /ffmpeg-tools — 프레임 추출, 클립 concat, 자막 삽입
  # /cost-tracker — 프로젝트별 비용 대시보드
scripts:
  images:
    - scripts/images/generate_flux2_pro_images.py    # Flux 2 Pro (fal.ai)
    - scripts/images/generate_qwen_images.py         # Qwen Image
    - scripts/images/generate_wan_images.py          # Wan 2.2 Image
  clips:
    - scripts/clips/generate_wan_clips.py            # Wan 2.2 (폴백용)
    - scripts/clips/veo3/veo_generator.py            # Veo 3 단일 호출
    - scripts/clips/veo3/generate_clip.py            # Veo 3 클립 생성
    - scripts/clips/veo3/batch_generate.py           # Veo 3 배치 실행
  build:
    - scripts/build_mv.sh                            # 최종 MV 컴포지션
changelog:
  v3.0.0: "실행 스크립트 통합 (Flux 2 Pro, Qwen, Wan, Veo 3). Ragnarok2 MV 실전 코드 흡수. GitHub 공개 배포 (2026-04-25)."
  v2.0.0: "gstack 통합 포인트 추가, 공유 모듈 마커 삽입, 허브 서버 위자드 문서 보강"
  v1.0.0: "초기 릴리즈 (왕사님 MV 시행착오 기반)"
---

# MV Clip Factory — AI 뮤직비디오 제작 파이프라인

## 핵심 원칙 3가지 (왕사님 MV 시행착오에서 도출)

> 이 원칙을 지키면 **72시간 → 12시간**으로 단축, 비용 **$34 → $20**으로 절감 가능.

### 1. 음원 먼저, 씬 나중
- 음원의 에너지 프로파일(RMS, 비트)이 씬 분할의 근거
- 음원 없이 씬을 나누면 반드시 재조정이 발생 (v4→v5에서 11씬→13씬 재작업)
- **Phase 1에서 음원 확보 → Phase 3에서 librosa 분석 후 씬 분할**

### 2. 한 번에 확정, 한 번에 생성
- 스토리보드 확정 → 필요 이미지 목록 확정 → prompts.json 완성 → 한 번에 일괄 생성
- 분할 생성(57장→9장→3장)은 3일에 걸쳐 비효율 발생
- **`--dry-run`으로 목록 확인 → 한 번에 전량 생성 → `--missing`으로 실패분만 재생성**

### 3. 1장 테스트, N장 실행
- 도구 선택 전 대표 이미지 1장으로 Veo/Kling/fal.ai 비교 (30분)
- Kling AI 5건 생성 후 포기 = 2시간 낭비 (왕사님 실제 사례)
- **동일 이미지 1장 → 3종 비교 → 최적 도구 결정 → 본 작업 배치 실행**

---

## 전체 워크플로우

```
Phase 0: 프로젝트 초기화 & 허브 등록         (10분)
           ↓
Phase 1: 레퍼런스 수집 & ⚡음원 확보⚡       (2시간)
           ↓
Phase 2: 가사 작성 & 런닝타임 검증           (1시간)
           ↓
Phase 3: 음원 분석 → 스토리보드 설계          (2시간)
           ↓
Phase 3.5: ⚡1장 도구 비교 테스트⚡           (30분)
           ↓
Phase 4: 이미지 ⚡일괄⚡ 생성                 (1시간)
           ↓
Phase 5: AI 영상 클립 ⚡배치⚡ 생성           (3시간)
           ↓
Phase 6: 나레이션/자막 (선택)                 (1시간)
           ↓
Phase 7: ffmpeg 편집 & 최종 출력              (1시간)
           ↓
Phase 8: 검수 & 버전 태깅 & 기록             (30분)
```

**⚡ = 왕사님 시행착오에서 추가/변경된 단계**

### 두 가지 시작 방법

| 방법 | 적합한 경우 | 절차 |
|------|-----------|------|
| **웹 위자드** | YouTube URL에서 바로 시작 | `http://localhost:8787/new/` → 5단계 자동 |
| **수동 초기화** | 세밀한 제어가 필요할 때 | Phase 0 원커맨드 실행 → Phase 1부터 순차 |

---

## Phase 0: 프로젝트 초기화 & 허브 등록

새 MV 제작 시 반드시 이 단계를 먼저 수행한다. 허브 서버(`hub_server.py`)가 자동으로 인식하는 구조로 폴더를 생성한다.

### 0-A: 프로젝트 폴더 생성

허브 서버의 루트 디렉토리(`video_creator/`) 아래에 프로젝트 폴더를 만든다.

```bash
PROJECT_ROOT="/path/to/your/video_creator"
PROJECT_NAME="my-new-mv"  # 영문 슬러그 (URL에 사용됨)

mkdir -p "${PROJECT_ROOT}/${PROJECT_NAME}"/{images,clips,downloads,storyboard,output}
```

### 0-B: project.json 생성 (허브 등록 필수 파일)

이 파일이 있어야 허브 서버가 프로젝트를 자동 인식한다.

```json
{
  "name": "프로젝트 표시 이름",
  "slug": "my-new-mv",
  "subtitle": "프로젝트 한줄 설명 — MV",
  "description": "프로젝트 상세 설명 (허브 카드에 표시)",
  "color": "#D4A853",
  "image_dir": "images",
  "clip_dir": "clips",
  "prompts_file": "prompts.json"
}
```

**필드 설명:**

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `name` | O | 허브에 표시되는 프로젝트명 | `"Journey to the Cross"` |
| `slug` | O | URL 경로 (영문, 하이픈 가능) | `"journey"` → `localhost:8787/journey/` |
| `subtitle` | O | 카드 부제목 | `"Pixar Style MV"` |
| `description` | O | 카드 설명 텍스트 | `"픽사 3D 스타일 워십 MV"` |
| `color` | O | 테마 색상 (hex) | `"#D4A853"` (골드), `"#4A90D9"` (블루) |
| `image_dir` | O | 이미지 폴더 경로 (상대) | `"images"` |
| `clip_dir` | O | 클립 저장 폴더 경로 (상대) | `"clips"` |
| `prompts_file` | O | 영상 생성 프롬프트 JSON 경로 | `"prompts.json"` |

### 0-C: prompts.json 생성 (빈 파일로 시작)

이미지별 영상 생성 프롬프트를 담는 파일. 초기에는 빈 객체로 생성하고, Phase 4~5에서 채운다.

```json
{}
```

**나중에 이미지가 추가되면 다음 형식으로 채운다:**

```json
{
  "01_scene_name.jpg": {
    "en": "Animate this illustration. [영문 프롬프트]. 5 seconds.",
    "ko": "일러스트를 애니메이트. [한글 프롬프트]. 5초."
  }
}
```

### 0-D: 최종 폴더 구조 확인

```
video_creator/
├── hub_server.py              ← 허브 서버
├── my-new-mv/                 ← 새 프로젝트 (자동 인식됨)
│   ├── project.json           ← 허브 등록 (필수)
│   ├── prompts.json           ← 영상 프롬프트 (필수)
│   ├── images/                ← 웹툰/픽사 이미지
│   ├── clips/                 ← AI 영상 클립 (Veo 생성물)
│   ├── downloads/             ← 레퍼런스 영상
│   ├── storyboard/            ← 스토리보드
│   └── output/                ← 최종 MV
├── 0320_왕사님/               ← 기존 프로젝트
├── 0321_신이랑법률사무소/      ← 기존 프로젝트
└── journey-to-cross/          ← 기존 프로젝트
```

### 0-E: 허브 서버 반영

프로젝트 폴더 생성 후 허브 서버를 재시작하면 자동 인식된다.

```bash
# 방법 1: 서버 재시작
lsof -i :8787 -t | xargs kill; sleep 1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ninestonelee/ai-mv-pipeline](https://github.com/ninestonelee/ai-mv-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
