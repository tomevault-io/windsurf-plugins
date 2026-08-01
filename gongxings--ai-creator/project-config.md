---
trigger: always_on
description: 这是一个AI创作者平台，提供AI写作、图片生成、视频生成、PPT生成等创作工具，并支持一键发布到多个平台。
---

# AI创作者平台开发规则

## 项目概述
这是一个AI创作者平台，提供AI写作、图片生成、视频生成、PPT生成等创作工具，并支持一键发布到多个平台。

## 核心设计理念
1. **场景化工具**：针对不同创作场景提供专门工具，而不是通用聊天机器人
2. **一键生成**：用户只需提供核心信息，系统自动处理其他细节
3. **最少输入**：避免多轮对话，直接生成可用内容
4. **智能优化**：自动处理格式、排版、SEO、平台规则等
5. **所见即所得**：实时预览，直接可用

## 技术栈

### 前端
- Vue 3 + TypeScript
- Vite（构建工具）
- Element Plus（UI框架）
- Pinia（状态管理）
- Axios（HTTP客户端）
- Quill/TipTap（富文本编辑器）

### 后端
- FastAPI（Python 3.10+）
- SQLAlchemy（ORM）
- MySQL 8.0+（数据库）
- Redis（缓存和任务队列）
- Celery（异步任务）
- JWT（认证）

## 代码规范

### Python后端规范
1. 使用类型注解（Type Hints）
2. 遵循PEP 8代码风格
3. 使用async/await处理异步操作
4. API路由使用RESTful风格
5. 所有API返回统一的响应格式：
   ```python
   {
       "code": 200,
       "message": "success",
       "data": {}
   }
   ```
6. 错误处理使用HTTPException
7. 数据验证使用Pydantic模型
8. 敏感信息（API密钥）必须加密存储
9. 数据库操作使用事务
10. 日志记录使用Python logging模块

### Vue前端规范
1. 使用Composition API（setup语法糖）
2. 组件命名使用PascalCase
3. 文件命名使用kebab-case
4. 使用TypeScript类型定义
5. API调用统一封装在api目录
6. 状态管理使用Pinia
7. 路由懒加载
8. 组件按功能模块组织
9. 使用ESLint和Prettier格式化代码
10. 响应式设计，支持移动端

### 数据库规范
1. 表名使用复数形式，小写+下划线
2. 字段名使用小写+下划线
3. 主键统一使用id（BIGINT UNSIGNED）
4. 时间字段统一使用created_at、updated_at
5. 软删除使用deleted_at字段
6. 外键约束明确定义
7. 适当添加索引优化查询
8. 使用COMMENT注释表和字段
9. JSON字段用于存储灵活配置
10. 敏感数据加密存储

## 目录结构规范

### 后端目录
```
backend/
├── app/
│   ├── api/v1/          # API路由（按功能模块分文件）
│   ├── core/            # 核心配置（config, security, database）
│   ├── models/          # 数据库模型（每个表一个文件）
│   ├── schemas/         # Pydantic模型（请求/响应）
│   ├── services/        # 业务逻辑层
│   │   ├── ai/          # AI服务集成
│   │   ├── writing/     # 写作服务
│   │   ├── publish/     # 发布服务
│   │   └── platform/    # 平台集成
│   ├── utils/           # 工具函数
│   └── main.py          # 应用入口
```

### 前端目录
```
frontend/
├── src/
│   ├── api/             # API接口定义
│   ├── components/      # 公共组件
│   ├── views/           # 页面组件
│   │   ├── writing/     # 写作工具（14个子工具）
│   │   ├── image/       # 图片生成
│   │   ├── video/       # 视频生成
│   │   ├── ppt/         # PPT生成
│   │   ├── history/     # 历史记录
│   │   ├── publish/     # 发布管理
│   │   └── admin/       # 管理后台
│   ├── router/          # 路由配置
│   ├── store/           # 状态管理
│   └── utils/           # 工具函数
```

## 功能模块说明

### 1. AI写作工具（14个专业工具）
- 公众号文章创作
- 小红书笔记创作
- 公文写作
- 论文写作
- 营销文案
- 新闻稿/软文
- 短视频脚本
- 故事/小说创作
- 商业计划书
- 工作报告
- 简历/求职信
- 教案/课件
- 内容改写/扩写/缩写
- 多语言翻译

每个工具都应该：
- 提供简单的输入界面
- 一键生成功能
- 实时预览
- 自动优化（格式、排版、SEO等）
- 支持微调编辑
- 保存历史记录
- 支持导出和发布

### 2. 图片生成工具
- 文本生成图片
- 图片变体
- 图片编辑（局部重绘、风格迁移）
- 图片放大（AI超分辨率）
- 提示词优化
- 批量生成

### 3. 视频生成工具
- 文本生成视频
- 图片转视频
- AI配音
- 自动字幕
- 背景音乐
- 转场效果

