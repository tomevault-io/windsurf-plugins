---
trigger: always_on
description: - 审核报告：`docs/MOBILE_AUDIT_REPORT.md`
---

# Mobile-First Mini App 开发规范

## 设计标准

- 审核报告：`docs/MOBILE_AUDIT_REPORT.md`
- 改造路线图：`docs/MOBILE_FIRST_ROADMAP.md`
- 现有移动端 UX 指南：`.cursor/skills/mobile-ux-guide/SKILL.md`

## Platform View 模式（核心规范）

### 同路由分视图

操作型关键页面（商品详情、搜索、购物车）**必须**拆分为独立的 Desktop/Mobile 视图。
浏览型页面（政策页、404、订单确认）使用 Tailwind 响应式即可。

```tsx
// ✅ 正确：thin shell 路由页面
export default function ProductPage({ params }) {
  const { shouldUseMobileView } = usePlatform();
  return shouldUseMobileView
    ? <ProductDetailMobile slug={params.slug} />
    : <ProductDetailDesktop slug={params.slug} />;
}

// ❌ 错误：单文件内 1500 行用 48 个断点硬撑
export default function ProductPage() {
  return <ProductDetail />; // 不拆分
}
```

### 视图拆分三件套

每个 Platform View 页面必须包含：

```
components/Xxx/
├── XxxDesktop.tsx          ← 桌面交互范式
├── XxxMobile.tsx           ← 移动交互范式
└── useXxx.ts               ← 共享业务逻辑（100% 复用）
```

**严格禁止**：Mobile 视图内包含业务逻辑（API 调用、状态计算）。
业务逻辑**全部**放在共享 hook 中，视图层只负责 UI。

### 命名约定

| 类型 | 命名 | 示例 |
|------|------|------|
| Desktop 视图 | `XxxDesktop.tsx` | `ProductDetailDesktop.tsx` |
| Mobile 视图 | `XxxMobile.tsx` | `ProductDetailMobile.tsx` |
| 共享 Hook | `useXxx.ts` | `useProductDetail.ts` |
| TG 增强 Hook | `useTGXxx.ts` | `useTGMainButton.ts` |
| 底部 Sheet | `XxxSheet.tsx` | `FilterSheet.tsx` |

## 移动端 UI 规范

### 触控目标

```tsx
// ✅ 正确：所有可交互元素 ≥ 44×44px
<button className="min-h-[44px] min-w-[44px] ...">

// ❌ 错误：按钮太小
<button className="h-6 w-6 ...">
```

### 底部固定操作栏

操作型页面必须有底部固定操作栏，且处理安全区：

```tsx
// ✅ 正确
<div className="fixed bottom-0 inset-x-0 bg-background border-t p-3
  pb-[max(0.75rem,env(safe-area-inset-bottom))]">
  <Button className="w-full min-h-[44px]">Add to Cart</Button>
</div>

// ❌ 错误：操作按钮在页面中间，滚动后不可见
<Button>Add to Cart</Button>
```

### 商品网格

```tsx
// ✅ 移动端 2 列
<div className="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 gap-3 md:gap-4">

// ❌ 移动端 1 列（浪费空间）
<div className="grid grid-cols-1 sm:grid-cols-2">
```

### 信息折叠

移动端长内容使用 Accordion 折叠，减少滚动深度：

```tsx
// ✅ 移动端折叠
<Accordion type="single" collapsible>
  <AccordionItem value="description">
    <AccordionTrigger>Description</AccordionTrigger>
    <AccordionContent>{description}</AccordionContent>
  </AccordionItem>
  <AccordionItem value="shipping">...</AccordionItem>
  <AccordionItem value="reviews">...</AccordionItem>
</Accordion>

// ❌ 全部展开（移动端需无限滚动）
<div>{description}</div>
<div>{shipping}</div>
<div>{reviews}</div>
```

### 筛选面板

移动端筛选使用底部 Sheet，不使用侧边栏或内联展开：

```tsx
// ✅ 移动端底部 Sheet
<BottomSheet open={filterOpen} onOpenChange={setFilterOpen}>
  <FilterContent />
</BottomSheet>

// ❌ 移动端侧边栏筛选
<aside className="w-64">{/* 移动端空间不够 */}</aside>
```

