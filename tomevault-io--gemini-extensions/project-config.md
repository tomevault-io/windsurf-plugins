---
trigger: always_on
description: > Datra는 게임 개발용 데이터 관리 시스템으로, C# Source Generator를 사용해 CSV/JSON/YAML 데이터의 직렬화/역직렬화 코드를 자동 생성합니다.
---

## datra

> Datra는 게임 개발용 데이터 관리 시스템으로, C# Source Generator를 사용해 CSV/JSON/YAML 데이터의 직렬화/역직렬화 코드를 자동 생성합니다.

# Datra - Claude 작업 가이드

## 프로젝트 개요

Datra는 게임 개발용 데이터 관리 시스템으로, C# Source Generator를 사용해 CSV/JSON/YAML 데이터의 직렬화/역직렬화 코드를 자동 생성합니다.

## 프로젝트 구조

```
Datra/
├── Datra/                      # 핵심 라이브러리 (런타임)
│   ├── Attributes/             # [TableData], [SingleData], [DatraConfiguration] 등
│   ├── DataTypes/              # DataRef<T>, LocaleRef 등
│   ├── Interfaces/             # ITableData, IDataRepository 등
│   └── Plugins/                # Unity용 빌드된 DLL (Generators, Analyzers)
│
├── Datra.Editor/               # 공유 에디터 레이어 (Unity/Blazor 공통)
│   ├── Interfaces/             # IFieldTypeHandler, IDataEditorService 등
│   ├── Models/                 # FieldCreationContext, FieldLayoutMode
│   ├── Services/               # FieldTypeRegistry
│   └── Utilities/              # TypeDetectionHelper, PathHelper
│
├── Datra.Generators/           # Source Generator (핵심!)
│   ├── Analyzers/              # DataModelAnalyzer - 클래스 분석
│   ├── Builders/               # CodeBuilder - 코드 생성 유틸리티
│   ├── Generators/             # 실제 코드 생성기들
│   │   ├── CsvSerializerBuilder.cs    # CSV 직렬화/역직렬화
│   │   ├── DataContextGenerator.cs    # DataContext 클래스 생성
│   │   ├── DataModelGenerator.cs      # 모델 partial 클래스 생성
│   │   └── JsonSerializerBuilder.cs   # JSON 직렬화/역직렬화
│   ├── Models/                 # DataModelInfo, PropertyInfo 등
│   └── DataContextSourceGenerator.cs  # 메인 진입점 (ISourceGenerator)
│
├── Datra.Analyzers/            # Roslyn Analyzer
├── Datra.SampleData/           # 테스트용 샘플 데이터 모델
├── Datra.SampleData2/          # 멀티 컨텍스트 테스트용
├── Datra.Tests/                # 유닛 테스트
├── Datra.Unity/                # Unity 패키지
└── Datra.Unity.Sample/         # Unity 샘플 프로젝트
```

## 빌드 명령어

```bash
# 전체 빌드 (Generator DLL → Unity 폴더로 복사)
./Scripts/build-all.sh

# 개별 빌드
dotnet build Datra.Generators/Datra.Generators.csproj -c Release
dotnet build Datra.SampleData/Datra.SampleData.csproj

# 테스트
dotnet test Datra.Tests/Datra.Tests.csproj

# 생성된 파일 확인 (EmitPhysicalFiles = true 설정 필요)
# DatraConfiguration.cs에서 EmitPhysicalFiles = true로 변경 후 빌드
```

## 핵심 파일 및 역할

### Source Generator 핵심 파일

| 파일 | 역할 |
|------|------|
| `DataContextSourceGenerator.cs` | 메인 진입점. DatraConfiguration 읽고 생성 시작 |
| `DataModelAnalyzer.cs` | 클래스 분석, PropertyInfo 추출, 중첩 타입 감지 |
| `CsvSerializerBuilder.cs` | CSV 직렬화/역직렬화 코드 생성 |
| `DataModelGenerator.cs` | 모델 partial 클래스, 생성자 생성 |
| `CodeBuilder.cs` | 코드 생성 헬퍼 (ToCamelCase, 예약어 처리 등) |

