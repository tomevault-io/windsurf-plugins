---
trigger: always_on
description: 前端 CSS 样式规范和最佳实践
---


# CSS 样式规范

## 基本规范

### 格式化

```css
/* ✅ 使用 2 空格缩进 */
.container {
  padding: 16px;
  margin: 0 auto;
}

/* ✅ 每个属性独占一行 */
.card {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

/* ❌ 避免：所有属性在一行 */
.card { display: flex; flex-direction: column; gap: 12px; }

/* ✅ 属性值后加分号 */
.box {
  width: 100%;
  height: auto;
}

/* ✅ 颜色值使用小写 */
.primary {
  color: #1890ff;
  background: rgba(24, 144, 255, 0.1);
}
```

### 命名规范

```css
/* ✅ 使用 kebab-case（连字符） */
.container-card { }
.user-profile-header { }
.btn-primary { }

/* ❌ 避免使用 camelCase 或 snake_case */
.containerCard { }
.user_profile_header { }

/* ✅ 使用有意义的类名 */
.container-list { }
.status-badge { }
.loading-spinner { }

/* ❌ 避免缩写和无意义的名称 */
.cl { }
.sb { }
.item1 { }
```

### BEM 命名法（可选）

```css
/* Block（块） */
.card { }

/* Element（元素）：使用双下划线 */
.card__header { }
.card__body { }
.card__footer { }

/* Modifier（修饰符）：使用双连字符 */
.card--primary { }
.card--disabled { }
.card__title--large { }

/* 示例 */
.container-card { }
.container-card__header { }
.container-card__title { }
.container-card--running { }
.container-card--stopped { }
```

## Vue Scoped 样式

```vue
<style scoped>
/* ✅ 使用 scoped 避免全局污染 */
.container {
  padding: 20px;
}

/* ✅ 使用深度选择器修改子组件样式 */
:deep(.n-button) {
  border-radius: 8px;
}

/* ✅ 使用 :slotted 为插槽内容添加样式 */
:slotted(.custom-content) {
  margin: 10px;
}

/* ✅ 使用 :global 定义全局样式（谨慎使用） */
:global(.global-utility) {
  display: flex;
}
</style>
```

## CSS 变量（主题系统）

```css
/* ✅ 使用 Naive UI 的主题变量 */
.card {
  background: var(--n-color);
  color: var(--n-text-color);
  border: 1px solid var(--n-border-color);
}

.title {
  color: var(--n-text-color);
  font-size: var(--n-font-size);
  font-weight: var(--n-font-weight-strong);
}

.button {
  background: var(--n-color-primary);
  color: var(--n-text-color-primary);
}

/* ✅ 定义自定义 CSS 变量 */
:root {
  --app-header-height: 64px;
  --app-sidebar-width: 240px;
  --app-spacing: 16px;
}

.layout {
  padding: var(--app-spacing);
}
```

## Flexbox 布局

```css
/* ✅ 使用 Flexbox 进行布局 */
.container {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

/* ✅ 使用 flex 简写 */
.sidebar {
  flex: 0 0 240px; /* flex-grow flex-shrink flex-basis */
}

.content {
  flex: 1; /* 占据剩余空间 */
}

/* ✅ 使用 gap 替代 margin */
.list {
  display: flex;
  gap: 8px; /* ✅ 推荐 */
}

/* ❌ 避免使用 margin 处理间距 */
.list-item {
  margin-right: 8px;
}
.list-item:last-child {
  margin-right: 0;
}
```

## Grid 布局

```css
/* ✅ 使用 Grid 进行复杂布局 */
.dashboard {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}

.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar content"
    "footer footer";
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
  gap: 0;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.content { grid-area: content; }
.footer { grid-area: footer; }
```

## 响应式设计

```css
/* ✅ 移动优先（Mobile First） */
.container {
  padding: 12px;
  font-size: 14px;
}

/* 平板 */
@media (min-width: 768px) {
  .container {
    padding: 16px;
    font-size: 16px;
  }
}

/* 桌面 */
@media (min-width: 1024px) {
  .container {
    padding: 24px;
    max-width: 1200px;
    margin: 0 auto;
  }
}

/* ✅ 使用统一的断点 */
/* 手机：< 768px */
/* 平板：768px - 1024px */
/* 桌面：> 1024px */

/* ✅ 使用容器查询（如果支持） */
@container (min-width: 400px) {
  .card {
    flex-direction: row;
  }
}
```

## 常用样式模式

### 卡片样式

```css
.card {
  padding: 16px;
  border-radius: 8px;
  background: var(--n-color);
  border: 1px solid var(--n-border-color);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  transition: all 0.3s ease;
}

.card:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
  transform: translateY(-2px);
}
```

### 文本省略

```css
/* ✅ 单行省略 */
.text-ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* ✅ 多行省略 */
.text-ellipsis-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

### 居中对齐

```css
/* ✅ Flexbox 居中 */
.center-flex {
  display: flex;
  align-items: center;
  justify-content: center;
}

/* ✅ Grid 居中 */
.center-grid {
  display: grid;
  place-items: center;
}

/* ✅ 绝对定位居中 */
.center-absolute {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

### 过渡和动画

```css
/* ✅ 使用 transition 实现平滑过渡 */
.button {
  background: var(--n-color-primary);
  transition: all 0.3s ease;
}

.button:hover {
  background: var(--n-color-primary-hover);
  transform: scale(1.05);
}

/* ✅ 使用 @keyframes 定义动画 */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-in {
  animation: fadeIn 0.3s ease-out;
}

/* ✅ 性能优化：只动画 transform 和 opacity */
.smooth-transition {
  transition: transform 0.3s ease, opacity 0.3s ease;
}
```

## 性能优化

```css
/* ✅ 使用 will-change 提示浏览器优化 */
.animated-element {
  will-change: transform, opacity;
}

/* ✅ 动画完成后移除 will-change */
.animated-element.animation-done {
  will-change: auto;
}

/* ✅ 避免使用昂贵的属性 */
/* 好：使用 transform */
.move {
  transform: translateX(100px);
}

/* 不好：使用 left/top */
.move {
  position: relative;
  left: 100px;
}

/* ✅ 使用 contain 优化渲染 */
.card {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jianxcao/watch-docker](https://github.com/jianxcao/watch-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
