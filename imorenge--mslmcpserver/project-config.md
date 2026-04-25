---
trigger: always_on
description: - **필수**: `YYYY-MM-DD` 형식 사용
---

# VoiceMacro Pro 개발 규칙 및 가이드라인

## 📝 Git 커밋 규칙

### 🏷️ 커밋 이름 형식
```
날짜-기능분류: 간략한 변경사항
```

#### 📅 날짜 형식
- **필수**: `YYYY-MM-DD` 형식 사용
- **예시**: `2025-mm-dd`, `2025-12-25`

#### 🏭 기능분류 태그
| 태그 | 설명 | 예시 |
|------|------|------|
| `feat` | 새로운 기능 추가 | `2025-mm-dd-feat: 음성인식 시스템 추가` |
| `fix` | 버그 수정 | `2025-mm-dd-fix: 매크로 실행 오류 수정` |
| `update` | 기존 기능 개선/수정 | `2025-mm-dd-update: UI 레이아웃 개선` |
| `refactor` | 코드 리팩토링 | `2025-mm-dd-refactor: 데이터베이스 모듈 분리` |
| `docs` | 문서 수정 | `2025-mm-dd-docs: API 문서 업데이트` |
| `style` | 코드 스타일 변경 | `2025-mm-dd-style: 코드 포맷팅 적용` |
| `test` | 테스트 추가/수정 | `2025-mm-dd-test: 단위테스트 추가` |
| `init` | 프로젝트 초기화 | `2025-mm-dd-init: 프로젝트 구조 생성` |

### 📋 커밋 메시지 작성 규칙

#### ✅ 올바른 커밋 예시
```bash
# 커밋 이름: 간략하게
git commit -m "2025-mm-dd-feat: 매크로 관리 UI 추가" -m "
📍 변경된 파일들:
- VoiceMacroPro/Views/MacroManagementWindow.xaml 생성
- VoiceMacroPro/Views/MacroManagementWindow.xaml.cs 생성
- VoiceMacroPro/Models/MacroModel.cs 수정

🔧 주요 변경사항:
- 매크로 목록 표시를 위한 DataGrid 구현
- 매크로 CRUD 기능 버튼 추가 (추가/수정/삭제/복사)
- 매크로 검색 및 필터링 기능 구현
- 매크로 정렬 기능 (이름순/생성일순/사용빈도순)

🎯 구현된 기능:
- 매크로 목록 실시간 업데이트
- 매크로 선택 시 상세정보 표시
- 우클릭 컨텍스트 메뉴 제공

🧪 테스트 상태:
- UI 레이아웃 테스트 완료
- 기본 CRUD 동작 테스트 완료
- 데이터바인딩 동작 확인 완료

⚠️ 알려진 이슈:
- 매크로 삭제 시 확인 대화상자 미구현
- 대량 데이터 로딩 시 성능 최적화 필요
"
```

```bash
# 버그 수정 예시
git commit -m "2025-mm-dd-fix: 데이터베이스 연결 실패 문제 해결" -m "
🐛 문제 상황:
- 앱 시작 시 데이터베이스 파일이 없을 때 크래시 발생
- SQLite 데이터베이스 경로 문제로 connection 실패

🔧 수정 내용:
- database.py에 데이터베이스 파일 존재 여부 확인 로직 추가
- 데이터베이스 파일이 없을 경우 자동 생성 기능 구현
- 데이터베이스 초기화 시 스키마 자동 생성

📍 변경된 파일들:
- database.py: init_database() 메서드 수정
- database.py: _create_initial_schema() 메서드 추가

🧪 테스트 결과:
- 새로운 환경에서 앱 시작 테스트 완료
- 기존 데이터베이스 연결 테스트 완료
- 스키마 생성 및 기본 데이터 삽입 테스트 완료
"
```

#### ❌ 잘못된 커밋 예시
```bash
# ❌ 날짜 누락
git commit -m "feat: UI 추가"

# ❌ 너무 간략한 메시지
git commit -m "2025-mm-dd-fix: 버그수정"

# ❌ 한글 타입 미사용 (가독성 저하)
git commit -m "2025-mm-dd-feat: add macro management" -m "
- Added MacroManagementWindow.xaml
- Modified MacroModel.cs
"

# ❌ 변경사항 설명 부족
git commit -m "2025-mm-dd-update: 코드 수정" -m "파일들 업데이트함"
```

### 📂 커밋 단위 규칙

#### ✅ 올바른 커밋 단위
- **하나의 기능 완성 단위로 커밋**
- **관련된 파일들을 한번에 커밋**
- **동작하는 상태에서 커밋**

