---
trigger: always_on
description: | Tailwind 포맷/유틸리티 | packages/ui/tailwind.config.cjs, packages/theme/tailwind.config.cjs |
---

# Tailwind CSS 베스트 프랙티스

## 1. 참고 파일 안내

| 상황/패턴 | 참고 파일 |
|---|---|
| Tailwind 포맷/유틸리티 | packages/ui/tailwind.config.cjs, packages/theme/tailwind.config.cjs |
| 디자인 토큰(CSS 변수) | packages/theme/src/styles/__tokens-light.css, __tokens-dark.css |
| SCSS(Sass) | 각 컴포넌트의 .scss 파일, packages/ui/src/style.css |
| 하이브리드/동적 스타일 | 위 모든 파일 + 컴포넌트 내 computed/style 로직 |

> **참고:**
> - Tailwind 커스텀 유틸리티/색상/spacing 등은 반드시 tailwind.config.cjs에서 확인/추가
> - 디자인 토큰(CSS 변수)은 __tokens-light.css, __tokens-dark.css에서 확인
> - SCSS에서 @apply, CSS 변수 사용 시 위 파일 경로 참고

---

## 2. 스타일 적용 기본 원칙
- **Tailwind 클래스로 정의된 것들은 Vue 파일에서 바로 사용한다.**
- **SCSS에서는 @apply를 사용하지 않고 CSS 변수를 직접 사용한다.**
- **미리 정의된 클래스 방식을 우선 사용하여 개발자 도구 가독성을 높인다.**
- **조건부 변경이 필요한 경우만 :style로 동적 처리한다.**
- **SCSS(Sass)를 적극 활용하여 컴포넌트별 스타일을 분리한다.**
- **디자인 토큰은 CSS 변수로 관리하고, 컴포넌트별 토큰만 SCSS에서 활용한다.**

---

## 3. CSS 클래스 방식 가이드

### 3.1 미리 정의된 클래스 방식 (권장)
개발자 도구에서 복잡한 클래스명을 피하고 가독성을 높이기 위해 미리 정의된 클래스를 사용하세요.

```vue
<!-- ❌ 피해야 할 방식 (동적 CSS 변수 클래스) -->
<button :class="`bg-[var(--button-${variant}-background)] text-[var(--button-${variant}-text)]`">
  버튼
</button>

<!-- ✅ 권장 방식 (미리 정의된 클래스) -->
<button :class="`btn-${variant}`">
  버튼
</button>
```

### 3.2 SCSS 파일 분리 패턴
컴포넌트별로 별도의 SCSS 파일을 생성하여 스타일을 관리하세요. SCSS에서는 컴포넌트별 토큰만 사용하고, Tailwind 클래스는 Vue 파일에서 직접 사용합니다.

```vue
<!-- BaseButton.vue -->
<script setup lang="ts">
import './BaseButton.scss'; // SCSS 파일 import
// ... 컴포넌트 로직
</script>

<template>
  <button :class="`btn-${variant}`">
    {{ label }}
  </button>
</template>
```

```scss
/* BaseButton.scss - 컴포넌트별 토큰만 사용 */
.btn-primary {
  background-color: var(--button-primary-background);
  color: var(--button-primary-text);
}

.btn-primary:hover {
  background-color: var(--button-primary-background-deep);
}

.btn-outline {
  background-color: transparent;
  color: var(--button-outline-text);
  border-color: var(--button-outline-border);
}

.btn-outline:hover {
  background-color: var(--button-outline-background);
}
```

### 3.3 클래스 매핑 패턴
Vue 컴포넌트에서 variant별 클래스 매핑을 사용하세요.

```typescript
const variantClasses = {
  primary: 'btn-primary',
  outline: 'btn-outline',
  red: 'btn-red',
  blue: 'btn-blue',
  // ... 기타 variant
};

const staticClasses = computed(() => {
  const classes = [
    'inline-flex items-center justify-center font-sans font-semibold transition-all',
    // ... 기본 클래스들
  ];
  
  classes.push(variantClasses[props.variant] || 'btn-primary');
  return classes.join(' ');
});
```