### 4. PPT生成工具
- 主题生成PPT
- 大纲生成PPT
- 文档转PPT
- 在线编辑
- 自动配图
- 多种模板

### 5. 自动发布功能
支持平台：
- 微信公众号
- 小红书
- 抖音/快手
- 今日头条/百家号
- 知乎/简书

功能：
- 一键多平台发布
- 定时发布
- 内容自动适配
- 发布状态追踪
- 平台规则检测

### 6. 管理后台
- 用户管理
- AI模型管理
- 平台账号管理
- 内容审核
- 数据统计
- 系统设置

## API设计规范

### 认证相关
- POST /api/v1/auth/register - 用户注册
- POST /api/v1/auth/login - 用户登录
- POST /api/v1/auth/refresh - 刷新Token
- GET /api/v1/auth/me - 获取当前用户信息

### AI写作相关
- POST /api/v1/writing/{tool_type}/generate - 生成内容
- GET /api/v1/writing/tools - 获取所有写作工具列表
- POST /api/v1/writing/{creation_id}/regenerate - 重新生成
- POST /api/v1/writing/{creation_id}/optimize - 优化内容

### 图片生成相关
- POST /api/v1/image/generate - 文本生成图片
- POST /api/v1/image/variation - 图片变体
- POST /api/v1/image/edit - 图片编辑
- POST /api/v1/image/upscale - 图片放大

### 视频生成相关
- POST /api/v1/video/generate - 生成视频
- POST /api/v1/video/text-to-video - 文本转视频
- POST /api/v1/video/image-to-video - 图片转视频
- GET /api/v1/video/{task_id}/status - 获取生成状态

### PPT生成相关
- POST /api/v1/ppt/generate - 生成PPT
- POST /api/v1/ppt/from-outline - 大纲生成PPT
- GET /api/v1/ppt/{ppt_id}/download - 下载PPT

### 创作记录相关
- GET /api/v1/creations - 获取创作列表
- GET /api/v1/creations/{id} - 获取创作详情
- PUT /api/v1/creations/{id} - 更新创作内容
- DELETE /api/v1/creations/{id} - 删除创作
- GET /api/v1/creations/{id}/versions - 获取版本历史

### 发布管理相关
- POST /api/v1/publish - 发布内容
- GET /api/v1/publish/platforms - 获取支持的平台列表
- POST /api/v1/publish/platforms/bind - 绑定平台账号
- GET /api/v1/publish/history - 获取发布历史
- GET /api/v1/publish/{id}/status - 获取发布状态

### AI模型管理相关
- GET /api/v1/models - 获取AI模型列表
- POST /api/v1/models - 添加AI模型配置
- PUT /api/v1/models/{id} - 更新AI模型配置
- DELETE /api/v1/models/{id} - 删除AI模型配置

### 管理后台相关
- GET /api/v1/admin/users - 获取用户列表
- PUT /api/v1/admin/users/{id} - 更新用户信息
- GET /api/v1/admin/statistics - 获取统计数据
- GET /api/v1/admin/system/config - 获取系统配置
- PUT /api/v1/admin/system/config - 更新系统配置

## 开发流程

### 1. 环境搭建
1. 安装Python 3.10+和Node.js 18+
2. 安装MySQL 8.0+和Redis
3. 克隆项目代码
4. 安装后端依赖：`pip install -r requirements.txt`
5. 安装前端依赖：`npm install`
6. 配置环境变量（.env文件）
7. 初始化数据库：`python scripts/init_db.py`

### 2. 开发顺序
1. 数据库设计和模型创建
2. 后端API开发（按模块）
3. 前端页面开发（按功能）
4. AI服务集成
5. 平台发布功能集成
6. 测试和优化

### 3. 测试规范
- 单元测试覆盖率 > 80%
- 使用pytest进行后端测试
- 使用Vitest进行前端测试
- API测试使用Postman/Thunder Client
- 性能测试使用Locust

### 4. 部署规范
- 使用Docker容器化部署
- Nginx作为反向代理
- 使用Supervisor管理进程
- 配置SSL证书（HTTPS）
- 设置日志轮转
- 配置自动备份

## 安全规范

1. **认证安全**
   - 使用JWT Token认证
   - Token过期时间：访问令牌2小时，刷新令牌7天
   - 密码使用bcrypt加密
   - 实施登录失败次数限制

2. **数据安全**
   - API密钥加密存储
   - 敏感数据传输使用HTTPS
   - 数据库连接使用SSL
   - 定期备份数据

3. **API安全**
   - 实施请求频率限制（Rate Limiting）
   - 输入验证和过滤
   - SQL注入防护
   - XSS攻击防护
   - CSRF防护

4. **文件安全**
   - 上传文件类型限制
   - 文件大小限制
   - 文件扫描（病毒检测）
   - 安全的文件存储路径

## 性能优化

1. **数据库优化**
   - 合理使用索引
   - 查询优化（避免N+1问题）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gongxings/ai-creator](https://github.com/gongxings/ai-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
