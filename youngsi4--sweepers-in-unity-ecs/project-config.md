---
trigger: always_on
description: - **게임명**: Sweepers in ECS
---

# Sweepers in ECS — Claude Code 작성 지침

## 프로젝트 개요

- **게임명**: Sweepers in ECS
- **엔진**: Unity 6 (6000.3.11f1)
- **렌더 파이프라인**: URP (Universal Render Pipeline)
- **아키텍처**: DOTS / ECS (Entity Component System)
- **언어**: C# (.NET)

## 핵심 패키지

| 패키지 | 버전 | 용도 |
|---|---|---|
| com.unity.render-pipelines.universal | 17.3.0 | URP 렌더링 |
| com.unity.inputsystem | 1.19.0 | 입력 처리 |
| com.unity.ai.navigation | 2.0.11 | AI 내비게이션 |
| com.unity.timeline | 1.8.11 | 컷씬/타임라인 |
| com.unity.burst | (간접 의존) | 고성능 컴파일 |
| com.unity.mathematics | (간접 의존) | SIMD 수학 라이브러리 |
| com.unity.collections | (간접 의존) | NativeContainer |

## 폴더 구조

```
Assets/
├── Scenes/          # Unity 씬 파일
├── Scripts/         # C# 스크립트
│   ├── Components/  # ECS 컴포넌트 (IComponentData 등)
│   ├── Systems/     # ECS 시스템 (ISystem 등)
│   ├── Authoring/   # Baker/Authoring MonoBehaviour
│   └── Hybrid/      # ECS와 연결되는 MonoBehaviour
├── Settings/        # URP 렌더러 에셋
└── ...
```

## 코드 컨벤션

### 일반 C# 규칙
- 클래스/구조체/메서드: `PascalCase`
- 필드(private): `_camelCase` (언더스코어 접두사)
- 로컬 변수/파라미터: `camelCase`
- 상수: `PascalCase` (const/static readonly)
- 인터페이스: `IFoo` 형태 유지

### ECS / DOTS 규칙
- 컴포넌트는 `IComponentData`를 구현하는 `struct`로 작성
- 시스템은 `ISystem` 인터페이스 사용 (class 기반 `SystemBase`보다 선호)
- `Burst.CompileAttribute`를 최대한 활용해 성능 확보
- `NativeArray`, `NativeList` 등 NativeContainer 사용 시 `Allocator` 명시
- `EntityQuery`는 `OnCreate`에서 캐싱하여 재사용
- Authoring 컴포넌트는 `Baker<T>`를 통해 ECS 데이터로 변환

### 예시 패턴

```csharp
// 컴포넌트
public struct Speed : IComponentData
{
    public float Value;
}

// 시스템
[BurstCompile]
public partial struct MoveSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        float dt = SystemAPI.Time.DeltaTime;
        foreach (var (transform, speed) in
            SystemAPI.Query<RefRW<LocalTransform>, RefRO<Speed>>())
        {
            transform.ValueRW.Position.y += speed.ValueRO.Value * dt;
        }
    }
}

// Authoring
public class SpeedAuthoring : MonoBehaviour
{
    public float speed = 5f;

    class Baker : Baker<SpeedAuthoring>
    {
        public override void Bake(SpeedAuthoring authoring)
        {
            var entity = GetEntity(TransformUsageFlags.Dynamic);
            AddComponent(entity, new Speed { Value = authoring.speed });
        }
    }
}
```

## 작업 시 주의사항

- `.meta` 파일은 Unity가 자동 생성하므로 직접 편집하지 말 것
- `Library/`, `Temp/`, `obj/` 폴더는 git에 포함하지 말 것 (이미 gitignore 처리됨)
- URP 설정(`Assets/Settings/`)을 변경할 때는 PC/Mobile 렌더러를 구분하여 적용
- `Packages/manifest.json` 변경 후 Unity 에디터를 재시작해야 적용됨
- DOTS 패키지 추가 시 `com.unity.entities` 등을 manifest.json에 명시적으로 추가 필요

## 금지 사항

- MonoBehaviour `Update()`에서 heavy한 로직 실행 금지 — ECS 시스템으로 이전
- `GameObject.Find`, `FindObjectOfType` 런타임 사용 금지 — 직접 참조 또는 ECS Query 활용
- `BurstCompile` 시스템 내에서 관리형(Managed) 타입 사용 금지
- 불필요한 `Allocator.Persistent` NativeContainer 생성 금지 — 수명 관리 명확히 할 것

