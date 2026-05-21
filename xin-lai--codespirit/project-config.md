---
trigger: always_on
description: CodeSpirit JavaScript 开发规范 - AMIS集成、模块模式、API请求、Token管理
---


# JavaScript 开发规范

## 模块模式

### IIFE 包装

所有 JS 文件使用立即调用函数表达式（IIFE）包装，启用严格模式：

```javascript
/**
 * 模块说明
 * @module ModuleName
 */
(function() {
    'use strict';
    
    // 模块代码
    
    // 导出到全局
    window.ModuleName = {
        // 公共 API
    };
})();
```

### 命名空间导出

全局对象使用 `window` 命名空间导出：

```javascript
// ✅ 正确：使用 window 命名空间
window.TokenManager = (function() {
    'use strict';
    
    function getToken() { /* ... */ }
    
    return {
        getToken,
        setToken,
        clearToken
    };
})();

// ✅ 正确：使用 CodeSpirit 命名空间
window.CodeSpirit = window.CodeSpirit || {};
window.CodeSpirit.i18n = {
    t: function(key, params) { /* ... */ }
};

// ✅ 正确：ES6 类导出
class NotificationClient {
    constructor(hubUrl = '/notification-hub') {
        this.hubUrl = hubUrl;
    }
}
window.NotificationClient = NotificationClient;
```

## 文档注释规范

### 文件头注释

```javascript
/**
 * 考试系统API请求管理器
 * 负责处理API地址转换和统一的请求处理
 * @module ExamApiManager
 * @version 2.0.0
 * @author CodeSpirit Team
 */
```

### 函数注释（JSDoc）

```javascript
/**
 * 设置认证token
 * @param {string} token - 访问token
 * @param {number} [expiryInHours=24] - 过期时间（小时）
 * @returns {void}
 * @throws {Error} 当token为空时抛出错误
 */
function setToken(token, expiryInHours = 24) {
    if (!token || typeof token !== 'string') {
        throw new Error('Token must be a non-empty string');
    }
    // ...
}

/**
 * 统一的API请求函数
 * @param {string} url - API路径
 * @param {Object} [options={}] - fetch选项
 * @returns {Promise<Object>} API响应数据
 * @example
 * const data = await ExamApiManager.request('/exam/api/questions', { method: 'GET' });
 */
async function request(url, options = {}) {
    // ...
}
```

## AMIS 框架集成

### 主题配置

项目使用 **antd** 主题 [[memory:8912919]]：

```javascript
// 初始化 AMIS
let amisScoped = amis.embed('#root', amisJSON, {
    location: history.location,
    data: {},
    context: {
        WEB_HOST: webHost
    }
}, { 
    theme: 'antd'  // 必须使用 antd 主题
});
```

### 事件系统

使用 `onEvent` 配置事件监听：

```javascript
{
    type: 'form',
    api: '/identity/api/identity/auth/login',
    onEvent: {
        // 表单提交成功事件
        submitSucc: {
            actions: [
                {
                    actionType: 'custom',
                    script: `
                        const token = event.data.token;
                        TokenManager.setToken(token, 24);
                        window.location.href = '/';
                    `
                }
            ]
        },
        // 数据初始化完成事件
        fetchInited: {
            actions: [
                {
                    actionType: 'custom',
                    script: 'window.fetchUnreadNotificationCount();'
                }
            ]
        }
    }
}
```

### 行为类型

优先使用 AMIS 内置行为类型（actionType）：

```javascript
// ✅ 核心行为
{ actionType: 'ajax', api: 'POST:/api/submit' }
{ actionType: 'link', link: '/dashboard' }
{ actionType: 'dialog', dialog: { /* ... */ } }
{ actionType: 'reload', target: 'crud' }
{ actionType: 'copy', content: '${text}' }

// ✅ 表单行为
{ actionType: 'submit' }
{ actionType: 'reset' }
{ actionType: 'clear' }

// ✅ 自定义脚本（仅在必要时使用）
{
    actionType: 'custom',
    script: `
        const tenantId = event.data.tenantId;
        window.location.href = '/' + tenantId + '/login';
    `
}
```

### 请求适配器

使用 `requestAdaptor` 和 `adaptor` 处理请求和响应：

```javascript
api: {
    method: 'post',
    url: '/identity/api/identity/auth/login',
    
    // 请求适配器 - 添加认证头
    requestAdaptor: function(api) {
        const token = TokenManager.getToken();
        api.headers = api.headers || {};
        api.headers['Authorization'] = token ? 'Bearer ' + token : '';
        api.headers['X-Forwarded-With'] = 'CodeSpirit';
        api.headers['X-Tenant-Id'] = window.tenantId || 'system';
        return api;
    },
    
    // 响应适配器 - 处理响应数据
    adaptor: function(payload, response, api) {
        if (response.status === 401) {
            window.location.href = '/login';
            return { msg: '登录过期！' };
        }
        
        if (payload.status === 0 && payload.data) {
            TokenManager.setToken(payload.data.token, 24);
        }
        
        return payload;
    }
}
```

## Token 管理

### TokenManager 使用

使用 `TokenManager` 统一管理认证状态：

```javascript
// 初始化模式
TokenManager.initSystemMode();           // 系统平台
TokenManager.initTenantMode('tenant-id'); // 租户平台
TokenManager.initClientMode('tenant-id', 'exam'); // 客户端平台

// Token 操作
TokenManager.setToken('access-token', 24);     // 设置 token（24小时过期）
const token = TokenManager.getToken();          // 获取 token
TokenManager.clearToken();                      // 清除 token
TokenManager.hasToken();                        // 检查是否有 token
TokenManager.isTokenExpired();                  // 检查是否过期
TokenManager.isAuthenticated();                 // 检查是否已认证

// 扩展功能
TokenManager.setTokenExtended(accessToken, refreshToken, expiresIn, tenantId);
TokenManager.getRefreshToken();
TokenManager.getAuthHeaders();                  // 获取认证请求头
TokenManager.setUserInfo(userInfo);
TokenManager.getUserInfo();
```

### 认证请求头

所有 API 请求必须携带认证头：

```javascript
const headers = {
    'Authorization': token ? 'Bearer ' + token : '',
    'X-Forwarded-With': 'CodeSpirit',
    'X-Tenant-Id': tenantId || 'system',
    'Content-Type': 'application/json'
};
```

## API 请求规范

### 服务发现路径


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
