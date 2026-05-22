---
trigger: always_on
description: - 服务端：JAVA、Spring Boot、Spring Framework、Maven、Mybatis-Plus、以及其他JAVA相关技术
---

## 角色定义

你是一个经验丰富的开发者，擅长技术：
- 服务端：JAVA、Spring Boot、Spring Framework、Maven、Mybatis-Plus、以及其他JAVA相关技术
- 前端：TypeScript、Node.js、Element UI等其他前端组件
- 微信小程序：微信组件、微信支付、微信登录等微信相关技术

## 项目结构
根目录/
├── src/                                # Java 后端源代码
│   ├── main/
│   │   ├── java/com/shechubbb/smdc/
│   │   │   ├── config/                 # 配置类
│   │   │   ├── controller/             # 控制器
│   │   │   │   ├── admin/              # 管理后台控制器，Controller统一以Admin作为前缀
│   │   │   │   └── mini/               # 小程序控制器
│   │   │   ├── entity/                 # 实体类
│   │   │   ├── mapper/                 # Mybatis-Plus Mapper接口
│   │   │   ├── service/                # 服务接口
│   │   │   │   └── impl/               # 服务实现类
│   │   │   ├── vo/                     # 视图对象
│   │   │   ├── common/                 # 公共组件
│   │   │   │   ├── exception/          # 异常处理
│   │   │   │   ├── util/               # 工具类
│   │   │   │   └── result/             # 统一响应结果
│   │   │   └── *Application.java       # 应用启动类
│   │   ├── resources/
│   │   │   ├── application.yml         # 应用配置文件
├── miniapp/                            # 微信小程序
├── admin/                              # Vue管理后台
├── .gitignore                          # Git忽略文件
├── pom.xml                             # Maven配置
└── README.md                           # 项目说明


## 强制规定

1. **一致性**：有任何更改，必须保证服务端（src目录）、微信小程序（miniapp目录）、管理后台（admin目录）的一致性
2. **代码清理**：替换新的解决方案后，要检查之前的代码是否还有使用，没有用就删除掉
3. **错误检查**：每次代码修改后，必须检查编译器/开发工具是否报告错误并立即解决
4. **零错误提交**：禁止提交包含编译错误的代码到代码库
5. **数据库更新一致性**：数据库实体定义有变更时，必须更新相关的初始化SQL，并提供更新SQL

## 开发规范

### 技术栈规范

#### 服务端
- **核心框架**：Spring Boot
- **ORM框架**：Mybatis-Plus
- **数据库**：MySQL
- **构建工具**：Maven
- **JDK版本**：JDK8

#### 微信小程序
- **基础技术**：微信小程序原生技术
- **UI组件**：Element UI等第三方组件

#### 管理后台
- **前端框架**：Vue
- **UI组件库**：Element UI、Element Plus
- **扩展组件**：其他第三方组件

### 接口规范

- **交互协议**：服务端和前端交互协议为POST，请求参数为JSON，返回数据为JSON
- **响应格式**：统一响应格式：`{"code":1,"msg":"errorMsg","data":obj}`
- **成功状态**：`code=1` 代表成功，`data`为具体的数据
- **失败状态**：`code=0` 代表失败，`msg`字段为错误信息用于提示

### 微信小程序开发规范

#### 颜色管理
- **统一定义**：所有颜色值必须在 `miniapp/styles/theme.wxss` 中统一定义
- **禁止硬编码**：禁止在页面或组件中直接使用色值（如 #FFFFFF）
- **变量引用**：使用CSS变量方式引用颜色值

#### 资源使用
- **图标优先级**：优先使用微信小程序内置图标
- **替代方案**：如无合适内置图标，可安装其他图标库依赖
- **最后选择**：最后考虑使用图片资源

### 后端已实现接口列表

#### 用户相关接口
##### 小程序端接口
- **POST /mini/user/login** - 微信登录
- **POST /mini/user/account/login** - 账号密码登录
- **POST /mini/user/info** - 获取当前用户信息
- **GET /mini/user/info/{id}** - 根据ID获取用户信息（管理员权限）
- **POST /mini/user/update** - 更新用户信息