### 코드 수정 규칙
- ⛔ **사용자의 명시적 요청 없이 코드를 수정하지 말 것**
- 문제를 발견했을 때는 원인과 수정 방법을 설명하고, 사용자의 승인을 받은 후 수정할 것
- 분석·설명·제안은 자유롭게 하되, 실제 파일 변경은 사용자가 "수정해줘", "적용해줘" 등 명시적으로 요청한 경우에만 가능
- 이유: 코드 수정은 의도치 않은 동작을 유발할 수 있으므로, 사용자의 최종 판단 필수

### 파일 삭제 규칙
- ⛔ **`rm -rf` 명령을 자동으로 실행하지 말 것**
- 삭제할 파일이나 폴더가 있으면, 경로와 대상을 사용자에게 명확히 **알리고 승인 받을 것**
- 사용자가 직접 `rm -rf`를 실행하거나, "진행하세요"라고 명시 지시해야만 실행 가능
- 이유: 파일 삭제는 되돌릴 수 없는 작업이므로, 사용자의 최종 판단 필수

---

## GUI 출력 지양

Claude Code와 상호작용할 때는 **텍스트 기반 인터페이스**만 사용합니다. 그래픽 요소나 선택 UI는 토큰을 낭비하고 자동화를 방해합니다.

### 금지 사항
- AskUserQuestion에서 `preview` 필드 사용 금지 (다중 라인 선택지 시각화)
- 이모지 과다 사용 (필요하지 않으면 제거 — 사용자가 명시 요청 시에만)
- 마크다운 `<details>` 태그 등 숨김 영역 생성 금지
- 실행 결과 요약표, 진행 상황 바, 컬러 코드 등 시각적 요소 최소화

### 권장 방식
- 선택지는 텍스트 옵션만 제시 (`option 1`, `option 2`, ...)
- 질문은 간결한 텍스트 목록 형식
- 필요시 마크다운 코드 블록으로 구조화
- 결과는 숫자, 파일명, 상태 텍스트로만 표현

### AskUserQuestion 사용 규칙
- multiSelect 필드 최소화 (true일 때도 선택지는 최소 2개, 최대 4개)
- preview 필드는 절대 사용 금지
- 한 번에 질문은 최대 3개 (더 많으면 순차 질문)
- 선택지 설명은 1줄 (2줄 이상 금지)

---

## claude_tools 작업 규칙

### 한글 출력 명시
- 모든 claude_tools 에이전트 프롬프트는 **한글로 작성**
- Plan, Review, Implementation, Orchestration 결과물도 모두 **한글 마크다운**
- 사용자 요청이 한글이면 에이전트 출력도 한글로 유지

### UTF-8 인코딩 처리
- **프롬프트 작성**: 파일 상단에 `# -*- coding: utf-8 -*-` 선언
- **subprocess 호출**: `encoding='utf-8', errors='replace'` 파라미터 필수
- **파일 쓰기/읽기**: 항상 `encoding='utf-8'` 명시
- **Windows 콘솔**: 세션 시작 시 stdout을 UTF-8로 래핑:
  ```python
  import io, sys
  if sys.platform == 'win32' and not isinstance(sys.stdout, io.TextIOWrapper):
      try:
          sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')
      except:
          pass  # 이미 래핑되었거나 불가능한 경우
  ```

### subprocess에서 Claude CLI 호출 시 (권장)

Claude Code CLI를 subprocess로 호출하여 agentic orchestration을 구현합니다. 추가 요금 없이 기존 Claude Code 환경을 활용합니다.

#### 권장 패턴: stdin + 환경변수

```python
import subprocess
import os

def call_agent(agent_name: str, prompt: str, timeout: int = 600) -> str:
    """Claude CLI를 stdin으로 호출 (복잡한 프롬프트 안정적 전달)"""
    cmd = [
        "claude",
        "-p",              # 비인터랙티브 모드 (stdout으로 출력)
        "--model", "claude-haiku-4-5-20251001"
    ]
    
    env = os.environ.copy()
    env["CLAUDE_CODE_GIT_BASH_PATH"] = r"D:\Git\usr\bin\bash.exe"
    
    result = subprocess.run(
        cmd,
        input=prompt,      # stdin으로 프롬프트 전달 (복잡한 텍스트 안정적)
        capture_output=True,
        text=True,
        encoding='utf-8',
        errors='replace',
        timeout=timeout,
        cwd="/path/to/project",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YoungSi4/Sweepers-in-Unity-ECS](https://github.com/YoungSi4/Sweepers-in-Unity-ECS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