### 모델 정보 (DataModelInfo)

```csharp
// Datra.Generators/Models/DataModelInfo.cs
public class PropertyInfo
{
    public string Name { get; set; }
    public string Type { get; set; }
    public bool IsArray { get; set; }
    public bool IsEnum { get; set; }
    public bool IsDataRef { get; set; }
    public bool IsLocaleRef { get; set; }
    public bool IsNestedType { get; set; }      // 중첩 struct/class
    public bool IsNestedStruct { get; set; }    // struct vs class
    public string NestedTypeName { get; set; }
    public List<PropertyInfo> NestedProperties { get; set; }
}
```

## DatraConfiguration 설정

```csharp
[assembly: DatraConfiguration("GameData",
    Namespace = "MyGame.Generated",           // 필수! Unity 호환성
    EnableLocalization = true,
    LocalizationKeyDataPath = "Localizations/LocalizationKeys.csv",
    EmitPhysicalFiles = false                 // 디버깅용
)]
```

**Namespace는 필수** - 미설정시 `DATRA003` 에러 발생

## 자주 발생하는 문제

### 1. Unity에서 컴파일 에러
- **증상**: `CS0116`, `CS1514` 등 구문 에러
- **원인**:
  - C# 예약어가 변수명으로 사용됨 (예: `ref`)
  - 네임스페이스에 공백 포함
- **해결**:
  - `CodeBuilder.ToCamelCase`에서 예약어 처리 (`@ref`)
  - `Namespace` 명시적 설정 (필수화됨)

### 2. 중복 정의 에러
- **증상**: `CS0101` already contains a definition
- **원인**: `EmitPhysicalFiles = true`로 물리 파일 생성 후 끄지 않음
- **해결**: 생성된 `*.g.cs` 파일 삭제, `EmitPhysicalFiles = false`

### 3. Generator가 동작 안 함
- **원인**: DLL이 오래됨
- **해결**: `./Scripts/build-all.sh` 실행

## 코드 생성 흐름

```
1. DataContextSourceGenerator.Execute()
   ↓
2. DataAttributeSyntaxReceiver로 [TableData], [SingleData] 클래스 수집
   ↓
3. DataModelAnalyzer.AnalyzeClasses() - PropertyInfo 추출
   ↓
4. DataContextGenerator.GenerateDataContext() - Context 클래스 생성
   ↓
5. DataModelGenerator.GenerateDataModelFile() - 각 모델별 Serializer 생성
   ↓
6. context.AddSource()로 컴파일에 추가
```

## CSV 직렬화 특이사항

### 중첩 타입 (Nested Type)
```csv
Id,Name,ModelPrefab.Path,ModelPrefab.InitialCount
hero_001,Knight,Assets/Prefabs/Knight.prefab,5
```
- 점 표기법 사용: `PropertyName.FieldName`
- `CsvSerializerBuilder.GenerateNestedTypeDeserialization()` 참조

### 배열
```csv
Stats,UpgradeCosts
Strength|Agility,100|200|300
```
- 기본 구분자: `|` (config.CsvArrayDelimiter)

### DataRef
```csv
CharacterRef,ItemRefs
hero_001,1001|1002|1003
```
- ID만 저장, `Evaluate(context)`로 해결

## 테스트 방법

### 통합 테스트 (권장)

```bash
# 전체 테스트 (빌드 + .NET 테스트 + Unity 테스트)
./Scripts/test-all.sh

# 옵션
./Scripts/test-all.sh --skip-build      # 빌드 스킵
./Scripts/test-all.sh --skip-unity      # Unity 테스트 스킵
./Scripts/test-all.sh --unity-only      # Unity 테스트만
```

### .NET 테스트

```bash
# 전체 테스트
dotnet test Datra.Tests/Datra.Tests.csproj

# 특정 테스트
dotnet test --filter "FullyQualifiedName~NestedTypeTests"
```

### Unity 테스트

```bash
# Unity 컴파일만 체크

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomevault-io/gemini-extensions](https://github.com/tomevault-io/gemini-extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
