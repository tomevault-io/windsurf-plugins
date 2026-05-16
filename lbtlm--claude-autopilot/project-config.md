---
trigger: always_on
description: 本项目已集成智能 Claude Autopilot 2.1 系统，专为 Bash 脚本项目优化的完整智能开发工作流程。
---

# CLAUDE.md - Bash 脚本项目智能协作指南

## 🚀 **智能 Claude Autopilot 2.1 已激活**

本项目已集成智能 Claude Autopilot 2.1 系统，专为 Bash 脚本项目优化的完整智能开发工作流程。

### **📊 项目信息**

- **项目名称**: claude-autopilot
- **项目类型**: Bash 脚本项目
- **技术栈**: bash-scripts + Make + Tests
- **CE 版本**: 2.0.0
- **配置时间**: 2025-07-28 16:31:30

### **🎯 智能命令**

#### **核心开发流程**

```bash
# 一键式完整功能开发 / Smart Feature Development
/智能功能开发 <功能需求描述>
# OR /smart-feature-dev <feature description>

# 智能问题诊断和修复 / Smart Bug Fix
/智能Bug修复 <问题描述>
# OR /smart-bugfix <problem description>

# 基于最佳实践的智能重构 / Smart Code Refactor
/智能代码重构 <重构目标>
# OR /smart-code-refactor <refactor target>
```

#### **辅助工具命令**

```bash
# 重新加载全局上下文和经验 / Load Global Context
/加载全局上下文
# OR /load-global-context

# 强制刷新模式（获取最新信息）
/加载全局上下文 --force-refresh
# OR /load-global-context --force-refresh

# 项目健康度分析 / Project Status Analysis
/项目状态分析
# OR /project-status-analysis

# 清理残余文件 / Cleanup Project
/清理残余文件
# OR /cleanup-project

# 提交到GitHub / Commit to GitHub
/提交github
# OR /commit-github

# 智能Docker部署 / Smart Docker Deploy
/智能Docker部署
# OR /smart-docker-deploy

# 智能项目规划 / Smart Project Planning
/智能项目规划
# OR /smart-project-planning

# 智能结构验证 / Smart Structure Validation
/智能结构验证
# OR /smart-structure-validation

# 智能工作总结 / Smart Work Summary
/智能工作总结
# OR /smart-work-summary
```

### **🐚 Bash 脚本开发特色功能**

#### **脚本质量优化**

- **Shell 脚本规范检查**: ShellCheck 集成和最佳实践应用
- **错误处理优化**: set -euo pipefail 和完善的错误处理机制
- **参数验证智能**: 输入参数验证和帮助文档自动生成
- **跨平台兼容性**: Linux/macOS/Windows WSL 兼容性检查

#### **项目结构管理**

- **模块化设计**: 脚本功能模块化和库函数复用
- **配置管理**: 环境变量和配置文件管理最佳实践
- **日志记录系统**: 统一的日志记录和错误跟踪
- **测试框架集成**: Bash 单元测试和集成测试

#### **部署和分发**

- **权限管理**: 执行权限和安全策略配置
- **包管理**: 脚本打包和版本管理
- **依赖检查**: 系统依赖和工具检查自动化
- **安装脚本**: 一键安装和卸载脚本生成

#### **标准项目结构支持 (基于 GNU 编码标准)**

```
bash-scripts项目/
├── bin/                          # 可执行脚本 (GNU标准)
│   ├── main-script               # 主执行脚本
│   ├── helper-tools              # 辅助工具脚本
│   └── setup.sh                  # 安装配置脚本
├── lib/                          # 库函数和公共模块 (GNU标准)
│   ├── core-functions.sh         # 核心功能库
│   ├── cross-platform-utils.sh   # 跨平台工具库
│   ├── error-handling.sh         # 错误处理库
│   └── logging.sh                # 日志记录库
├── etc/                          # 配置文件 (GNU标准)
│   ├── config.conf               # 主配置文件
│   ├── defaults.conf             # 默认配置
│   └── aliases.json              # 命令别名配置
├── share/project-name/           # 共享资源 (GNU标准)
│   ├── templates/                # 模板文件
│   ├── docs/                     # 文档资源
│   └── examples/                 # 示例脚本
├── tests/                        # 测试框架
│   ├── test-framework.sh         # 测试框架核心
│   ├── test-basic-functionality.sh
│   ├── test-main-script.sh
│   └── run-tests.sh              # 测试运行器
├── .shellcheckrc                 # ShellCheck配置
├── Makefile                      # 构建和测试命令
├── README.md                     # 项目说明 (中文简体)
├── README-TW.md                  # 项目说明 (中文繁体)
├── README-EN.md                  # 项目说明 (英文)
├── CHANGELOG.md                  # 更新日志
├── VERSION                       # 版本信息
└── CLAUDE.md                     # Claude智能开发配置
```

