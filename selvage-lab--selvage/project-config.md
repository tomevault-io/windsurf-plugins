---
trigger: always_on
description: architecture and workflow
---


# Selvage 앱 아키텍처 및 워크플로우

Selvage는 LLM(대규모 언어 모델)을 활용한 코드 리뷰 CLI 도구입니다. 이 도구는 Git diff를 분석하고, LLM에 전송하여 코드 리뷰 피드백을 받는 워크플로우를 구현합니다.

## 핵심 컴포넌트 및 워크플로우

### 1. Git Diff 파싱 및 메타데이터 처리

코드 리뷰 프로세스는 Git diff 파싱으로 시작됩니다:

- [parse_git_diff](mdc:selvage/src/diff_parser/parser.py) 함수는 Git diff 텍스트를 받아 구조화된 데이터로 변환합니다.
- 이 과정에서 파일 콘텐츠, 언어 타입, 변경된 코드 블록(hunks) 등의 메타데이터가 추가됩니다.

#### Git Diff 획득 과정

Selvage는 [GitDiffUtility](mdc:selvage/src/utils/git_utils.py) 클래스를 통해 다양한 모드로 Git diff를 획득합니다.

```python
class GitDiffMode(str, Enum):
    """git diff 동작 모드 열거형"""
    STAGED = "staged"        # 스테이징된 변경사항 (git add로 추가된 내용)
    TARGET_COMMIT = "target_commit"  # 특정 커밋과 HEAD 사이의 차이
    TARGET_BRANCH = "target_branch"  # 특정 브랜치와 현재 브랜치 사이의 차이
    UNSTAGED = "unstaged"    # 워킹 디렉토리의 변경사항 (스테이징되지 않은 내용)
```

CLI에서는 `get_diff_content` 함수를 통해 사용자 옵션에 따라 적절한 모드의 diff를 획득합니다:

```python
def get_diff_content(args: argparse.Namespace) -> str:
    """Git diff 내용을 가져옵니다."""
    try:
        git_diff = GitDiffUtility.from_args(args)
        return git_diff.get_diff()
    except ValueError as e:
        logger.error(str(e))
        return ""
```

GitDiffUtility의 `get_diff` 메서드는 모드에 맞는 Git 명령을 구성합니다:

```python
def get_diff(self) -> str:
    """Git diff 명령을 실행하고 결과를 반환합니다."""
    cmd = ["git", "-C", self.repo_path, "diff", "--unified=5"]

    if self.mode == GitDiffMode.STAGED:
        cmd.append("--cached")  # 스테이징된 변경사항
    elif self.mode == GitDiffMode.TARGET_COMMIT:
        # 특정 커밋과 HEAD 사이 비교
        cmd.append(f"{self.target}..HEAD")
    elif self.mode == GitDiffMode.TARGET_BRANCH:
        # 특정 브랜치와 HEAD 사이 비교
        cmd.append(f"{self.target}..HEAD")
    # UNSTAGED는 추가 옵션 없이 기본 명령 사용

    # 명령 실행 및 결과 반환
    process_result = subprocess.run(cmd, capture_output=True, text=True)
    return process_result.stdout
```

이를 통해 사용자는 다양한 Git 비교 모드로 코드 리뷰를 수행할 수 있습니다:

- `selvage review --staged`: 스테이징된 변경사항 리뷰
- `selvage review --target-commit abcd1234`: 특정 커밋(abcd1234)과 HEAD 사이의 변경사항 리뷰
- `selvage review --target-branch develop`: develop 브랜치와 현재 브랜치 사이의 변경사항 리뷰
- `selvage review`: 현재 워킹 디렉토리의 변경사항(unstaged) 리뷰

#### Git Diff 형식과 파싱 과정

표준 Git diff 형식은 다음과 같습니다:

```
diff --git a/file.py b/file.py
index abc123..def456 100644
--- a/file.py
+++ b/file.py
@@ -10,7 +10,7 @@ def example_function():
     # 이전 코드
-    old_code = "old"
+    new_code = "new"
     # 나머지 코드
```

이 diff 형식은 다음과 같은 구조로 파싱됩니다:

1. **DiffResult**: 전체 diff를 나타내는 컨테이너 객체

   ```python
   class DiffResult:
       files: list[FileDiff]
       # 기타 메타데이터
   ```

