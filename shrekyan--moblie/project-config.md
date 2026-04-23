---
trigger: always_on
description: 本项目是一个基于 React 18 + TypeScript 的现代化移动端 Web 应用，采用 Vite 作为构建工具，集成了丰富的前端技术栈，具备良好的可维护性和扩展性。
---

# 项目开发指南

## 1. 项目概述

本项目是一个基于 React 18 + TypeScript 的现代化移动端 Web 应用，采用 Vite 作为构建工具，集成了丰富的前端技术栈，具备良好的可维护性和扩展性。

## 2. 技术栈

### 核心技术
- **框架**: React 18.3.1 + TypeScript 5.5.3
- **构建工具**: Vite 6.0.5
- **状态管理**: MobX 6.13.5 + MobX React 9.1.1
- **路由**: React Router DOM 6.27.0 + react-activation
- **UI 组件库**: Ant Design Mobile 5.38.1
- **样式**: Sass 1.79.6 + PostCSS

### 功能技术
- **网络请求**: Axios 1.7.7
- **表单处理**: React Hook Form 7.54.2 + Zod 3.24.2
- **动画**: react-spring 9.7.5 + motion 12.0.1
- **工具库**: dayjs、crypto-js、qrcode、html2canvas 等
- **数据可视化**: ECharts 5.6.0

## 3. 项目结构

```
src/
├── api/              # API 接口定义
│   ├── config/       # API 配置
│   └── product/      # 业务模块 API
├── assets/           # 静态资源
│   ├── css/          # 全局样式
│   └── images/       # 图片资源
├── business/         # 业务逻辑
│   ├── channel/      # 渠道相关
│   ├── hooks/        # 业务 Hook
│   └── order/        # 订单相关
├── components/       # 组件
│   ├── business/     # 业务组件
│   ├── common/       # 通用组件
│   └── ui/           # UI 组件
├── core-tools/       # 核心工具
│   ├── http/         # HTTP 工具
│   ├── jsBridge/     # JSBridge
│   └── sdk/          # SDK 集成
├── pages/            # 页面组件
├── routes/           # 路由配置
├── types/            # TypeScript 类型定义
├── utils/            # 工具函数
├── App.tsx           # 应用入口组件
├── main.tsx          # 应用入口文件
└── styles.css        # 全局样式
```

## 4. 开发流程

### 4.1 环境配置

1. 安装依赖
```bash
npm install
```

2. 启动开发服务器
```bash
# 开发环境
npm run dev

# 测试环境
npm run test-dev

# 预发布环境
npm run pre-dev

# 生产环境
npm run prd-dev
```

3. 构建项目
```bash
# 开发环境构建
npm run dev-build

# 生产环境构建
npm run build
```

### 4.2 代码提交

项目使用 Husky 进行 Git 钩子管理，提交代码前会自动执行：
- ESLint 代码检查
- Stylelint 样式检查
- 提交信息规范检查

提交信息格式要求：
```
<type>(<scope>): <subject>

<body>

<footer>
```

常用 type：
- feat: 新功能
- fix: 修复 bug
- docs: 文档更新
- style: 代码格式调整
- refactor: 代码重构
- test: 测试相关
- chore: 构建或依赖更新

## 5. 代码规范

### 5.1 TypeScript 规范

1. **类型定义**
   - 为所有变量、函数参数和返回值添加类型
   - 使用接口（interface）定义对象类型
   - 使用类型别名（type）定义复杂类型组合

2. **命名规范**
   - 组件名：PascalCase（如 `UserProfile.tsx`）
   - 变量名：camelCase（如 `userInfo`）
   - 常量名：UPPER_CASE（如 `API_BASE_URL`）
   - 类型名：PascalCase（如 `UserType`）

3. **代码风格**
   - 使用 4 个空格缩进
   - 使用双引号
   - 每行不超过 120 个字符
   - 适当使用空行分隔代码块

### 5.2 React 组件规范

1. **组件类型**
   - 优先使用函数组件
   - 使用 React Hooks 管理组件状态和副作用

2. **组件结构**
   ```typescript
   // 导入
   import React, { useState, useEffect } from "react";
   import { Button } from "antd-mobile";
   
   // 类型定义
   interface Props {
     title: string;
     onClick: () => void;
   }
   
   // 组件
   const CustomButton: React.FC<Props> = ({ title, onClick }) => {
     // 状态管理
     const [loading, setLoading] = useState(false);
     
     // 副作用
     useEffect(() => {
       // 初始化逻辑
     }, []);
     
     // 事件处理
     const handleClick = async () => {
       setLoading(true);
       try {
         await onClick();
       } finally {
         setLoading(false);
       }
     };
     
     // 渲染
     return (
       <Button loading={loading} onClick={handleClick}>
         {title}
       </Button>
     );
   };
   
   export default CustomButton;
   ```

3. **Hooks 使用规范**
   - 只在函数组件或自定义 Hook 中使用 Hooks
   - 遵循 Hooks 的调用顺序
   - 使用 ESLint 插件确保 Hooks 使用正确

### 5.3 MobX 状态管理规范

1. **Store 结构**
   ```typescript
   import { makeObservable, observable, action, computed } from "mobx";
   
   class UserStore {
     // 可观察状态
     userInfo = null;
     loading = false;
     
     constructor() {
       makeObservable(this, {
         userInfo: observable,
         loading: observable,
         isLoggedIn: computed,
         setUserInfo: action,
         setLoading: action,
         fetchUserInfo: action
       });
     }
     
     // 计算属性
     get isLoggedIn() {
       return !!this.userInfo;
     }
     
     // 动作
     setUserInfo(userInfo) {
       this.userInfo = userInfo;
     }
     
     setLoading(loading) {
       this.loading = loading;
     }
     
     // 异步动作
     async fetchUserInfo() {
       this.setLoading(true);
       try {
         const response = await api.getUserInfo();
         this.setUserInfo(response.data);
       } catch (error) {
         console.error("Failed to fetch user info:", error);
       } finally {
         this.setLoading(false);
       }
     }
   }
   
   export default new UserStore();
   ```

2. **组件中使用 Store**
   ```typescript
   import React, { useEffect } from "react";
   import { observer } from "mobx-react";
   import userStore from "../stores/userStore";
   
   const UserProfile: React.FC = () => {
     useEffect(() => {
       userStore.fetchUserInfo();
     }, []);
     
     if (userStore.loading) {
       return <div>Loading...</div>;
     }
     
     return (
       <div>
         <h1>{userStore.userInfo?.name}</h1>
         <p>{userStore.userInfo?.email}</p>
       </div>
     );
   };
   
   export default observer(UserProfile);
   ```

### 5.4 API 调用规范

1. **API 配置**
   - 在 `src/api/config/` 中配置 API 基础 URL 和拦截器
   - 根据环境自动加载对应的环境变量

2. **API 定义**
   ```typescript
   // src/api/product/queryProductRate.ts
   import request from "../http";
   import { QueryProductRateParams, QueryProductRateResponse } from "../../types";
   
   /**
    * 查询产品利率
    * @param params 查询参数
    * @returns 产品利率数据
    */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShrekYan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
