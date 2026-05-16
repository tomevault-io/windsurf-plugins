---
trigger: always_on
description: Fleet UI Playground 앱 아키텍처 및 구조 가이드
---


# Fleet UI Playground 아키텍처 가이드

이 문서는 `apps/playground` 패키지의 전체 아키텍처, 라우트 구조, 페이지 설계 원칙을 정의합니다.

---

## 1. Playground의 역할과 목적

### 1.1 핵심 역할

Playground는 Fleet UI 디자인 시스템의 **시각적 경험을 제공하는 콘텐츠 앱**입니다.

| 역할 | 설명 |
|------|------|
| **콘텐츠 제공자** | 컴포넌트 데모, 쇼케이스, 시나리오를 순수하게 제공 |
| **독립 앱** | iOS/Android/Web에서 독립적으로 실행 가능 |
| **임베딩 소스** | Docs 웹사이트에서 iframe으로 임베딩 |

### 1.2 책임 분리 원칙

| 영역 | Playground | Docs (외부) |
|------|------------|-------------|
| 컴포넌트 데모 콘텐츠 | ✅ | ❌ |
| 쇼케이스 콘텐츠 | ✅ | ❌ |
| 디바이스 프레임 (PhoneMockup) | ❌ | ✅ |
| iframe 래퍼 (DemoFrame) | ❌ | ✅ |
| 기술 문서 (Props, Types) | ❌ | ✅ |

**원칙**: Playground는 순수 콘텐츠만 담당. 프레임/래핑은 소비자(Docs)가 담당.

---

## 2. 라우트 구조

### 2.1 전체 구조

```
apps/playground/app/
├── _layout.tsx                    # 루트 레이아웃
├── index.tsx                      # 메인 홈 (대시보드)
│
├── components/                    # 컴포넌트 데모 (Docs 임베딩용)
│   ├── _layout.tsx                # embed 쿼리로 헤더 토글
│   ├── index.tsx                  # 컴포넌트 목록 + 검색
│   └── [slug].tsx                 # 개별 컴포넌트 데모 (button, input 등)
│
├── showcases/                     # 컴포넌트 쇼케이스 (랜딩 페이지용)
│   ├── _layout.tsx                # embed 쿼리로 헤더 토글
│   ├── index.tsx                  # 쇼케이스 목록 + 검색
│   └── [slug].tsx                 # 개별 쇼케이스 (추가 예정)
│
├── scenarios/                     # 샘플 시나리오 페이지
│   ├── _layout.tsx
│   ├── index.tsx                  # 시나리오 목록
│   ├── onboarding.tsx
│   ├── settings.tsx
│   ├── billing.tsx
│   └── form.tsx
│
└── theme-demo.tsx                 # 토큰 시스템 시각화
```

### 2.2 라우트별 목적

| 라우트 | 목적 | 주요 소비자 |
|--------|------|-------------|
| `/components/[slug]` | 모든 Props를 체계적으로 시연 | Docs 컴포넌트 문서 페이지 |
| `/showcases/[slug]` | 실제 사용 맥락에서 컴포넌트 강조 | 랜딩 페이지, 마케팅 |
| `/scenarios/*` | 완성된 앱 화면 플로우 시연 | 독립 앱, Docs 쇼케이스 섹션 |
| `/token-architecture` | 토큰 시스템 전체 구조 시각화 | Docs 토큰 문서, 독립 앱 |

---

## 3. 임베딩 지원

### 3.1 쿼리 파라미터

모든 페이지는 쿼리 파라미터를 통해 임베딩 모드를 지원합니다.

| 파라미터 | 용도 | 값 |
|----------|------|-----|
| `embed` | 임베드 모드 (헤더 숨김) | `1`, `true` |
| `section` | 특정 섹션만 표시 | `variants`, `colorSchemes`, `sizes`, `states`, `all` |
| `theme` | 테마 강제 지정 | `light`, `dark` |
| `bg` | 배경색 투명 | `transparent` |

### 3.2 _layout.tsx 패턴