### 3.4 장점
- **개발자 도구 가독성**: `btn-primary` 같은 깔끔한 클래스명
- **성능**: 동적 클래스 생성보다 정적 클래스가 효율적
- **유지보수성**: 스타일을 별도 파일로 분리하여 관리 용이
- **디버깅**: 특정 variant의 스타일 문제를 쉽게 찾을 수 있음

---

## 4. 상황별 권장 방식

| 상황 | 권장 방식 | 참고 파일 |
|------|-----------|-----------|
| 단순 색상/값 적용 | 미리 정의된 클래스 + SCSS | 컴포넌트.scss |
| 조건부 색상 변경 | 동적 CSS 변수 (:style) | __tokens-light.css, 컴포넌트 |
| 반응형/상태 클래스 | Tailwind 클래스 | tailwind.config.cjs |
| 복잡한 조건부 스타일 | 하이브리드 | tailwind.config.cjs, __tokens-light.css, 컴포넌트 |
| 정적 컴포넌트 스타일 | 미리 정의된 클래스 + SCSS | 컴포넌트.scss |

---

## 5. 상황별 스타일 적용 예시

### 5.1 미리 정의된 클래스 + SCSS (권장)
```vue
<!-- Tailwind 클래스는 Vue 파일에서 직접 사용 -->
<!-- 컴포넌트별 토큰은 SCSS에서 정의 -->
<button :class="`btn-${variant} inline-flex items-center justify-center px-4 py-2 rounded-lg font-semibold transition-all`">
  {{ label }}
</button>
```

```scss
/* BaseButton.scss - 컴포넌트별 토큰만 사용 */
.btn-primary {
  background-color: var(--button-primary-background);
  color: var(--button-primary-text);
  border-color: var(--button-primary-border);
}

.btn-primary:hover {
  background-color: var(--button-primary-background-deep);
}

.btn-primary:disabled {
  background-color: var(--button-disabled-background);
}
```

### 5.2 동적 CSS 변수 (:style) - 조건부 변경 시만
```vue
<!-- 조건부 색상 변경이 필요한 경우만 -->
<!-- 참고: __tokens-light.css, 컴포넌트 내 computed -->
<script setup>
const buttonStyle = computed(() => {
  if (isDisabled.value) {
    return {
      backgroundColor: 'var(--button-disabled-background)',
      color: 'var(--button-disabled-text)'
    };
  }
  return {
    backgroundColor: `var(--button-${props.variant}-background)`,
    color: `var(--button-${props.variant}-text)`
  };
});
</script>
<template>
  <button :style="buttonStyle">
    {{ label }}
  </button>
</template>
```

### 5.3 하이브리드 접근법 (최적)
```vue
<!-- 미리 정의된 클래스 + 동적 조건부 스타일 -->
<!-- 참고: __tokens-light.css, 컴포넌트 내 computed -->
<script setup>
// 미리 정의된 클래스 사용
const variantClasses = {
  primary: 'btn-primary',
  outline: 'btn-outline',
  // ... 기타 variant
};

const staticClasses = computed(() => {
  const classes = [
    'inline-flex items-center justify-center font-sans font-semibold transition-all',
    // ... 기본 클래스들
  ];
  
  classes.push(variantClasses[props.variant] || 'btn-primary');
  return classes.join(' ');
});

const dynamicStyle = computed(() => {
  // 조건부 변경이 필요한 경우만
  if (borderVariants.includes(props.variant)) {
    return {
      borderWidth: '1px',
      borderStyle: 'solid',
    };
  }
  return {};
});
</script>
<template>
  <button
    :class="staticClasses"
    :style="dynamicStyle"
  >
    {{ label }}
  </button>
</template>
```

### 5.4 :class 사용 가이드
```vue
<!-- Tailwind 클래스는 Vue 파일에서 직접 사용 -->
<!-- 조건/상태/반응형에 따라 Tailwind 클래스 조합 -->
<!-- 참고: tailwind.config.cjs, __tokens-light.css -->
<template>
  <button
    :class="[

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bcg-dev-team) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
