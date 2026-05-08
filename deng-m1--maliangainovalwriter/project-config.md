---
trigger: always_on
description: name: "ai-novel-assistant"
---


# AI写作软件项目全局配置
project_config:
  name: "ai-novel-assistant"
  description: "AI驱动的小说创作管理系统"

# 后端配置
backend:
  framework: "spring-boot"
  version: "3.2.0"
  java_version: "23"
  base_package: "com.ainovel"


# 前端配置
frontend:
  framework: "flutter"
  web_first: true


# 数据结构
  rule：数据结构，noval和act是一对多关系  act和chapter是一对多关系，chapter和sense是一对多关系，sense和摘要是一对一关系

# AINovalWriter 项目架构说明

## 项目概述

AINovalWriter 是一个AI辅助小说创作平台，包含前端应用（Flutter）和后端服务（Spring Boot）两个主要部分。

## 目录结构

```
AINovalWriter/
├── DESIGN_DOCUMENT.md                       # 项目顶层设计文档
├── OPTIMIZATION_SUMMARY.md                  # 优化总结文档
├── temp.java                                # 临时 Java 文件 (用途待定)
├── 提示词模板生成需求.md                      # AI 提示词模板生成功能的需求文档
├── 版本控制功能说明.md                        # 编辑器版本控制功能的设计或说明文档
├── 项目架构说明.md                          # 项目整体架构的说明文档
│
├── 需求文档/                                # 存放项目需求相关文档
│   ├── prd.md                               # 产品需求文档 (Product Requirement Document)
│   ├── requirements_validation.md           # 需求验证文档
│   ├── user_flow_and_wireframes.md          # 用户流程和线框图文档
│   ├── 前端后台任务系统需求文档.md              # 前端后台任务系统的需求文档
│   ├── 前端概要设计文档.md                    # 前端整体概要设计文档
│   ├── 前端详细设计文档.(小说列表模块,编辑器模块).md # 前端小说列表和编辑器模块的详细设计
│   ├── 前端详细设计文档（AI聊天模块）.md         # 前端 AI 聊天模块的详细设计
│   ├── 前端详细设计文档（Codex知识库模块）.md    # 前端 Codex 知识库模块的详细设计
│   ├── 前端详细设计文档（小说计划预览模块）.md     # 前端小说计划（Plan）预览模块的详细设计
│   ├── 前端详细设计文档（文件导出模块详细设计）.md # 前端文件导出模块的详细设计
│   ├── 后台任务系统需求文档.md                # 后端后台任务系统的需求文档
│   │
│   └── AI生成场景摘要和场景内容/              # AI 生成功能的特定需求文档
│       ├── 前端 AI 生成与提示词管理需求文档.md   # 前端 AI 生成与提示词管理的需求
│       ├── 后端 AI 生成与提示词管理 - 详细设计文档.md # 后端 AI 生成与提示词管理的详细设计
│       └── 后端 AI 生成与提示词管理需求文档.md   # 后端 AI 生成与提示词管理的需求
│
├── 测试计划与文档/                            # 存放测试相关计划和文档
│   ├── 后端接口文档.md                        # 后端 API 接口文档
│   ├── 改进报告.md                            # 项目改进报告
│   ├── 第一次测试计划与功能点.md                # 项目第一次迭代的测试计划和功能点
│   └── 第二次迭代第一次代码评审.md              # 第二次迭代的代码评审记录
│
├── 后端开发文档/                            # 存放后端开发过程中的文档
│   ├── AI小说助手系统后端概要设计文档.md          # 后端概要设计文档
│   ├── AI小说助手系统敏捷开发计划 - 技术验证阶段（调整版）.md # 技术验证阶段的敏捷开发计划
│   ├── AI小说助手系统敏捷开发计划 - 边验证边开发模式.md # 边验证边开发的敏捷计划
│   ├── spint1-2.md                          # Sprint 1-2 的相关文档或笔记
│   ├── 技术评估.md                            # 项目涉及的技术评估文档
│   └── 迭代日记.md                            # 开发迭代过程中的日记或记录
│
├── 原型图/                                  # 存放界面原型图及相关说明
│   ├── 实现动态获取模型列表和配置验证功能/      # 动态获取模型列表功能的文档
│   │   └── 实现动态获取模型列表和配置验证功能.md # 该功能的 Markdown 说明文档
│   │
│   ├── 剧情推演html原型图和需求文档/          # 剧情推演（Next Outline）功能的原型和需求
│   │   ├── next_outline_prototype.html      # 剧情推演功能的 HTML 原型
│   │   ├── 前端需求.md                        # 剧情推演功能的前端需求
│   │   └── 后端需求.md                        # 剧情推演功能的后端需求
│   │
│   └── plan文档和原型图/                    # 小说计划（Plan）视图的原型和文档
│       ├── plan.html                        # Plan 视图的 HTML 原型
│       └── 前端 Plan 视图原型描述.md          # Plan 视图原型的 Markdown 描述
│
├── 前端产品敏捷迭代计划/                      # 前端产品的迭代计划
│   └── 前端产品迭代计划.md                    # 前端产品迭代计划文档
│
├── target/                                  # 构建输出目录 (通常由构建工具生成)
│   └── performance-reports/                 # 性能测试报告目录
│       ├── performance_test_platform_50_users_20250313_101351.md # 平台线程模型的 50 用户性能测试报告
│       └── performance_test_virtual_50_users_20250313_101432.md # 虚拟线程模型的 50 用户性能测试报告
│
├── postman/                                 # Postman API 测试集合
│   ├── AINovalWriter_Test_API.json          # AINovalWriter 项目的 Postman 测试 API 集合
│   ├── AINoval_API_Collection.json          # AINoval 项目的 Postman API 集合
│   └── README.md                            # Postman 集合的说明文档
│
├── AINovalServer/                           # 后端 Spring Boot 项目目录
│   ├── AINoval_API_Collection.json          # (重复) AINoval API Postman 集合
│   ├── AINoval_API_Tests.postman_collection.json # AINoval API 测试 Postman 集合
│   ├── AINoval_Performance_Tests.postman_collection.json # AINoval 性能测试 Postman 集合
│   ├── hs_err_pid53088.log                  # Java 虚拟机错误日志
│   ├── hs_err_pid73408.log                  # Java 虚拟机错误日志
│   ├── PERFORMANCE_TESTING.md               # 性能测试相关说明文档
│   ├── performance_test_script.js           # (可能用于 k6 等工具) 性能测试脚本
│   ├── pom.xml                              # Maven 项目对象模型文件，定义项目依赖和构建配置
│   ├── README.md                            # AINovalServer 项目的说明文档
│   ├── Sprint1开发总结.md                   # Sprint 1 开发总结
│   ├── start-performance-test.sh            # 启动性能测试的 Shell 脚本
│   │
│   ├── 设计文档/                            # 后端特定模块的设计文档
│   │   ├── 后台任务系统后端任务分解.md          # 后台任务系统后端的任务分解文档
│   │   ├── 小说导入功能 - 高性能实施方案.md     # 小说导入功能的高性能方案设计
│   │   └── 小说导入功能设计方案 (更新版).md     # 小说导入功能的设计方案 (更新版)
│   │
│   ├── target/                              # Maven 构建输出目录
│   │   ├── test-classes/                    # 编译后的测试类目录
│   │   │   └── performance-test-report-template.md # 性能测试报告模板 (测试资源)
│   │   ├── classes/                         # 编译后的主类目录
│   │   │   ├── application-dev.yml          # 开发环境配置文件
│   │   │   ├── application-performance-test.yml # 性能测试环境配置文件
│   │   │   ├── application-test.yml         # 测试环境配置文件
│   │   │   ├── application.yml              # 主应用程序配置文件
│   │   │   ├── static/                      # 静态资源目录

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Deng-m1/MaliangAINovalWriter](https://github.com/Deng-m1/MaliangAINovalWriter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
