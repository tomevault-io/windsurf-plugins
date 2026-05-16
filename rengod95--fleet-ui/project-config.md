---
trigger: always_on
description: > 이 문서는 Playground의 컴포넌트 데모 페이지(`/components/[slug].tsx`) 작성 시 따라야 할 표준 템플릿을 정의합니다.
---

# Playground 컴포넌트 데모 페이지 템플릿

> 이 문서는 Playground의 컴포넌트 데모 페이지(`/components/[slug].tsx`) 작성 시 따라야 할 표준 템플릿을 정의합니다.

---

## 1. 페이지 목적 및 철학

### 1.1 목적

컴포넌트 데모 페이지는 **시각적 속성(Visual Props)**에 집중하여 Fleet UI 컴포넌트가 어떻게 보이는지 보여줍니다.

**Docs vs Playground 역할 구분:**

| | Docs 페이지 | Playground 페이지 |
|---|-------------|-------------------|
| **목적** | 기술 문서 | 시각적 데모 |
| **내용** | API, Props 타입, 사용법, 코드 예제 | 실제 렌더링 결과 |
| **대상** | 개발자 (구현 중) | 디자이너, PM, 개발자 (탐색 중) |
| **표시 속성** | 모든 Props (기능 포함) | 시각적 Props 중심 |
| **임베딩** | - | Docs/랜딩에 iframe 임베딩 |

### 1.2 표시할 Props

**필수 표시 (시각적 속성):**
- `variant` - 스타일 변형 (filled, outlined, ghost 등)
- `colorScheme` - 색상 테마 (primary, success, error 등)
- `size` - 크기 (sm, md, lg, xl 등)
- `rounded` - 모서리 둥글기 (none, sm, md, lg, full 등)
- `shadow` - 그림자 (none, sm, md, lg 등)
- `state` - 상태 (default, hover, focus, disabled, loading 등)
- 컴포넌트별 특수 시각적 Props

**표시하지 않음:**
- 이벤트 핸들러 (`onPress`, `onChange` 등)
- 제어 Props (`value`, `checked` 등)
- Ref, testID 등 기술적 Props

---

## 2. 페이지 구조

### 2.1 전체 레이아웃

```typescript
import { ComponentName } from '@fleet-ui/components';
import { ScrollView, View } from 'react-native';
import { StyleSheet, useUnistyles } from 'react-native-unistyles';
import { commonStyles, PageHeader, Section, DemoIcon } from '../../common/views';

// 1. Props 상수 정의
const VARIANTS = ['filled', 'outlined', 'ghost'] as const;
const COLOR_SCHEMES = ['primary', 'neutral', 'success', 'error'] as const;
const SIZES = ['sm', 'md', 'lg', 'xl'] as const;
const ROUNDED = ['none', 'sm', 'md', 'lg', 'full'] as const;
const SHADOWS = ['none', 'sm', 'md', 'lg'] as const;

export default function ComponentNameScreen() {
  useUnistyles();
  
  return (
    <ScrollView style={commonStyles.container}>
      <View style={commonStyles.content}>
        {/* 2. 페이지 헤더 */}
        <PageHeader 
          title="ComponentName" 
          description="Brief description of component purpose and visual characteristics." 
        />

        {/* 3. 섹션들 (Section 내부가 collapsible) */}
        <Section title="Overview" value="overview">
          <View style={styles.overviewContainer}>
            <ComponentName variant="filled">
              Basic Example
            </ComponentName>
          </View>
        </Section>

        <Section title="Variants" value="variants">
          <View style={commonStyles.row}>
            {VARIANTS.map(variant => (
              <ComponentName key={variant} variant={variant}>
                {variant}
              </ComponentName>
            ))}
          </View>
        </Section>

        <Section title="Color Schemes" value="colorSchemes">
          {COLOR_SCHEMES.map(scheme => (
            <View key={scheme} style={commonStyles.row}>
              {VARIANTS.map(variant => (
                <ComponentName 
                  key={`${scheme}-${variant}`}
                  colorScheme={scheme} 
                  variant={variant}
                >
                  {scheme}
                </ComponentName>
              ))}
            </View>
          ))}
        </Section>

        <Section title="Sizes" value="sizes">
          <View style={commonStyles.row}>
            {SIZES.map(size => (
              <ComponentName key={size} size={size}>
                {size}
              </ComponentName>
            ))}
          </View>
        </Section>

        <Section title="States" value="states">
          <View style={commonStyles.row}>
            <ComponentName>Default</ComponentName>
            <ComponentName disabled>Disabled</ComponentName>
            <ComponentName loading>Loading</ComponentName>
          </View>
        </Section>
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

**주요 변경사항:**
- `Section` 컴포넌트가 내부적으로 `Accordion.Item` 사용
- `Section`에 `value` prop 전달 (없으면 title 기반 자동 생성)
- 코드가 더 간결하고 읽기 쉬워짐
- 기존 데모 페이지 호환성 유지

### 2.2 Accordion 기반 섹션 구성

**`Section` 컴포넌트는 내부적으로 Accordion을 생성하여 섹션을 접고 펼칠 수 있습니다.**\n\n- 페이지는 `Section`을 그대로 나열하면 됩니다.\n- 기본은 **모두 열린 상태(defaultOpen=true)** 입니다.\n\n```typescript\n<Section title=\"Overview\" value=\"overview\">\n  {/* 내용 */}\n</Section>\n\n<Section title=\"Variants\" value=\"variants\">\n  {/* 내용 */}\n</Section>\n```

**Section Props:**
- `title` (필수): 섹션 제목
- `value`: Accordion 식별자 (없으면 title의 camelCase 자동 생성)
- `description`: 섹션 설명 (선택)
- `defaultOpen`: 기본 열림 상태 (기본값: true)

### 2.3 섹션 구성 순서 (표준)

| 순서 | 섹션 이름 | value | 설명 | 필수 여부 | defaultValue 포함 |
|------|-----------|-------|------|-----------|-------------------|
| 0 | **Overview** | `overview` | 기본 형태 단일 예제 | **필수** | ✅ |
| 1 | **Variants** | `variants` | 스타일 변형 | 필수 | ✅ |
| 2 | **Color Schemes** | `colorSchemes` | 색상 테마 | 필수 | ✅ |
| 3 | **Variants × Color Schemes** | `variantsColorSchemes` | 조합 매트릭스 | 선택 | ✅ |
| 4 | **Sizes** | `sizes` | 크기 변형 | 필수 | ✅ |
| 5 | **States** | `states` | 상태 | 필수 | ✅ |
| 6 | **Rounded** | `rounded` | 모서리 둥글기 | 선택 | 선택 |
| 7 | **Shadows** | `shadows` | 그림자 | 선택 | 선택 |
| 8 | **Icon Compositions** | `iconCompositions` | 아이콘 조합 | 선택 | 선택 |
| 9 | **Special Props** | `specialProps` | 컴포넌트 고유 속성 | 선택 | 선택 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rengod95/Fleet-UI](https://github.com/Rengod95/Fleet-UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
