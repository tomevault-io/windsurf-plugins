---
trigger: always_on
description: Vige 是一个 FastAPI + Vue 的全栈工程模板，采用前后端分离架构：
---

# Vige 项目开发规则

## 📋 目录索引
- [项目概述](#项目概述)
- [后端开发规范](#后端开发规范)
- [前端开发规范](#前端开发规范)
- [代码示例模板](#代码示例模板)
- [开发注意事项](#开发注意事项)
- [禁止事项](#禁止事项)
- [推荐做法](#推荐做法)
- [文件命名规范](#文件命名规范)
- [注释规范](#注释规范)

---

## 项目概述
Vige 是一个 FastAPI + Vue 的全栈工程模板，采用前后端分离架构：
- **vige-api**: 后端 API 服务 (FastAPI + SQLAlchemy + PostgreSQL + Redis + Huey)
- **vige-bo**: 后台管理系统 (Vue 2 + iView)
- **vige-web**: 前端用户网站 (Vue 2 + iView)
- **vige-wechat**: 微信 H5 客户端 (Vue 2 + Mint UI)

定位：通用型 Web 框架脚手架，提供认证、任务队列、媒体上传、国际化与统一接口规范，便于二次开发。

## 后端开发规范 (vige-api)

### 技术栈
- Web框架: FastAPI
- ORM: SQLAlchemy 2.0 (新式类型注解)
- 数据库: PostgreSQL + Redis
- 认证: JWT + Cookie + CSRF
- 任务队列: Huey + Redis
- 配置管理: Pydantic Settings

### Pydantic 2.x 兼容性规范 ⚠️ 极其重要

#### 核心变更说明：
项目使用 Pydantic 2.x 版本，某些语法已发生变更，必须严格遵循新版本规范。

#### 字段验证参数变更：
```python
# ❌ 错误：Pydantic 1.x 语法，会导致 PydanticUserError
class UserForm(BaseModel):
    mobile: str = Field(..., description="手机号码", regex="^1[3-9]\d{9}$")

# ✅ 正确：Pydantic 2.x 语法
class UserForm(BaseModel):
    mobile: str = Field(..., description="手机号码", pattern="^1[3-9]\d{9}$")
```

#### 配置类参数变更：
```python
# ❌ 错误：Pydantic 1.x 语法，会产生用户警告
class UserForm(BaseModel):
    username: str = Field(...)
    
    class Config:
        schema_extra = {
            "example": {
                "username": "example"
            }
        }

# ✅ 正确：Pydantic 2.x 语法
class UserForm(BaseModel):
    username: str = Field(...)
    
    class Config:
        json_schema_extra = {
            "example": {
                "username": "example"
            }
        }
```

#### 常见兼容性问题对照表：
```python
# Pydantic 1.x → Pydantic 2.x
regex="pattern"          → pattern="pattern"
schema_extra             → json_schema_extra
allow_reuse=True         → 已移除，无需设置
min_length               → min_length (保持不变)
max_length               → max_length (保持不变)
gt, ge, lt, le          → gt, ge, lt, le (保持不变)
```

#### 验证装饰器使用：
```python
# ✅ 正确：继续使用 field_validator，但注意参数变化
from pydantic import field_validator

class UserForm(BaseModel):
    mobile: str = Field(..., pattern="^1[3-9]\d{9}$")
    
    @field_validator('mobile')
    @classmethod
    def validate_mobile(cls, v):
        if not v:
            raise ValueError('手机号为必填项')
        return v
```

#### 重要注意事项：
1. **严禁使用 `regex`**：必须使用 `pattern` 替代
2. **配置类更新**：使用 `json_schema_extra` 而不是 `schema_extra`
3. **立即修复**：遇到 PydanticUserError 或相关警告立即按规范修复
4. **团队统一**：所有开发者都必须遵循 Pydantic 2.x 语法
5. **检查清单**：每次创建表单类都要检查是否使用了正确的语法

#### 错误信息识别：
- **`PydanticUserError: 'regex' is removed. use 'pattern' instead`** → 将 `regex` 改为 `pattern`
- **`Valid config keys have changed in V2: 'schema_extra' has been renamed to 'json_schema_extra'`** → 将 `schema_extra` 改为 `json_schema_extra`

#### 标准表单模板：
```python
from pydantic import BaseModel, Field
from typing import Optional

class StandardForm(BaseModel):
    """标准表单模板"""
    mobile: str = Field(..., pattern="^1[3-9]\d{9}$")
    username: str = Field(..., min_length=3, max_length=50)
    age: Optional[int] = Field(None, ge=0, le=120)
```

### 代码风格
- 类名: PascalCase (如 User, MediaModel)
- 函数/变量名: snake_case (如 send_validation_code, user_id)
- 常量: UPPER_CASE (如 AUTHJWT_SECRET_KEY)
- 数据库字段: snake_case

### 数据库操作核心原则 ⚠️ 极其重要

#### 基础原则：
1. **只读查询**: 使用 `db: Session = Depends(sm.get_db)` 依赖注入
2. **数据变更**: 使用 `with sm.transaction_scope() as sa:` 事务上下文
3. **用户认证**: 始终使用 `user: User = Depends(get_user)` 进行 JWT 验证
4. **禁止混用**: 不要在同一函数中同时使用两种数据库上下文

#### Session对象管理规范 ⚠️ 极其重要核心原则：

**关键原则：不同session中的对象不能相互操作！**

1. **session隔离原则**：
   - `current_user: User = Depends(get_user)` 获取的对象属于依赖注入的session
   - `with sm.transaction_scope() as sa:` 创建的是新的独立session
   - **绝对禁止**在新session中直接修改其他session的对象

2. **正确的对象修改方式**：
```python
# ❌ 错误：直接修改其他session的对象
@mp_required
async def update_user(form: UpdateForm, current_user: User = Depends(get_user)):
    with sm.transaction_scope() as sa:
        # 错误！current_user属于另一个session，不能在新session中修改
        current_user.nickname = form.nickname  
        current_user.updated_at = datetime.utcnow()

# ✅ 正确：在新session中重新获取对象再修改
@mp_required
async def update_user(form: UpdateForm, current_user: User = Depends(get_user)):
    with sm.transaction_scope() as sa:
        # 正确：在新session中重新获取用户对象
        user = User.get_or_404(sa, current_user.id)
        user.nickname = form.nickname
        user.updated_at = datetime.utcnow()
```

3. **核心要点**：
   - **重新获取对象**：在事务session中必须使用 `Model.get_or_404(sa, id)` 重新获取
   - **ID传递**：可以使用 `current_user.id` 获取ID，但不能直接操作对象
   - **session归属**：每个数据库对象只能在其所属的session中被修改
   - **事务完整性**：确保所有相关操作都在同一个事务session中完成

4. **常见错误模式**：
```python
# ❌ 错误模式1：直接修改依赖注入对象
with sm.transaction_scope() as sa:
    current_user.field = new_value  # 错误！

# ❌ 错误模式2：混用不同session的对象
with sm.transaction_scope() as sa:
    other_user.field = new_value  # 错误！
    current_user.related_id = other_user.id  # 错误！

# ✅ 正确模式：统一在新session中操作
with sm.transaction_scope() as sa:
    user = User.get_or_404(sa, current_user.id)
    other = OtherModel.get_or_404(sa, other_id)
    user.field = new_value
    user.related_id = other.id
```

5. **检查清单**：
   - ✅ 在事务中重新获取需要修改的对象
   - ✅ 使用对象ID进行查询，不直接操作依赖注入的对象
   - ✅ 所有相关对象都在同一个事务session中获取和操作
   - ❌ 不在新session中直接修改current_user等依赖注入对象
   - ❌ 不混用来自不同session的对象

### API 路由设计（与当前项目一致）
- API 基础前缀为 `/v1`
- 端前缀区分: `/admin/*`（后台管理）, `/web/*`（Web 端）, `/wechat/*`（微信端）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Versus2017/vige](https://github.com/Versus2017/vige) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
