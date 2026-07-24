---
trigger: always_on
description: DjangoStarter 是下一代 Django 项目快速开发模板，专为提升开发效率和性能而设计。通过这个全新的框架版本，开发者能够迅速搭建起符合现代 web 应用标准的项目基础架构。
---

# DjangoStarter项目开发规范

## 项目背景

DjangoStarter 是下一代 Django 项目快速开发模板，专为提升开发效率和性能而设计。通过这个全新的框架版本，开发者能够迅速搭建起符合现代 web 应用标准的项目基础架构。

## 技术栈

### 后端

- Python版本: 3.14
- 包管理器: uv
- Web框架: Django 6.0+ (with argon2)
- API框架: Django-Ninja 1.5+
- 缓存: 
  - 本地开发时使用内存缓存  `django.core.cache.backends.locmem.LocMemCache`
  - 部署生产环境时使用 Redis `django_redis.cache.RedisCache`


### 前端(基于Django后端渲染)

- 包管理: pnpm
- 构建工具: Gulp
- 模板引擎: Django Templates
- 交互框架: HTMX
- JavaScript框架: Alpine.js
- CSS框架: TailwindCSS v4
- 组件库: DaisyUI v5 (Pure CSS)
- 图标: Font Awesome Free 6
- 动画: AOS (Animate On Scroll)

### 开发工具

- 容器化: Docker + Docker Compose
- 测试框架: Pytest

## Essential Commands

### Setup

```bash
# Install Python dependencies
uv sync

# Run database migrations
uv run ./src/manage.py migrate

# Install frontend dependencies
pnpm install

# Copy frontend assets
pnpm gulp:move
```

### Development

```bash
# Django dev server (WSGI)
uv run ./src/manage.py runserver

# Watch TailwindCSS changes
pnpm run tw:watch
```

### Code Generation

```bash
# Generate CRUD + tests + admin for an app
uv run ./src/manage.py autocode app_name "Display Name"

# Generate specific models only
uv run ./src/manage.py autocode blog "Blog" --models post category

# skip some modules
uv run ./src/manage.py autocode blog "Blog" --no-admin --no-apps --no-tests --no-apis

# Generate seed data for an app
uv run ./src/manage.py  seed app_label 10
```

### Database

```bash
# Create migrations
uv run ./src/manage.py makemigrations

# Apply migrations
uv run ./src/manage.py migrate
```

### Testing

```bash
# Django tests
uv run ./src/manage.py test
```

## Architecture

### Directory Structure

```
src/
├── apps/                  # Business applications
│   ├── account/           # Authentication system
│   └── demo/              # Demo app (reference implementation)
├── config/                # Django configuration
│   ├── settings/          # Split settings (django-split-settings)
│   │   ├── components/    # Config components (cache, auth, security, etc.)
│   │   └── environments/  # Environment-specific configs
│   ├── urls.py            # Main URL config
│   ├── apis.py            # NinjaAPI initialization - register routers here
│   └── wsgi.py/asgi.py    # Entry points
├── django_starter/        # Core framework code
│   ├── contrib/           # Built-in components (code_generator, admin, monitoring)
│   ├── db/models.py       # ModelExt base class
│   ├── http/              # Response handling
│   └── middleware/        # Security middleware
├── static/                # Static files
├── templates/             # common Django templates, don't edit!
└── locale/                # i18n
```

### Core Framework Components

**ModelExt Base Class** (`src/django_starter/db/models.py`):
All models inherit from `ModelExt`, which provides:

- Soft delete via `is_deleted` field
- Automatic timestamps (`created_time`, `updated_time`)
- Custom manager that filters out deleted objects

**Django-Ninja API Organization**:

- APIs organized per app in `apps/[app]/apis/`
- Automatic CRUD generation via `autocode` command
- Type-safe Pydantic schemas
- Auto-generated OpenAPI docs at `/api/docs`

**Split Settings** (`src/config/settings/`):

- Base settings in `components/`
- Environment-specific overrides in `environments/`
- Docker-aware configuration detection

### Application Development Pattern

When creating a new app:

1. Create app: `cd apps && uv run django-admin startapp app_name`
2. Add to `INSTALLED_APPS` in `src/config/settings/components/install_apps.py`
3. Define models in `apps/app_name/models.py` (inherit from `django_starter.db.models.ModelExt`)
4. Run `python manage.py autocode app_name "Display Name"` to generate CRUD apis, tests, admin
5. Register router in `src/config/apis.py`: `api.add_router('app_name', router)`
6. Run migrations

## 开发规范

### 代码风格

#### Python代码规范

- 遵循 PEP 8 代码风格
- 使用类型注解（Type Hints）
- 函数和类必须添加 docstring 注释 (reStructuredText风格)
- 变量和函数命名使用snake_case
- 类名使用PascalCase
- 常量使用UPPER_CASE


### Django应用开发规范

#### 模型定义规范

- 使用明确的字段名称
- 每个字段需要提供友好的 `verbose_name`
- 添加适当的 `Meta` 类配置
- 实现 `__str__` 方法
- 使用 `django-simple-history` 进行历史记录

```python
from django.db import models
from simple_history.models import HistoricalRecords

class UserProfile(models.Model):
    """
    用户档案模型
    """
    user = models.OneToOneField('auth.User', on_delete=models.CASCADE, verbose_name='用户')
    nickname = models.CharField(max_length=50, verbose_name='昵称')
    avatar = models.ImageField(upload_to='avatars/', blank=True, verbose_name='头像')
    created_at = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    updated_at = models.DateTimeField(auto_now=True, verbose_name='更新时间')
    
    # 历史记录
    history = HistoricalRecords()
    
    class Meta:
        verbose_name = '用户档案'
        verbose_name_plural = '用户档案'
        db_table = 'user_profile'
        
    def __str__(self) -> str:
        return f"{self.user.username} - {self.nickname}"
```

### API开发规范

#### Django-Ninja API

- 代码路径：
  - 简单逻辑:  apis path `app/apis.py`, schema path `app/schemas.py`
  - 复杂逻辑: 先将接口进行分组，代码放在 `app/apis/[group]`  package，分 apis.py, schemas.py 文件存储

- 使用Pydantic模型进行数据验证
- 实现适当的错误处理
- 添加 API 文档说明 (summary, description 参数)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [djangostarter/web](https://github.com/djangostarter/web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