```typescript
// apps/playground/app/components/_layout.tsx
import { Stack, useLocalSearchParams } from 'expo-router';
import { useUnistyles } from 'react-native-unistyles';

export default function ComponentsLayout() {
  const { theme } = useUnistyles();
  const { embed } = useLocalSearchParams<{ embed?: string }>();
  const isEmbed = embed === '1' || embed === 'true';

  return (
    <Stack
      screenOptions={{
        headerShown: !isEmbed,
        headerStyle: { backgroundColor: theme.colors.neutral.content_1 },
        headerTintColor: theme.colors.neutral.text_1,
      }}
    >
      {/* Screen 정의 */}
    </Stack>
  );
}
```

### 3.3 URL 패턴 예시

```
# Docs 컴포넌트 문서 임베딩
/components/button?embed=1
/components/button?embed=1&section=colorSchemes

# 랜딩 페이지 쇼케이스 임베딩
/showcases/tabbar?embed=1
/showcases/bottom-sheet-modal?embed=1

# 시나리오 임베딩
/scenarios/onboarding?embed=1
```

---

## 4. 페이지 유형별 설계

### 4.1 Components 페이지 (`/components/[slug]`)

**목적**: Layer 2 스펙의 모든 시각적 속성을 체계적으로 시연

**구조**:
1. **PageHeader**: 컴포넌트 이름, 설명
2. **Quick Preview**: colorScheme × variant 매트릭스 (선택적)
3. **Visual Anatomy**: variants, sizes, rounded, shadow 각 섹션
4. **States**: hover, pressed, disabled, loading 상태
5. **Compositions**: 아이콘/텍스트 조합

**특징**:
- 스크롤 형태의 문서형 레이아웃
- 모든 Public Props를 최소 1회 이상 시연
- Section 컴포넌트로 그룹화

### 4.2 Showcases 페이지 (`/showcases/[slug]`)

**목적**: 실제 사용 맥락에서 컴포넌트의 시각적 임팩트 강조

**대상 컴포넌트 선정 기준**:
| 기준 | 예시 |
|------|------|
| 전체 화면/레이아웃 차지 | `TabBar`, `BottomSheetModal`, `Modal` |
| 인터랙티브 시나리오 중심 | `Swiper`, `OTPInput`, `Accordion` |
| 실제 사용 맥락이 중요 | `Toast`, `Menu`, `ContextHeader` |

**구조**:
- 실제 앱처럼 보이는 풀스크린 레이아웃
- 인터랙티브한 동작 시연
- 최소한의 설명, 컴포넌트 자체에 집중

**예시 - TabBar Showcase**:
```typescript
export default function TabBarShowcase() {
  const [activeTab, setActiveTab] = useState('home');
  
  return (
    <View style={styles.fullScreen}>
      <ScrollView style={styles.content}>
        {activeTab === 'home' && <FakeHomeFeed />}
        {activeTab === 'search' && <FakeSearchScreen />}
      </ScrollView>
      
      <TabBar
        items={TAB_ITEMS}
        activeKey={activeTab}
        onSelect={setActiveTab}
      />
    </View>
  );
}
```

### 4.3 Scenarios 페이지 (`/scenarios/*`)

**목적**: 여러 컴포넌트를 조합한 실제 앱 화면 플로우 시연

**현재 시나리오**:
- `onboarding.tsx` - 온보딩 플로우
- `settings.tsx` - 설정 화면
- `billing.tsx` - 결제/빌링 화면
- `form.tsx` - 폼 입력 화면

**특징**:
- 완성된 앱 화면 느낌
- 여러 컴포넌트 조합
- 실제 사용 시나리오 기반

### 4.4 Token Architecture 페이지 (`/token-architecture`)

**목적**: Fleet UI 토큰 시스템 전체 구조를 시각적으로 표현

**내용**:
- Color 팔레트 (primitive → semantic)
- Typography 스케일
- Spacing 시스템
- Border Radius
- Shadow 레벨
- 테마 전환 데모 (Light/Dark)

---

## 5. 목록 페이지 설계

### 5.1 공통 구조

`/components/index.tsx`와 `/showcases/index.tsx`는 동일한 패턴을 따릅니다.

```typescript
export default function ComponentsListPage() {
  const [searchQuery, setSearchQuery] = useState('');
  
  const filteredItems = useMemo(() => 
    COMPONENT_LIST.filter(item => 
      item.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
      item.description.toLowerCase().includes(searchQuery.toLowerCase())
    ),
    [searchQuery]
  );

  return (

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rengod95/Fleet-UI](https://github.com/Rengod95/Fleet-UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
