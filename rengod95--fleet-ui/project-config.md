---
trigger: always_on
description: 이 문서는 `apps/playground/app/components/` 디렉터리에 새로운 컴포넌트 예제 페이지를 작성할 때 따라야 할 규칙을 정의합니다.
---

# Playground 컴포넌트 예제 페이지 작성 가이드

이 문서는 `apps/playground/app/components/` 디렉터리에 새로운 컴포넌트 예제 페이지를 작성할 때 따라야 할 규칙을 정의합니다.

> **참조**: 
> - 전체 Playground 아키텍처: `playground-architecture-guide.mdc`
> - 상세 템플릿 및 패턴: `playground-demo-page-template.mdc` ⭐

## 1. 기본 원칙

1. **1:1 매핑**: `@fleet-ui/components` 패키지의 모든 컴포넌트는 Playground에 대응하는 예제 페이지가 있어야 합니다.
2. **전수 조사**: Layer 2 Spec(`component-spec-template.md`)에 정의된 모든 Public Props를 예제로 시연해야 합니다.
3. **Unistyles 사용**: 모든 스타일링은 `react-native-unistyles`와 테마 토큰을 사용해야 합니다. 하드코딩된 값(색상 코드 등)은 금지됩니다.
4. **레지스트리 등록**: 새 컴포넌트 페이지 생성 시 `common/registry/components.ts`에 등록해야 합니다.

## 2. 컴포넌트 사용 (Mandatory)

### 2.1 Fleet UI 컴포넌트 우선 사용

**Playground의 모든 페이지는 `@fleet-ui/components` 패키지의 컴포넌트를 기반으로 구축해야 합니다.**

```typescript
// ✅ 올바른 예시
import {
  LayoutTop,
  Section,
  Item,
  ItemContent,
  Input,
  Icon,
  Button,
} from '@fleet-ui/components';
import { PageHeader, Section as DemoSection, DemoIcon } from '../../common/views'; // 데모 전용만
```

**작성 원칙:**
1. **`@fleet-ui/components` 먼저 확인** - 필요한 컴포넌트가 Fleet UI에 있는지 검토
2. **Fleet UI 컴포넌트 사용** - 있다면 반드시 Fleet UI 컴포넌트 사용
3. **데모 전용만 `common/views/` 사용** - PageHeader, Section, DemoIcon 등 데모 목적 컴포넌트만
4. **신규 생성 금지** - 범용 UI 컴포넌트를 `common/views/`에 생성하지 않음

### 2.2 컴포넌트 데모 페이지용 Import

```typescript
// Fleet UI 컴포넌트 (우선)
import { Button } from '@fleet-ui/components';

// Playground 데모 전용 컴포넌트
import { PageHeader, Section, DemoIcon, commonStyles } from '../../common/views';

// React Native 기본
import { ScrollView, View } from 'react-native';
import { useUnistyles } from 'react-native-unistyles';
```

### 2.3 컴포넌트 선택 가이드

| 용도 | 사용할 컴포넌트 | 출처 |
| :--- | :--- | :--- |
| 데모 페이지 헤더 | `PageHeader` | `common/views` |
| Props 그룹 섹션 | `Section` | `common/views` |
| 더미 아이콘 | `DemoIcon` | `common/views` |
| 공통 레이아웃 | `commonStyles` | `common/views` |
| 기타 모든 UI | Fleet UI 컴포넌트 | `@fleet-ui/components` |

## 3. 작성 패턴

> **⭐ 상세 템플릿**: 섹션별 패턴, 스타일링 규칙, 완전한 예제는 `playground-demo-page-template.mdc`를 참조하세요.

### 3.1 기본 구조 (Section 자체가 collapsible)

```typescript
import { MyComponent } from '@fleet-ui/components';
import { ScrollView, View } from 'react-native';
import { StyleSheet, useUnistyles } from 'react-native-unistyles';
import { PageHeader, Section, commonStyles } from '../../common/views';

// 1. Props 상수 정의
const VARIANTS = ['filled', 'outlined', 'ghost'] as const;
const COLOR_SCHEMES = ['primary', 'neutral', 'success'] as const;
const SIZES = ['sm', 'md', 'lg'] as const;

export default function MyComponentScreen() {
  useUnistyles();
  
  return (
    <ScrollView style={commonStyles.container}>
      <View style={commonStyles.content}>
        <PageHeader 
          title="MyComponent" 
          description="Visual characteristics and variations." 
        />

        {/* Section은 내부적으로 Accordion을 생성합니다 (기본: 열림) */}
        <Section title="Overview" value="overview">
          <View style={styles.overviewContainer}>
            <MyComponent variant="filled">Basic</MyComponent>
          </View>
        </Section>

        <Section title="Variants" value="variants">
          <View style={commonStyles.row}>
            {VARIANTS.map(variant => (
              <MyComponent key={variant} variant={variant}>{variant}</MyComponent>
            ))}
          </View>
        </Section>

        <Section title="Color Schemes" value="colorSchemes">
          {COLOR_SCHEMES.map(scheme => (
            <View key={scheme} style={commonStyles.row}>
              {VARIANTS.map(variant => (
                <MyComponent 
                  key={`${scheme}-${variant}`}
                  colorScheme={scheme} 
                  variant={variant}
                >
                  {scheme}
                </MyComponent>
              ))}
            </View>
          ))}
        </Section>

        {/* Sizes, States 등 추가 섹션들... */}
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create((theme) => ({
  overviewContainer: {
    alignItems: 'center',
    justifyContent: 'center',
    paddingVertical: theme.spacing[8],
  },
}));
```

**주요 특징:**
- `Section` 컴포넌트가 내부적으로 `Accordion.Item` 사용
- 코드가 간결하고 읽기 쉬움
- 자동 스타일링 및 간격 처리
- 기존 데모 페이지와 호환 가능

### 3.2 섹션 구성 순서

**필수 섹션 (순서대로, 모두 기본 열림):**
0. **Overview** - 기본 형태 단일 예제 ⭐ 최상단 필수
   ```typescript
   <Section title="Overview" value="overview">...</Section>
   ```
1. **Variants** - 스타일 변형
   ```typescript
   <Section title="Variants" value="variants">...</Section>
   ```
2. **Color Schemes** - 색상 테마
   ```typescript
   <Section title="Color Schemes" value="colorSchemes">...</Section>
   ```
3. **Sizes** - 크기 변형
   ```typescript
   <Section title="Sizes" value="sizes">...</Section>
   ```
4. **States** - 상태
   ```typescript
   <Section title="States" value="states">...</Section>
   ```

**선택 섹션 (필요시 추가):**
5. Rounded - 모서리 둥글기 (`value="rounded"`)
6. Shadows - 그림자 (`value="shadows"`)
7. Icon Compositions - 아이콘 조합 (`value="iconCompositions"`)
8. Full Width - 전체 너비 (`value="fullWidth"`)

**Section Props:**
- `title` (필수): 섹션 제목
- `value`: Accordion 식별자 (없으면 title의 camelCase 자동 생성)
- `description`: 섹션 설명 (선택)
- `defaultOpen`: 기본 열림 상태 (기본값: true)

**상세 내용**: `playground-demo-page-template.mdc` 섹션 2, 3 참조

## 4. 체크리스트

### 필수 항목
- [ ] `PageHeader`에 컴포넌트 이름과 설명 포함

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rengod95/Fleet-UI](https://github.com/Rengod95/Fleet-UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
