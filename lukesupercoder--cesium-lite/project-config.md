---
trigger: always_on
description: 本文件为 Claude Code (claude.ai/code) 在此代码库中工作时提供指导。
---

# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此代码库中工作时提供指导。

## 项目概述

CesiumLite 是一个基于 Cesium.js 的轻量级封装库，为常见的三维地图功能提供简化的 API。它被设计为可在多个前端项目中复用的开箱即用的地图功能模块。

**主要语言**: JavaScript (ES6+)
**构建工具**: Vite 2.5.6
**核心依赖**: Cesium 1.127.0

## 常用命令

### 开发
```bash
npm run dev          # 启动开发服务器，运行在 8020 端口
```

### 构建与部署
```bash
npm run build        # 构建生产版本到 dist/ 目录
npm run serve        # 构建并在本地预览
npm run deploy       # 构建、修复路径并部署到 GitHub Pages
npm run fix-paths    # 修复 GitHub Pages 部署时的 Cesium 资源路径
```

### 测试
当前项目未配置测试框架。

## 开发辅助命令（Claude Code Skills）

项目配置了自定义 AI 辅助命令（Skills），帮助提升开发效率。这些命令通过 Claude Code 调用。

### `/plugin-analysis` - 插件需求分析工具

**功能**：对地图功能插件进行系统化的需求分析，生成结构化的需求文档。

**使用场景**：在开发新功能插件前，使用此命令进行全面的需求分析和技术设计规划。

**使用方式**：
```bash
/plugin-analysis <功能插件名称>
```

**示例**：
```bash
/plugin-analysis 地形裁剪功能
/plugin-analysis 三维测量工具
/plugin-analysis 图层切换器
/plugin-analysis 飞行漫游
```

**工作流程**：
1. **基础信息收集** - 通过交互式问答收集插件的基本信息（功能概述、应用场景、功能范围）
2. **技术设计分析** - 分析技术可行性、依赖的 Cesium API、性能考量、技术风险
3. **API 设计规划** - 设计类结构、公共方法、配置选项、事件回调
4. **实现方案设计** - 规划核心实现思路、状态管理、资源管理、错误处理
5. **示例页面规划** - 设计演示页面的 UI 和功能点
6. **开发规划** - 制定开发步骤、评估工作量、识别风险挑战

**输出文档**：
- 保存路径：`docs/需求分析/[插件名称]-需求分析.md`
- 文档包含：功能概述、应用场景、技术方案、API 设计、开发规划、验收标准等完整章节

**使用建议**：
- ✅ **开发前必用**：在开始编码前先进行需求分析，避免设计缺陷
- ✅ **团队协作**：需求文档可作为技术评审和团队讨论的依据
- ✅ **指导开发**：按照需求文档进行开发，确保实现完整性
- ✅ **文档留存**：作为项目知识库的一部分，便于后续维护

---

### `/juejin-blog` - 掘金技术博客生成器

**功能**：为 CesiumLite 项目的功能模块自动生成掘金风格的技术博客文章。

**使用场景**：功能开发完成后，使用此命令生成技术博客用于推广和介绍项目功能。

**使用方式**：
```bash
/juejin-blog <功能模块名称>
```

**示例**：
```bash
/juejin-blog 图层切换器模块
/juejin-blog 地形裁剪功能
/juejin-blog 三维分析工具
```

**工作流程**：
1. **信息收集** - 询问模块的基本信息、源码路径、示例页面路径
2. **代码分析** - 阅读模块源码，理解核心实现和 API 设计
3. **内容创作** - 按照掘金博客模板编写文章（痛点分析 → 解决方案 → 使用教程）
4. **文档保存** - 保存到 `docs/juejin/` 目录

**输出文档**：
- 保存路径：`docs/juejin/CesiumLite-[模块名称]介绍.md`
- 文章结构：标题、摘要、前言、在线演示、痛点分析、解决方案、核心代码、使用教程、快速开始、总结

**博客特点**：
- 突出开发痛点和解决方案对比
- 包含完整的代码示例和使用教程
- 提供在线演示链接和项目地址
- 符合掘金平台的写作风格和排版规范

**使用建议**：
- ✅ **功能完成后生成**：确保功能已开发完成并测试通过
- ✅ **提供清晰的示例**：示例页面应包含完整的功能演示
- ✅ **强调实用性**：突出功能的实际应用价值
- ✅ **配合需求文档**：可结合 `/plugin-analysis` 生成的需求文档进行内容创作

---

### 推荐工作流程

完整的功能开发流程应该是：

