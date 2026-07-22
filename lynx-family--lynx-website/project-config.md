---
trigger: always_on
description: > 面向已经熟悉 Web 前端技术栈的开发者，解释 Lynx 的核心理念、与 Web 差异、以及如何用 Lynx 构建跨平台应用。
---

# Lynx 开发前必读

> 面向已经熟悉 Web 前端技术栈的开发者，解释 Lynx 的核心理念、与 Web 差异、以及如何用 Lynx 构建跨平台应用。

---

> [!IMPORTANT] > **非常重要：**
>
> 此文档在一些陈述之后提供了参考链接，指向 Lynx 官方文档的相关章节，请**主动**阅读以获得更全面的理解。

---

## 1. Lynx 的定位与能力边界

- **What**：Lynx 是一个以 Web 技术为语义参照的跨平台渲染引擎，目标是在一套代码内覆盖 iOS、Android、HarmonyOS 以及 Web。它通过统一的元件抽象在不同宿主上创建原生视图或 Web 自定义元件，避免传统 WebView 的性能瓶颈。（参考：[Composing Elements](/zh/guide/ui/elements-components.md)）
- **Why**：移动端用户极度敏感首屏时间与交互延迟。Lynx 通过双线程 JavaScript 运行时、即时首帧渲染（Instant First-Frame Rendering, IFR）和原生渲染管线，让 React 开发体验与近原生性能同时存在。（参考：[Instant First-Frame Rendering](/zh/guide/interaction/ifr.md)）
- **How**：在工程侧使用 Rspeedy（Rspack 驱动的构建工具）生成 Lynx Bundle，前端层采用 ReactLynx（Preact 内核的 React 实现）来描述 UI，与宿主端通过 Native Modules / Custom Elements 互通。（参考：[ReactLynx](/zh/react/introduction.md), [Native Modules](/zh/guide/use-native-modules.md)）

## 2. 心智模型：把 Lynx 对齐到 Web

| Web 心智               | Lynx 对应物                                                           | 差异重点                                                                                                                                                                                                                         |
| ---------------------- | --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `index.html` + 资源    | Lynx Bundle（二进制文件，包含 JS 字节码 + 样式）或 `template.js` 文件 | Bundle 需与 Lynx Engine 版本兼容，需设置 `engineVersion`（历史上曾称 `targetSdkVersion`）。（参考：[Compatibility](/zh/guide/compatibility.md)）                                                                                 |
| DOM + CSSOM            | 元件树（Element Tree）+ 样式系统                                      | 所有元件都类比 block-level，`view`/`text` 等自定义标签映射到原生控件。（参考：[Composing Elements](/zh/guide/ui/elements-components.md)）                                                                                        |
| 浏览器主线程           | Lynx Main Thread                                                      | 负责首屏渲染、布局、主线程脚本，运行 PrimJS 字节码。（参考：[Main Thread Runtime](/zh/guide/scripting-runtime/main-thread-runtime.md)）                                                                                          |
| 浏览器渲染进程任务队列 | Lynx Background Thread                                                | 执行 ReactLynx 调度、生命周期、绝大部分副作用。运行 PrimJS/JavaScriptCore，语法最高仅到 ES2015（构建期经 SWC 转译）。（参考：[JavaScript Runtime](/zh/guide/scripting-runtime/index.md)）                                        |
| `window`/`document`    | `lynx` 全局对象 + API 集                                              | 无 DOM API；通过 `lynx.getElementById`、SelectorQuery、`main-thread:ref` 等访问节点。（参考：[ReactLynx](/zh/react/introduction.md), [Direct Manipulation](/zh/guide/interaction/event-handling/manipulating-element.react.md)） |

## 3. 运行时架构：双线程 React 的工作方式

- **双线程并行渲染**：首屏时主线程直接执行 ReactLynx 渲染输出像素，同时后台线程构建完整节点树并与主线程状态对齐，避免白屏。（参考：[Rendering Process and Lifecycle](/zh/react/lifecycle.md), [IFR](/zh/guide/interaction/ifr.md)）
- **“你的代码运行在两个线程”**：双线程 React 意味着你的代码会在两个线程中运行。但并不是所有的代码都能在两个线程运行——一些 API 仅在后台线程提供；ReactLynx 也只会从后台线程执行事件、生命周期和 `useEffect` 等副作用。
- **`'background only'`**：无需运行在主线程（如事件处理、生命周期、副作用等）、或使用了仅在后台线程提供的 API 的函数，必须在函数体首行添加 `'background only'` 指示符；模块会通过 `import "background-only"` 表达自己预期只运行在后台线程。
- **主线程脚本（MTS）**：通过 `main-thread` 指示符标注的函数可以直接在主线程响应事件，常用于高频动画、手势和无延迟反馈。主线程事件需要用 `main-thread:` 前缀标识，使用 `main-thread:bindtap`、`useMainThreadRef` 等 API。（参考：[Main Thread Script](/zh/react/main-thread-script.md)）
- **跨线程通信**：使用 `runOnMainThread` / `runOnBackground` 进行异步调用，参数需可 JSON 序列化。

## 4. UI 构建：元件体系与原生映射

- **元件标签**：`view`、`text`、`image`、`scroll-view` 等内置标签抽象原生控件。与 HTML 不同，标签不是标准 DOM，但语法（开始标签/结束标签/属性）保持类 HTML 习惯。（参考：[Composing Elements](/zh/guide/ui/elements-components.md)）
- **跨平台映射**：同一元件自动映射到平台原生视图（如 `view` → iOS `UIView`、Android `ViewGroup`；Web → 自定义元件），无需手动分平台写代码。
- **文本语义**：必须使用 `text` 元件承载文字，不能像 Web 在 `div` 里直接写文本。行内布局依赖嵌套 `text`。（参考：[Typography](/zh/guide/styling/text-and-typography.md)）
- **可扩展性**：当内置元件不足，可注册自定义原生元件，分别在 iOS/Android/Harmony 实现，前端通过统一标签使用。（参考：[Custom Element](/zh/guide/custom-native-component.md)）

此外：

- `svg` 元件和 Web 中存在较大差异，直接将 SVG 内容作为 `content` 属性或将 SVG URL 作为 `src` 属性传递给 `<svg />` 元件：

  ```jsx
  <svg content={`<svg ... />`} />;
  // or
  <svg src={urlOfYourSvgFile} />;
  ```

## 5. 布局系统：默认 Block，支持四种布局模式

| 布局模式            | Web 对应                      | Lynx 差异                                                                                                                                          |
| ------------------- | ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `display: linear`   | 无直接对应，近似更简化的 Flex | 主轴默认竖向，可用 `linear-direction`、`linear-weight` 控制尺寸分配，适合线性排布。（参考：[Linear Layout](/zh/guide/ui/layout/linear-layout.md)） |
| `display: flex`     | CSS Flexbox                   | 绝大部分属性一致，但当前不支持 `min-content`，子项收缩下限视作 `0px`。（参考：[Flexible Box Layout](/zh/guide/ui/layout/flexible-box-layout.md)）  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lynx-family/lynx-website](https://github.com/lynx-family/lynx-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