## Mini App 导航规范

### MobileNav / MobilePageHeader 按平台显隐

```tsx
// ✅ 正确：TG/Discord 环境隐藏 MobileNav（宿主 App 有自己的底栏）
function MobileNav() {
  const { isTGMiniApp, isDiscordActivity } = usePlatform();
  if (isTGMiniApp || isDiscordActivity) return null;
  return <nav className="fixed bottom-0 ...">...</nav>;
}

// ✅ 正确：TG 环境隐藏返回箭头（用 BackButton 替代），保留标题
function MobilePageHeader({ title, onBack, actions }) {
  const { isTGMiniApp } = usePlatform();
  return (
    <header>
      {!isTGMiniApp && <BackArrow onClick={onBack} />}
      <span>{title}</span>
      {actions}
    </header>
  );
}

// ❌ 错误：所有环境统一显示 MobileNav + MobilePageHeader
```

### Toast/Snackbar 定位

```tsx
// ✅ 正确：Mini App 中 Toast 在顶部（底部有 MainButton）
const toastPosition = isTGMiniApp ? 'top-center' : 'bottom-center';

// ✅ TG 环境优先使用原生 showPopup/showAlert
if (isTGMiniApp && window.Telegram?.WebApp?.showPopup) {
  window.Telegram.WebApp.showPopup({
    title: 'Confirm',
    message: 'Remove this item?',
    buttons: [
      { type: 'cancel' },
      { id: 'confirm', type: 'destructive', text: 'Remove' },
    ],
  });
} else {
  // 降级到自定义 Modal
}
```

## Mini App 增强规范

### TG MainButton 集成

关键 CTA（加购、结账、确认收货）在 TG 环境用 MainButton 替代底部操作栏：

```tsx
// ✅ 条件渲染
const { isAvailable: isTG } = useTGMiniApp();

return (
  <>
    {/* 页面内容 */}
    {!isTG && <BottomActionBar />}
  </>
);

// ✅ MainButton 通过 useEffect 绑定（在 Mobile View 内部）
useEffect(() => {
  if (isTG && mainButton) {
    mainButton.setText('Add to Cart');
    mainButton.show();
    mainButton.onClick(handleAddToCart);
    return () => { mainButton.offClick(handleAddToCart); mainButton.hide(); };
  }
}, [isTG]);
```

### BackButton 集成

所有非首页的移动端页面在 TG 环境绑定 BackButton：

```tsx
// ✅ 在 MobilePageHeader 或各 Mobile View 内统一处理
useEffect(() => {
  if (isTG && backButton) {
    backButton.show();
    backButton.onClick(() => router.back());
    return () => { backButton.offClick(() => router.back()); backButton.hide(); };
  }
}, [isTG]);
```

### HapticFeedback

在以下场景触发触觉反馈：

| 场景 | 类型 |
|------|------|
| 加购成功 | `notificationOccurred('success')` |
| 支付确认 | `notificationOccurred('success')` |
| 滑动删除 | `impactOccurred('medium')` |
| 表单错误 | `notificationOccurred('error')` |

### 渐进增强层次

```
Layer 0：Mobile Web（基线，所有环境可用）
  └── 底部操作栏、MobilePageHeader、MobileNav
Layer 1：TG Mini App（增强层）
  └── MainButton 替代底部 CTA
  └── BackButton 替代返回按钮
  └── HapticFeedback
  └── themeParams 主题同步
  └── shareUrl 原生分享
Layer 2：Discord Activity（增强层）
  └── SDK 认证
  └── 主题同步
  └── Activity 生命周期
```

### 主题冲突处理

Mobazha 有店铺主题系统，TG/Discord 也推送宿主主题。遵循以下优先级：

```
TG Mini App 内：
  背景色 / 基础文本色 → 使用 TG themeParams（视觉融合）
  品牌色 / CTA 按钮色 → 保留店铺主题色（品牌辨识度）
  MainButton 颜色 → TG 控制（无法自定义）

CSS 变量策略：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mobazha/mobazha-unified](https://github.com/mobazha/mobazha-unified) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