##### 管理端接口
- **POST /admin/employee/login** - 员工登录
- **GET /admin/employee/info/{id}** - 获取员工信息
- **GET /admin/employee/page** - 员工列表分页查询
- **POST /admin/employee/add** - 添加员工
- **POST /admin/employee/update** - 更新员工信息

#### 分类相关接口
##### 小程序端接口
- **GET /mini/category/list** - 获取分类列表

##### 管理端接口
- **GET /admin/category/page** - 分类分页查询
- **GET /admin/category/list** - 分类列表查询
- **POST /admin/category/add** - 添加分类
- **POST /admin/category/update** - 更新分类
- **POST /admin/category/delete** - 删除分类

#### 菜品相关接口
##### 小程序端接口
- **GET /mini/dish/list/{categoryId}** - 根据分类ID获取菜品列表
- **GET /mini/dish/list?categoryId=** - 根据分类ID获取菜品列表（请求参数方式）
- **GET /mini/dish/detail/{id}** - 获取菜品详情

##### 管理端接口
- **GET /admin/dish/page** - 菜品分页查询
- **GET /admin/dish/info/{id}** - 获取菜品信息
- **POST /admin/dish/add** - 添加菜品
- **POST /admin/dish/update** - 更新菜品
- **POST /admin/dish/delete** - 删除菜品

#### 桌位相关接口
##### 管理端接口
- **GET /admin/table/page** - 桌位分页查询
- **POST /admin/table/add** - 添加桌位
- **POST /admin/table/update** - 更新桌位
- **POST /admin/table/delete** - 删除桌位
- **GET /admin/table/qrcode/{id}** - 生成桌位二维码

#### 订单相关接口
##### 小程序端接口
- **POST /mini/order/create** - 创建订单
- **POST /mini/order/pay** - 支付订单
- **POST /mini/order/cancel** - 取消订单
- **GET /mini/order/list** - 订单列表
- **GET /mini/order/detail/{id}** - 订单详情
- **GET /mini/order/table/{code}** - 扫码获取桌位信息

##### 管理端接口
- **GET /admin/order/page** - 订单分页查询
- **GET /admin/order/detail/{id}** - 获取订单详情
- **POST /admin/order/accept** - 接单
- **POST /admin/order/complete** - 完成订单
- **POST /admin/order/cancel** - 取消订单

#### 店铺信息相关接口
##### 小程序端接口
- **POST /mini/shop/info** - 获取店铺信息

##### 管理端接口
- **GET /admin/shop/info** - 获取店铺信息
- **POST /admin/shop/update** - 更新店铺信息


### 小程序端已实现页面

#### 基础页面
- **登录页面** (`pages/login/login`) - 用户登录页面，支持微信登录和账号密码登录
- **首页** (`pages/index/index`) - 应用首页，展示店铺信息和热门菜品
- **分类页** (`pages/category/category`) - 展示菜品分类列表
- **菜品页** (`pages/dish/dish`) - 展示具体分类下的菜品列表
- **购物车** (`pages/cart/cart`) - 用户已选菜品的购物车

#### 订单相关页面
- **订单列表** (`pages/order/list/list`) - 用户的历史订单列表
- **订单确认** (`pages/order/confirm/confirm`) - 确认订单信息和支付
- **订单详情** (`pages/order/detail/detail`) - 查看订单的详细信息

#### 用户相关页面
- **用户中心** (`pages/user/user`) - 用户个人中心，展示用户信息和功能入口

#### 导航结构
- **底部Tab导航**：主页和用户中心
- **顶部导航**：页面标题"扫码点餐"，采用品牌主色调 #FF5722


### 数据库表定义
-- 用户表
CREATE TABLE IF NOT EXISTS `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `open_id` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '微信openid',
  `session_key` varchar(255) DEFAULT NULL COMMENT '会话密钥',
  `nick_name` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '昵称',
  `username` varchar(50) DEFAULT NULL COMMENT '用户名',
  `password` varchar(255) DEFAULT NULL COMMENT '密码',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jarcms/wechat-smdc](https://github.com/jarcms/wechat-smdc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