```bash
# ✅ 좋은 예: 매크로 추가 기능 완성
git add VoiceMacroPro/Views/AddMacroWindow.xaml
git add VoiceMacroPro/Views/AddMacroWindow.xaml.cs  
git add VoiceMacroPro/Models/MacroModel.cs
git commit -m "2025-mm-dd-feat: 매크로 추가 기능 구현"

# ✅ 좋은 예: 버그 수정 단위
git add database.py
git commit -m "2025-mm-dd-fix: 데이터베이스 연결 오류 수정"
```

#### ❌ 잘못된 커밋 단위  
```bash
# ❌ 나쁜 예: 파일별로 개별 커밋
git add VoiceMacroPro/Views/AddMacroWindow.xaml
git commit -m "2025-mm-dd-feat: XAML 파일 추가"

git add VoiceMacroPro/Views/AddMacroWindow.xaml.cs
git commit -m "2025-mm-dd-feat: C# 파일 추가"

# ❌ 나쁜 예: 동작하지 않는 상태에서 커밋
git add . 
git commit -m "2025-mm-dd-feat: 개발 중인 기능 저장"
```

### 🔄 커밋 히스토리 관리

#### 📜 브랜치 전략
```bash
# 메인 개발 브랜치
main (또는 master)

# 기능별 브랜치 생성 시 (선택사항)
feature/2025-mm-dd-voice-recognition
feature/2025-mm-dd-macro-management
```

#### 🏃‍♂️ 커밋 전 체크리스트
- [ ] 코드가 정상 동작하는가?
- [ ] 컴파일 오류가 없는가?
- [ ] 주석이 적절히 작성되었는가?
- [ ] 커밋 메시지가 규칙에 맞는가?
- [ ] 관련 파일들이 모두 포함되었는가?


---

## 📁 디렉토리 구조 규칙

### 🚫 절대 금지 사항
- **기존 디렉토리 구조 변경 금지**
- **중복 파일 생성 금지** (동일한 기능의 파일을 여러 위치에 생성)
- **무분별한 새 폴더 생성 금지**

### ✅ 디렉토리 사용 규칙

```
📁 Gamesuport/ (프로젝트 루트)
├── 🐍 Python 백엔드 파일들 (루트에 직접 배치)
│   ├── database.py           # 데이터베이스 전용
│   ├── *_service.py          # 비즈니스 로직 전용
│   ├── api_server.py         # API 엔드포인트 전용
│   └── *_utils.py            # 공통 유틸리티 (향후 추가시)
│
└── 📁 VoiceMacroPro/ (C# WPF 전용)
    ├── Models/               # 데이터 모델만
    ├── Services/             # API 통신 및 비즈니스 로직만
    ├── Views/                # UI 윈도우만
    ├── Utils/                # 공통 유틸리티 (향후 추가시)
    └── Resources/            # 리소스 파일 (향후 추가시)
```

---

## 🔄 코드 중복 방지 규칙

### 1. 🐍 Python 백엔드 중복 방지

#### ✅ DO (권장사항)
```python
# ✅ 올바른 예: 공통 기능은 별도 파일로 분리
# common_utils.py (새로 생성할 경우)
def format_datetime(dt):
    """공통 날짜 포맷팅 함수"""
    return dt.strftime("%Y-%m-%d %H:%M:%S")

def validate_required_fields(data, required_fields):
    """공통 필드 검증 함수"""
    for field in required_fields:
        if not data.get(field):
            raise ValueError(f"필수 필드 누락: {field}")

# database.py에서 사용
from common_utils import format_datetime

# macro_service.py에서 사용  
from common_utils import validate_required_fields
```

#### ❌ DON'T (금지사항)
```python
# ❌ 금지: 동일한 함수를 여러 파일에 복사
# database.py
def format_datetime(dt):  # 중복!
    return dt.strftime("%Y-%m-%d %H:%M:%S")

# macro_service.py  
def format_datetime(dt):  # 중복!
    return dt.strftime("%Y-%m-%d %H:%M:%S")
```

### 2. 🖥️ C# WPF 중복 방지

#### ✅ DO (권장사항)
```csharp
// ✅ 올바른 예: 공통 UI 로직은 Utils 클래스로 분리
// Utils/UIHelper.cs (새로 생성할 경우)
public static class UIHelper
{
    /// <summary>
    /// 공통 메시지 박스 표시
    /// </summary>
    public static void ShowError(string message)
    {
        MessageBox.Show(message, "오류", MessageBoxButton.OK, MessageBoxImage.Error);
    }
    
    /// <summary>
    /// 공통 확인 대화상자
    /// </summary>
    public static bool ShowConfirm(string message)
    {
        return MessageBox.Show(message, "확인", MessageBoxButton.YesNo, 
                              MessageBoxImage.Question) == MessageBoxResult.Yes;
    }
}

// MainWindow.xaml.cs에서 사용
UIHelper.ShowError("매크로 로드 실패");

// MacroEditWindow.xaml.cs에서 사용
if (UIHelper.ShowConfirm("변경사항을 저장하시겠습니까?"))
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ImOrenge) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