#### **智能开发和测试**

```bash
# 开发环境设置
make dev-setup                    # 设置开发环境和权限

# 代码质量检查
make lint                         # ShellCheck语法检查和代码规范
make format                       # 代码格式化 (如果支持)

# 测试执行
make test                         # 运行完整测试套件
./tests/run-tests.sh             # 直接运行测试
./tests/run-tests.sh --basic-only # 仅运行基础功能测试

# 项目构建
make build                        # 构建项目 (如果需要)
make package                      # 打包分发

# 安装部署
make install                      # 安装到系统
make deploy                       # 部署脚本

# 实用工具
make clean                        # 清理临时文件
make help                         # 查看所有可用命令
make health-check                 # 项目健康度检查
```

### **🧠 智能能力**

#### **MCP 工具链集成**

- **sequential-thinking**: 复杂脚本逻辑设计和流程优化
- **context7**: 动态获取 Bash 最新文档和最佳实践
- **memory**: Bash 开发经验自动复用和模式库
- **filesystem**: 脚本文件结构智能分析和组织

#### **全局规则遵守**

- **Bash 代码规范**: 自动应用 ShellCheck 规则和 Google Shell Style Guide
- **安全编程**: 避免 Shell 注入、路径遍历等安全漏洞
- **错误处理**: 完善的错误处理和退出码管理
- **文档规范**: 标准的脚本注释和使用说明

#### **Bash 专项智能特性**

- **函数库设计**: 可复用函数库的设计和管理，模块化架构
- **参数处理优化**: getopts 和长选项处理最佳实践
- **并发控制**: 后台任务和进程管理优化
- **跨平台兼容性**:
  - Linux/macOS/Windows WSL 完全兼容
  - 自动处理 realpath、sed 等命令差异
  - 路径分隔符和权限管理跨平台适配
  - 命令存在性检查和降级方案
- **测试驱动开发**:
  - 内置 Bash 测试框架
  - 断言函数和测试报告
  - 持续集成测试支持
- **智能错误处理**:
  - set -euo pipefail 最佳实践
  - 统一的错误退出码管理
  - 详细的错误日志和调试信息

### **📋 Bash 开发建议**

#### **开始开发**

1. 描述脚本需求，如："创建系统监控脚本"
2. 系统会自动设计脚本架构和函数结构
3. 生成符合最佳实践的 Bash 代码

#### **脚本优化**

1. 说明优化目标，如："提高脚本执行效率"
2. 系统会分析性能瓶颈和优化机会
3. 提供基于最佳实践的优化方案

#### **错误处理**

1. 描述错误场景，如："网络连接失败处理"
2. 系统会设计完善的错误处理机制
3. 确保脚本的健壮性和可靠性

### **🔧 开发工具集成**

#### **质量保证工具**

- **ShellCheck**: 静态代码分析和语法检查 (集成在 make lint 中)
- **内置测试框架**: Bash 原生测试框架，无外部依赖
- **语法验证**: bash -n 语法检查集成
- **跨平台验证**: 自动检测和测试多平台兼容性

#### **开发辅助工具**

- **Make 构建系统**: 统一的开发工作流管理
- **跨平台工具库**: 自动处理平台差异的工具函数
- **智能路径处理**: realpath 替代方案和路径规范化
- **权限管理**: 自动化脚本权限设置和验证
- **版本管理**: 基于 Git 的版本控制和标签管理

#### **测试和 CI/CD 工具**

- **内置测试运行器**: ./tests/run-tests.sh
- **测试报告**: 彩色输出和详细测试结果
- **健康检查**: make health-check 项目状态检查
- **持续集成**: 适配 GitHub Actions 和其他 CI 系统

#### **部署和分发工具**

- **Make install**: 标准化系统安装
- **打包系统**: make package 创建分发包

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lbtlm/claude-autopilot](https://github.com/lbtlm/claude-autopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
