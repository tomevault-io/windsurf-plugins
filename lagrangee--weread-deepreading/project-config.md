---
trigger: always_on
description: - **分层架构**: Content Script + Background Script + Popup，各层职责明确
---

# Chrome Extension Development Rules - WeRead Deep Reading Assistant

## 项目架构原则

### 核心架构模式
- **分层架构**: Content Script + Background Script + Popup，各层职责明确
- **事件驱动**: 使用EventUtils作为事件总线，实现松耦合通信
- **单例模式**: 核心服务类使用异步单例模式，确保唯一实例
- **桥接模式**: ContentBridge作为content和background的通信桥梁
- **组件化**: UI组件独立封装，支持生命周期管理

### 目录结构规范
```
src/
├── shared/           # 共享模块（CONFIG、工具类）
├── content/         # Content Script相关
│   ├── content.js   # 入口文件，负责初始化
│   ├── assistant-panel.js  # 主UI组件
│   ├── chat-service.js     # 聊天服务
│   └── content-bridge.js   # 通信桥梁
├── background/      # Background Script相关
│   ├── background.js       # Service Worker入口
│   ├── chat-service.js     # AI服务处理
│   └── settings-service.js # 设置管理
├── popup/          # Popup相关
└── utils/          # 工具函数
```

## 代码实现规范

### 1. 异步单例模式
```javascript
class ServiceName {
  static #instance = null;
  static #initPromise = null;

  static async getInstance() {
    if (this.#instance) return this.#instance;
    if (this.#initPromise) return this.#initPromise;
    
    this.#initPromise = this.#createInstance();
    this.#instance = await this.#initPromise;
    this.#initPromise = null;
    return this.#instance;
  }

  static async #createInstance() {
    const instance = new ServiceName();
    await instance.#init();
    return instance;
  }
}
```

### 2. 事件驱动通信
```javascript
// 发送事件
EventUtils.emit('event-name', data);

// 监听事件
EventUtils.on('event-name', (data) => {
  // 处理逻辑
});

// 一次性监听
EventUtils.once('event-name', callback);
```

### 3. Chrome扩展通信模式
```javascript
// Content -> Background
const response = await chrome.runtime.sendMessage({
  action: 'actionName',
  data: payload
});

// Background消息处理
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  handleMessage(message, sender).then(sendResponse);
  return true; // 异步响应
});
```

### 4. UI组件生命周期
```javascript
class UIComponent {
  constructor() {
    this.isInitialized = false;
    this.isDestroyed = false;
  }

  async init() {
    if (this.isInitialized) return;
    await this.#render();
    this.#bindEvents();
    this.isInitialized = true;
  }

  destroy() {
    if (this.isDestroyed) return;
    this.#unbindEvents();
    this.#cleanup();
    this.isDestroyed = true;
  }
}
```

## 具体实现指南

### Content Script开发
1. **入口文件模式**:
   - content.js只负责初始化和协调
   - 等待DOM ready后再初始化组件
   - 使用Promise确保初始化顺序

2. **组件初始化顺序**:
   ```javascript
   // 1. 初始化配置和工具
   window.CONFIG = CONFIG;
   
   // 2. 初始化通信桥梁
   const bridge = await ContentBridge.getInstance();
   window.contentBridge = bridge;
   
   // 3. 初始化UI组件
   const panel = await AssistantPanel.getInstance();
   ```

3. **UI组件设计**:
   - 支持浮动和内嵌两种模式
   - 实现拖拽、调整大小功能
   - 使用CSS变量管理主题
   - 支持键盘快捷键

### Background Script开发
1. **Service Worker模式**:
   - 使用Manifest V3规范
   - 避免使用document相关API
   - 正确处理异步操作

2. **消息路由模式**:
   ```javascript
   const handlers = {
     'chat': ChatService.handleMessage,
     'settings': SettingsService.handleMessage
   };
   
   async function handleMessage(message, sender) {
     const handler = handlers[message.action];
     return handler ? await handler(message, sender) : null;
   }
   ```

3. **AI服务集成**:
   - 支持多个AI服务商
   - 统一的请求/响应格式
   - 错误处理和重试机制

### 配置管理
1. **CONFIG结构**:
   ```javascript
   export const CONFIG = {
     ai: {
       providers: { /* 服务商配置 */ },
       prompts: { /* 提示词模板 */ }
     },
     ui: {
       themes: { /* 主题配置 */ },
       shortcuts: { /* 快捷键配置 */ }
     }
   };
   ```

2. **设置存储**:
   - 使用chrome.storage.local
   - 敏感信息加密存储
   - 支持设置导入导出

## 构建和部署

### Webpack配置要点
1. **多入口配置**:
   - 分别构建content、background、popup
   - 使用splitChunks优化vendor代码

2. **Babel配置**:
   ```javascript
   {
     presets: [['@babel/preset-env', {
       modules: 'auto',  // 重要：支持CommonJS转换
       targets: { chrome: "88" }
     }]]
   }
   ```

3. **文件复制**:
   - manifest.json、HTML、CSS文件
   - 图标资源文件

### 发布流程
1. **构建检查**:
   ```bash
   npm run build          # 生产构建
   npm run pre-publish    # 发布前检查
   npm run package        # 生成发布包
   ```

2. **必要文件**:
   - README.md (详细说明)
   - PRIVACY.md (隐私政策)
   - LICENSE (开源协议)
   - 截图素材 (1280x800)

## 错误处理规范

### 1. 统一错误处理
```javascript
class ErrorHandler {
  static handle(error, context) {
    console.error(`[${context}]`, error);
    // 用户友好的错误提示
    this.showUserError(this.getUserMessage(error));
  }
  
  static getUserMessage(error) {
    const messages = {
      'NetworkError': '网络连接失败，请检查网络设置',
      'AuthError': 'API密钥无效，请检查配置',
      'QuotaError': 'API调用次数已达上限'
    };
    return messages[error.name] || '操作失败，请重试';
  }
}
```

### 2. 异步操作错误处理
```javascript
try {
  const result = await apiCall();
  return result;
} catch (error) {
  ErrorHandler.handle(error, 'ChatService');
  throw error; // 重新抛出供上层处理
}
```

## 性能优化

### 1. 懒加载模式
- 组件按需初始化
- 大型依赖延迟加载
- 使用IntersectionObserver优化渲染

### 2. 内存管理
- 及时清理事件监听器
- 避免循环引用
- 使用WeakMap存储临时数据

### 3. 网络优化
- 请求去重和缓存
- 实现请求重试机制
- 使用AbortController取消请求

## 测试策略

### 1. 单元测试
- 工具函数测试
- 服务类方法测试
- 使用Jest框架

### 2. 集成测试
- 组件交互测试
- 消息通信测试
- Chrome API模拟

### 3. E2E测试
- 用户操作流程测试
- 跨浏览器兼容性测试

## 开发工具配置

### 1. ESLint规则
```javascript
{
  "extends": ["eslint:recommended"],
  "env": { "browser": true, "es2022": true },
  "globals": { "chrome": "readonly" },
  "rules": {
    "no-console": "warn",
    "prefer-const": "error",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lagrangee/weread_deepreading](https://github.com/lagrangee/weread_deepreading) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
