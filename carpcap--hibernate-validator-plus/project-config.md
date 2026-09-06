---
trigger: always_on
description: - 用户名：CarpCap；默认使用中文沟通。
---

# Hibernate Validator Plus — 项目知识库

## 协作约定

- 用户名：CarpCap；默认使用中文沟通。
- 所有文本和源码使用 UTF-8 编码。
- 新增或修改代码时，仅在必要位置添加简洁、明确的注释。
- 禁止批量删除文件或目录；禁止使用 `del /s`、`rd /s`、`rmdir /s`、`Remove-Item -Recurse`、`rm -rf`。
- 如需删除文件，只能一次删除一个已确认的明确路径；如需批量删除，停止操作并请用户手动处理。
- 生成i18n文件时要考虑编码问题，参考每个文件现有的编码

## 项目概述

Hibernate Validator Plus（简称 HVP）是基于 **Hibernate Validator** 的增强校验框架，提供更丰富、实用的校验注解、分组校验机制以及统一的校验工具类。项目由作者 **CarpCap** 开发，采用 Apache License 2.0 开源协议。项目同时维护 1.x 和 2.x 两个大版本。

| 项目 | 说明                                                                                        |
|------|-------------------------------------------------------------------------------------------|
| Maven 坐标 | `com.carpcap:hibernate-validator-plus`                                                    |
| 1.x 版本线 | `1.4.0`；JDK 8 + `javax.validation` + Hibernate Validator 6.2.x；`1.x` 分支                   |
| 2.x 版本线 | `2.4.0`；JDK 11+ + `jakarta.validation` + Hibernate Validator 8.x；`2.x` 与 `main` 同步        |
| 公共依赖 | hutool-core 5.8.41；具体依赖以对应分支 `pom.xml` 为准                                                 |
| 验证器注册 | Google Auto Service（`@AutoService`）自动注册 `ConstraintValidator` 实现                          |
| 框架集成 | 不强制依赖 Spring；支持独立使用及 Spring MVC / Spring Boot 集成                                          |
| 示例项目 | [hibernate-validator-plus-demo](https://github.com/carpcap/hibernate-validator-plus-demo) |

### 知识库导航

- [项目结构](#项目结构)
- [校验注解](#校验注解一览)
- [验证器实现](#验证器实现)
- [分组机制](#分组机制)
- [工具类](#工具类)
- [版本历史](#版本历史)
- [构建与发布](#构建与发布)
- [已知问题](#已知问题2026-08-07)

### 分支与文档规则

- `1.x` 和 `2.x` 都是持续维护的开发线，均可增加新功能、修复缺陷并发布新版本。
- `main` 与 `2.x` 保持同步；`main` 是默认展示和 2.x 稳定开发入口，`2.x` 是明确的版本线分支。
- 1.x 新功能必须保持 JDK 8 与 `javax.validation` 兼容；2.x 新功能必须保持 JDK 11+ 与 `jakarta.validation` 兼容。
- 根目录 README 同时说明两个大版本；`docs/versions*.md` 和 `docs/usage*.md` 只记录当前 2.x 的版本与使用方式。
- 1.x 的使用说明以 `1.x` 分支中的文档和 `pom.xml` 为准，不能将 2.x 的 `jakarta.validation` 示例复制到 1.x 项目。

---

## 项目结构

```
hibernate-validator-plus/
├── pom.xml                          # Maven 构建配置
├── readme.md / readme_en.md         # 中英文文档
├── LICENSE                          # Apache 2.0 许可证
├── .gitignore
├── docs/
│   ├── versions.md / versions_en.md # 中英文版本日志
│   ├── anns.png                     # 注解使用示例图
│   ├── img0.png / img1.png / img2.png
├── assets/
├── src/
│   ├── main/
│   │   ├── java/com/carpcap/hvp/
│   │   │   ├── annotation/               # 校验注解定义（20 个）
│   │   │   ├── constraintvalidators/     # 验证器源码（25 个：22 个具体实现 + 3 个抽象基类）
│   │   │   ├── groups/                   # 校验分组接口 (16个)
│   │   │   └── utils/                    # 工具类 (2个)
│   │   └── resources/
│   │       └── ValidationMessages*.properties  # 17 个 ResourceBundle 文件
│   └── test/
│       ├── java/
│       │   ├── AnnotationTest*.java      # 3 个 main 方法测试/演示类
│       │   └── User*.java                # 3 个测试实体
│       └── resource/                     # 注意：当前使用单数 resource，非 Maven 默认 resources
│           └── 3.png                     # 文件校验测试资源
```

---

## 校验注解一览

所有注解位于 `com.carpcap.hvp.annotation` 包，均支持：
- **allowNull**: 是否允许 null 值（默认 true）
- **groups**: 分组校验
- **payload**: 负载
- **@Repeatable**: 可重复标注（含内嵌 List 注解）

| 注解 | 功能 | 关键属性 |
|------|------|----------|
| @CAccount | 账号格式验证 | regexp（正则）, min/max（长度范围，默认 5-16） |
| @CPassword | 密码强度验证 | min/max（长度 6-18），默认需包含字母+数字 |
| @CIdCard | 身份号码格式验证 | region 支持 CN/US/JP/KR/UK，CN 仅支持 18 位格式 |
| @CPhone | 手机号验证 | region 参数支持 CN/US/JP/KR 等多国号码 |
| @CIpv4 | IPv4 地址验证 | 标准 IPv4 正则 |
| @CIpv6 | IPv6 地址验证 | 通过 InetAddress 原生解析 |
| @CDomain | 域名格式验证 | 支持中文域名，level 控制层级，allowTld 控制是否允许顶级域名 |
| @CEmail | 邮箱格式验证 | 支持域名黑白名单、最大子域层级及 allowTld |
| @CPlateNumber | 中国车牌号验证 | 支持新能源/普通车牌 |
| @CFile | 文件验证 | fileNameSuffix（后缀限制）, fileSize（大小，默认 1MB） |
| @CUrl | URL 验证 | protocols（协议白名单）, allowLocalhost, allowIp |
| @CBankCard | 银行卡号验证 | Luhn 算法校验, allowedPrefixes/forbiddenPrefixes |
| @CMoney | 金额格式验证 | min/max, 整数/小数位数, 货币符号, 千分位 |
| @CDateRange | 日期范围验证 | min/max 日期, format, 支持 String/Date/LocalDate/LocalDateTime/Instant/ZonedDateTime |
| @CStrAllow | 字符串白名单验证 | value 定义允许的字符串集合 |
| @CStrDeny | 字符串黑名单验证 | value 定义禁止的字符串集合 |
| @CJson | JSON 格式验证 | 校验字符串 JSON 语法，可限制对象或数组根节点 |
| @CMacAddress | MAC 地址验证 | allowLowercase, allowEui64, allowOmittingLeadingZero |
| @CPassport | 护照号验证 | region/regexp，内置 CN/US/JP/UK/KR |
| @CPostCode | 邮政编码格式验证 | region/regexp，内置 CN/US/JP/UK/KR |

### 注解设计模式

所有注解均采用 `validatedBy = { }`（空数组），实际验证器通过 `@AutoService(ConstraintValidator.class)` 自动注册。这样注解与验证器解耦，无需在注解中硬编码验证器类。

---

## 验证器实现

验证器位于 `com.carpcap.hvp.constraintvalidators` 包，通过 Google Auto Service SPI 机制自动注册。

| 验证器 | 对应注解 | 实现策略 |
|--------|----------|----------|
| AbstractCPatternValidator | 抽象基类 | 通用的正则匹配 + null 处理基类 |
| CAccountValidator | @CAccount | 继承 AbstractCPatternValidator，叠加长度范围校验 |
| CPasswordValidator | @CPassword | 继承 AbstractCPatternValidator，叠加长度范围校验 |
| CIpAddressValidator | @CIpv4 | 直接继承 AbstractCPatternValidator |
| CDomainValidator | @CDomain | 直接继承 AbstractCPatternValidator |
| CPhoneValidator | @CPhone | 扩展正则校验，支持多地区手机号模板 |
| CIdCardValidator | @CIdCard | 按 region 选择身份号码正则，regexp 优先 |
| CPlateNumberValidator | @CPlateNumber | 扩展正则校验 |
| CUrlValidator | @CUrl | 使用 java.net.URL 解析 + 正则回退 |
| CBankCardValidator | @CBankCard | Luhn 算法校验，前缀黑白名单 |
| CMoneyValidator | @CMoney | 复杂金额格式校验（符号/千分位/小数位） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CarpCap/hibernate-validator-plus](https://github.com/CarpCap/hibernate-validator-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
