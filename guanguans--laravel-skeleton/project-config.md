---
trigger: always_on
description: [//]: # (https://github.com/github/awesome-copilot)
---

# Laravel PHP 8.5+ Package Copilot Instructions

[//]: # (https://github.com/github/awesome-copilot)

## 项目背景

- 技术栈：PHP (>=8.5), Laravel 13, Composer, NPM, JavaScript, Go, SQL
- 代码风格：PSR-12，使用 Laravel Pint / PHP-CS-Fixer (仓库含自定义 Fixer)
- 测试：Pest 优先；必要时 PHPUnit
- 目标：可维护、类型明确、易测试、高性能

## PHP 代码规范

- 严格遵守 PSR-12；必须使用 `declare(strict_types=1);`
- 使用 PHP 8.5+ 新特性：readonly 类、枚举、联合类型、属性、泛型注解
- 方法参数和返回值必须有类型声明，包括 nullable 类型使用 `?Type`
- 优先使用命名参数和简短闭包语法 `fn() =>`
- 使用新的 `match` 表达式替代简单的 `switch`
- 类常量使用 `final` 修饰符防止重写
- 优先使用 `readonly` 类和属性提升不可变性

## JavaScript/前端规范

- 使用 ES6+ 语法，优先 `const`/`let`，避免 `var`
- 函数优先使用箭头函数 `() => {}`
- 使用解构赋值和扩展运算符
- 异步操作使用 `async/await`，避免 Promise 链
- 模块化导入/导出，使用 `import/export`
- 遵循 ESLint 配置规则

## Go 代码规范

- 严格遵循 `go fmt` 和 `gofmt` 格式化标准
- 错误处理：必须检查每个 `error` 返回值，不忽略
- 接口设计：保持简小，遵循单一职责原则
- 包命名：简短、小写、无下划线
- 函数命名：驼峰式，公共函数首字母大写
- 使用 `context.Context` 处理超时和取消操作
- 并发安全：使用 `sync` 包的原语，避免数据竞争

## SQL 规范

- 关键字使用大写：`SELECT`, `FROM`, `WHERE`, `ORDER BY`
- 表名和字段名使用 snake_case
- 复杂查询使用适当的缩进和换行
- 避免 `SELECT *`，明确指定需要的字段
- 使用参数化查询防止 SQL 注入
- 索引命名：`idx_表名_字段名`，唯一索引：`uk_表名_字段名`
- 外键命名：`fk_当前表_引用表_字段名`

## Laravel 架构约定

### 控制器 (Controllers)

- 保持精简，单一方法不超过 20 行
- 业务逻辑委托给 Service/Action 类
- 使用资源控制器方法命名：`index`, `show`, `store`, `update`, `destroy`
- 返回统一的 JSON 响应格式
- 使用类型提示注入依赖
- 控制器方法使用 `__invoke()` 实现单一职责行为类

### 服务层 (Services)

- 处理复杂业务逻辑
- 方法命名动词开头：`create`, `update`, `delete`, `process`
- 单一职责，每个服务类专注一个领域
- 返回 DTO 或领域对象，不直接返回 Eloquent 模型
- 使用数据库事务处理复杂操作

### Actions (单一职责行为类)

- 每个 Action 只处理一个具体业务操作
- 使用 `__invoke()` 方法作为入口
- 命名使用动词 + 名词格式：`CreateUser`, `SendNotification`
- 返回明确的值对象或 DTO

### 仓储模式 (Repositories)

- 处理数据访问逻辑
- 定义接口，使用依赖注入
- 方法命名：`findById`, `findByEmail`, `getActive`
- 返回 Collection 或 Model 实例
- 使用查询构建器优化性能

### 数据传输对象 (DTOs)

- 使用 readonly 类定义 DTO
- 构造函数参数使用属性提升
- 提供 `fromArray` 和 `toArray` 方法
- 验证数据完整性

### 值对象 (Value Objects)

- 使用 readonly 类定义不可变值对象
- 实现 `equals()` 方法进行值比较
- 提供工厂方法创建实例
- 封装业务规则和验证逻辑

### 请求验证 (Form Requests)

- 每个创建/更新操作单独的 Request 类
- 规则方法返回数组，使用 Laravel 验证规则
- 自定义错误消息使用 `messages()` 方法
- 授权逻辑放在 `authorize()` 方法
- 使用 `prepareForValidation()` 预处理数据
- 自定义验证规则放在 `app/Rules` 目录

### API 资源 (Resources)

- 使用 API Resource 转换模型数据
- 避免在 Resource 中执行查询
- 条件字段使用 `when()` 方法
- 嵌套资源使用 `Resource::collection()`
- 使用 `whenLoaded()` 处理关联关系
- 分页资源使用 `ResourceCollection`

### Eloquent 模型 (Models)

- 使用 `$fillable` 属性明确可填充字段
- 定义关联关系并指定返回类型
- 使用访问器和修改器处理数据转换
- 实现模型事件和观察者
- 使用枚举定义状态字段
- 使用 `$casts` 属性自动类型转换
- 定义查询作用域方法 `scope*`
- 软删除使用 `SoftDeletes` trait

### 模型观察者 (Observers)

- 创建独立的观察者类处理模型事件
- 观察者方法命名对应模型事件：`created`, `updated`, `deleted`
- 在服务提供者中注册观察者
- 避免在观察者中执行复杂业务逻辑

### 策略类 (Policies)

- 为每个模型创建对应的策略类
- 策略方法命名：`view`, `create`, `update`, `delete`
- 使用 `Gate::allows()` 或 `@can` 指令检查权限
- 在控制器中使用 `authorize()` 方法

### 枚举 (Enums)

- 使用 PHP 8.1+ 枚举定义常量值
- 实现方法提供额外功能
- 使用 backed enums 存储到数据库
- 枚举方法提供标签、颜色等业务属性

### 集合 (Collections)

- 使用 Laravel Collection 处理数组数据
- 优先使用 Collection 方法而非原生 PHP 数组函数
- 自定义 Collection 类继承 `Illuminate\Support\Collection`
- 使用管道 `pipe()` 处理复杂数据转换

## 依赖管理

### 依赖注入

- 优先使用构造函数注入
- 接口注入，不直接依赖具体实现
- 使用 Laravel 服务容器管理依赖
- 避免服务定位器模式

### 服务提供者 (Service Providers)

- 按功能模块组织服务提供者
- `register()` 方法注册服务
- `boot()` 方法执行初始化逻辑
- 使用延迟加载提升性能
- 条件绑定使用 `when()` 方法

### 门面 (Facades)

- 谨慎使用门面，优先依赖注入
- 自定义门面继承 `Illuminate\Support\Facades\Facade`
- 提供 `getFacadeAccessor()` 方法
- 在测试中使用 `shouldReceive()` 模拟门面

## 测试约定

### Pest 测试规范

- 测试方法命名：`it('should return user when valid id provided')`
- 使用描述性的测试组织：`describe('UserService')`
- 测试数据使用 Factory 生成
- 断言使用 Pest 提供的 `expect()` 语法

### 测试覆盖

- 单元测试：Service、Repository、DTO 类
- 功能测试：API 端点、完整业务流程
- 集成测试：数据库交互、外部服务调用
- 浏览器测试：使用 Laravel Dusk 进行 E2E 测试
- 测试覆盖率目标：80% 以上

### 模型工厂 (Factories)

- 使用 Factory 生成测试数据
- 定义状态方法便于创建特定状态的数据
- 避免在测试中硬编码数据
- 使用 `sequence()` 方法创建序列数据

### HTTP 测试

- 使用 `actingAs()` 模拟用户认证
- 测试 JSON 响应使用 `assertJson()` 系列方法
- 文件上传测试使用 `UploadedFile::fake()`
- 数据库断言使用 `assertDatabaseHas()`

## 数据库设计

### 迁移文件 (Migrations)

- 文件命名清晰描述变更内容
- 使用 Laravel 迁移方法，避免原生 SQL
- 外键约束使用 `constrained()` 方法
- 添加适当的索引
- 使用 `after()` 方法指定字段位置
- 批量修改使用 `table()` 方法

### 数据库填充 (Seeders)

- 使用 Factory 生成测试数据
- 环境特定的种子数据
- 避免在生产环境运行不必要的 Seeder
- 使用 `DatabaseSeeder` 统一管理

### 查询优化

- 避免 N+1 问题，使用 `with()` 预加载
- 大数据集使用 `chunk()` 或 `lazy()` 处理
- 复杂查询使用 Query Builder 或原生 SQL
- 使用数据库索引优化查询性能
- 子查询使用 `whereExists()` 或 `whereHas()`

### 数据库事务

- 复杂操作使用 `DB::transaction()` 包装
- 嵌套事务使用 `DB::beginTransaction()` 手动控制
- 事务回调中抛出异常自动回滚
- 测试中使用 `RefreshDatabase` trait

## 队列处理 (Queues)

### 队列任务

- 任务类使用 `ShouldQueue` 接口
- 实现 `failed()` 方法处理失败情况
- 使用唯一 ID 防止重复执行
- 设置重试次数和延迟时间
- 批量任务使用 `Bus::batch()`

### 队列监控

- 监控队列长度和处理时间
- 失败任务及时处理
- 使用 Horizon 管理队列状态
- 队列工作进程监控和自动重启

### 任务调度 (Task Scheduling)

- 使用 `Schedule` 定义定时任务
- 任务频率使用 Laravel 提供的方法
- 任务输出重定向到日志
- 使用 `onOneServer()` 避免重复执行

## 事件系统 (Events)

- 事件类使用 `Dispatchable` trait
- 监听器处理具体业务逻辑
- 使用队列处理异步监听器
- 避免在事件中执行重逻辑
- 事件发现使用 `shouldDiscoverEvents()` 方法

## 通知系统 (Notifications)

### 通知类

- 继承 `Illuminate\Notifications\Notification`
- 实现 `via()` 方法定义通知渠道
- 每个渠道实现对应的方法：`toMail()`, `toDatabase()`
- 使用 `Notifiable` trait 添加通知功能

### 通知渠道

- 邮件通知使用 Mailable 类
- 数据库通知存储到 `notifications` 表
- 广播通知使用 WebSocket
- 短信通知集成第三方服务

## 邮件系统 (Mail)

### Mailable 类

- 继承 `Illuminate\Mail\Mailable`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guanguans/laravel-skeleton](https://github.com/guanguans/laravel-skeleton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