```mermaid
graph LR
    A[需求分析] --> B[开发实现] --> C[测试验证] --> D[文档推广]
    A1[/plugin-analysis] --> A
    B1[参考需求文档编码] --> B
    C1[创建示例页面] --> C
    D1[/juejin-blog] --> D
```

**具体步骤**：

1. **规划阶段** 📋
   ```bash
   # 创建功能分支
   git checkout -b feature-xxx

   # 执行需求分析
   /plugin-analysis 功能名称

   # 查看生成的需求文档
   # docs/需求分析/功能名称-需求分析.md
   ```

2. **开发阶段** 💻
   ```bash
   # 按照需求文档创建插件文件
   # src/[模块分类]/[文件名].js

   # 在 src/index.js 中集成模块

   # 创建示例页面
   # examples/[示例名].html

   # 更新 vite.config.js 添加示例入口
   ```

3. **测试阶段** 🧪
   ```bash
   # 启动开发服务器
   npm run dev

   # 在浏览器中测试示例页面
   # http://localhost:8020/examples/[示例名].html
   ```

4. **发布阶段** 🚀
   ```bash
   # 生成技术博客
   /juejin-blog 功能名称

   # 查看生成的博客文章
   # docs/juejin/CesiumLite-功能名称介绍.md

   # 提交代码
   git add .
   git commit -m "feat: 添加[功能名称]"
   git push origin feature-xxx
   ```

---

### Skills 配置文件

自定义 Skills 的配置文件位于：
- `.claude/skills/plugin-analysis.md` - 需求分析工具配置
- `.claude/skills/juejin-blog.md` - 技术博客生成器配置

如需自定义 Skills 行为，可直接修改这些配置文件。

## 开发规范

### Git 分支管理
开发新功能模块插件时，必须先创建新的 Git 分支：
```bash
# 功能开发分支命名规范
git checkout -b feature-xxx

# 示例
git checkout -b feature-layer-switcher     # 图层切换器功能
git checkout -b feature-3d-analysis        # 三维分析功能
git checkout -b feature-terrain-clip       # 地形裁剪功能
```

### 插件开发规范
开发插件时必须遵循**类的写法**，参考现有插件的架构模式：

#### 1. 创建管理类
在对应的功能目录下创建管理类文件，例如：

```javascript
// src/newModule/newFeature.js
import { Viewer } from 'cesium';

/**
 * 新功能管理类
 * 功能描述...
 */
class NewFeature {
    /**
     * 构造函数
     * @param {Viewer} viewer - Cesium viewer 实例
     * @param {Object} options - 配置选项
     */
    constructor(viewer, options = {}) {
        if (!viewer) throw new Error('Viewer instance is required');
        this.viewer = viewer;

        // 默认配置
        this.defaultOptions = {
            // 默认选项...
            ...options
        };

        // 初始化内部状态
        this._internalState = new Map();
    }

    /**
     * 公开方法
     */
    publicMethod(params) {
        // 实现逻辑...
    }

    /**
     * 清理方法
     */
    destroy() {
        // 清理资源...
    }
}

export default NewFeature;
```

#### 2. 在主入口引入并实例化
在 [src/index.js](src/index.js) 中导入新模块并实例化：

```javascript
// 1. 导入模块
import NewFeature from './newModule/newFeature';

// 2. 在构造函数中实例化
constructor(containerId, options={}) {
    // ... 其他初始化代码

    // 初始化新功能模块
    this.newFeature = new NewFeature(this.mapCore.viewer, this.options.map.newFeature);
}
```

#### 3. 参考现有插件写法
- **图层管理类**：参考 [src/layers/staticFileLayer.js](src/layers/staticFileLayer.js) 或 [src/layers/vectorTileLayer.js](src/layers/vectorTileLayer.js)
  - 使用 `Map()` 存储实例
  - 提供 `addXxx()`, `removeXxx()`, `clearAll()` 等方法
  - 支持 ID 管理和配置合并

- **控件类**：参考 [src/controls/zoomControl.js](src/controls/zoomControl.js)
  - 提供 `.show()`, `.hide()`, `.setPosition()` 方法
  - 使用 DOM 操作创建 UI 元素

- **工具类**：参考 [src/mark/draw.js](src/mark/draw.js)
  - 使用 `ScreenSpaceEventHandler` 处理交互
  - 提供回调函数机制
  - 实现资源清理方法

### 示例页面规范
开发完插件后，**必须**在 [examples/](examples/) 文件夹创建对应的示例页面：

#### 1. 创建示例 HTML 文件
```bash
# 在 examples/ 目录下创建
examples/newFeature.html
```

#### 2. 示例页面模板
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukeSuperCoder/cesium-lite](https://github.com/lukeSuperCoder/cesium-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