2. **FileDiff**: 변경된 각 파일을 나타내는 객체

   ```python
   class FileDiff:
       filename: str
       hunks: list[Hunk]
       language: str
       file_content: str
       additions: int = 0    # 추가된 라인 수
       deletions: int = 0    # 삭제된 라인 수
   ```

3. **Hunk**: 파일 내의 변경된 각 코드 블록
   ```python
   class Hunk:
       start_line_original: int
       line_count_original: int
       start_line_modified: int
       line_count_modified: int
       content: str  # 변경 내용을 포함하는 텍스트
   ```

파싱 과정:

1. diff 텍스트를 줄 단위로 분석
2. 파일 헤더(`diff --git`) 식별 및 새 FileDiff 객체 생성
3. 변경 블록 헤더(`@@ -10,7 +10,7 @@`) 식별 및 Hunk 객체 생성
4. 변경 줄 식별:
   - `-`로 시작: 제거된 줄
   - `+`로 시작: 추가된 줄
   - 공백으로 시작: 변경되지 않은 줄
5. 전체 파일 내용을 로드
6. 파일 확장자에 기반하여 언어 감지 및 설정

파싱 결과는 `ReviewRequest` 객체의 `processed_diff` 필드에 저장되어 프롬프트 생성에 활용됩니다.

#### 파싱된 결과 예시 (JSON)

예를 들어, 위의 간단한 diff 예시는 다음과 같은 JSON 구조로 변환됩니다:

```json
{
  "files": [
    {
      "filename": "file.py",
      "language": "python",
      "additions": 1,
      "deletions": 1,
      "hunks": [
        {
          "start_line_original": 10,
          "line_count_original": 7,
          "start_line_modified": 10,
          "line_count_modified": 7,
          "content": "def example_function():\n    # 이전 코드\n-    old_code = \"old\"\n+    new_code = \"new\"\n    # 나머지 코드",
          "before_code": "def example_function():\n    # 이전 코드\n    old_code = \"old\"\n    # 나머지 코드",
          "after_code": "def example_function():\n    # 이전 코드\n    new_code = \"new\"\n    # 나머지 코드"
        }
      ],
      "file_content": "# file.py\n\ndef example_function():\n    # 이전 코드\n    new_code = \"new\"\n    # 나머지 코드\n\n# 파일의 나머지 내용..."
    }
  ]
}
```

이 구조화된 데이터를 통해 각 파일의 변경 사항과 전체 컨텍스트를 명확히 이해할 수 있습니다:

1. **filename**: 변경된 파일 이름
2. **language**: 파일의 프로그래밍 언어
3. **additions**: 추가된 라인 수
4. **deletions**: 삭제된 라인 수
5. **hunks**: 변경된 코드 블록 목록
   - **start_line_original**: 원본 파일에서 변경이 시작된 줄 번호
   - **line_count_original**: 원본 파일에서 변경된 줄 수
   - **start_line_modified**: 수정된 파일에서 변경이 시작된 줄 번호
   - **line_count_modified**: 수정된 파일에서 변경된 줄 수
   - **content**: 변경 내용(기호 포함: +, -)
   - **before_code**: 변경 전 코드(기호 제외)
   - **after_code**: 변경 후 코드(기호 제외)
6. **file_content**: 전체 파일 내용

이 구조화된 데이터는 `create_code_review_prompt` 메서드에서 LLM에 전송할 프롬프트 생성에 사용됩니다.

### 2. 리뷰 요청 데이터 구조화

파싱된 diff는 [ReviewRequest](mdc:selvage/src/utils/token/models.py) 모델로 구조화됩니다:

```python
class ReviewRequest(BaseModel):
    """코드 리뷰 요청 모델"""
    diff_content: str
    processed_diff: DiffResult
    file_paths: list[str] = Field(default_factory=list)
    model: str
    repo_path: str
```

#### ReviewRequest 모델 상세 설명

`ReviewRequest` 클래스는 코드 리뷰에 필요한 모든 정보를 캡슐화하는 Pydantic 모델입니다:

1. **diff_content**: 원본 Git diff 텍스트

   ```
   diff --git a/main.py b/main.py
   index abc123..def456 100644
   ...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selvage-lab/selvage](https://github.com/selvage-lab/selvage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
