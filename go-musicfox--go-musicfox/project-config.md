---
trigger: always_on
description: > 最后更新：2026-06-15 | 基于代码库当前状态
---


# go-musicfox 代码规范文档

> 最后更新：2026-06-15 | 基于代码库当前状态

## 目录

1. [Go 版本与模块](#1-go-版本与模块)
2. [项目结构规范](#2-项目结构规范)
3. [命名规范](#3-命名规范)
4. [代码格式化](#4-代码格式化)
5. [Import 组织](#5-import-组织)
6. [注释与文档规范](#6-注释与文档规范)
7. [错误处理模式](#7-错误处理模式)
8. [接口设计规范](#8-接口设计规范)
9. [并发编程规范](#9-并发编程规范)
10. [配置管理规范](#10-配置管理规范)
11. [Git 提交规范](#11-git-提交规范)
12. [项目特定约定](#12-项目特定约定)

---

## 1. Go 版本与模块

- **Go 版本**：`go 1.24.0`（go.mod 声明）
- **模块路径**：`github.com/go-musicfox/go-musicfox`
- **依赖管理**：Go Modules + vendoring（`vendor/` 目录随代码提交）
- **replace 指令**：多处 fork 依赖的 replace（如 bubbletea、beep、gohook 等）

示例（`go.mod:99-113`）：
```go
replace (
    github.com/charmbracelet/bubbletea v0.25.0 => github.com/go-musicfox/bubbletea v0.25.0-foxful
    github.com/cocoonlife/goflac v0.0.0-20170210142907-50ea06ed5a9d => github.com/go-musicfox/goflac v0.1.5
    // ...
)
```

---

## 2. 项目结构规范

### 顶层目录

| 目录 | 用途 | 说明 |
|------|------|------|
| `cmd/` | 应用入口 | 仅 `musicfox.go`，`main()` 调用 `runtime.Run()` |
| `internal/` | 核心业务 | 22 个子包，外部不可 import |
| `utils/` | 工具库 | 可对外暴露，按功能分子包 |
| `configs/` | 嵌入配置 | 默认 TOML 配置文件 |
| `vendor/` | 依赖 vendoring | 随代码提交 |

### internal 子包组织

```
internal/
├── automator/     # 自动播放
├── commands/      # CLI 命令定义
├── composer/      # 分享文本模板
├── configs/       # 配置结构体与加载
├── keybindings/   # 快捷键定义
├── lastfm/        # Last.fm API
├── lyric/         # 歌词服务
├── macdriver/     # macOS 原生 API 封装
├── netease/       # 网易云 API 错误类型
├── player/        # 播放引擎（多平台多引擎）
├── playlist/      # 播放列表管理（策略模式）
├── remote_control/# 远程控制（MPRIS/NowPlaying）
├── reporter/      # 播放上报（Last.fm/网易）
├── runtime/       # 运行时初始化
├── storage/       # BoltDB 存储层
├── structs/       # 数据模型
├── track/         # 音频轨道管理
├── types/         # 全局常量与类型
└── ui/            # TUI 界面（60+ 文件，最大包）
```

### 文件组织约定

- **一个 struct/接口 = 一个文件**：如 `internal/playlist/ordered.go` 只含 `OrderedPlayMode`
- **平台特定文件**：使用 `_darwin.go`、`_linux.go`、`_windows.go` 后缀
  - 示例：`osx_player_darwin.go`、`win_media_player_windows.go`
- **测试文件**：与源文件同目录，`_test.go` 后缀
- **菜单文件命名**：`menu_<功能>.go`（如 `menu_main.go`、`menu_album_list.go`）

---

## 3. 命名规范

### 包命名

- **全小写**，不使用下划线或驼峰
- 简短、描述性强：`configs`、`structs`、`player`、`storage`
- 特殊：`utils/_struct` 因与关键字冲突使用 underscore 前缀
- 注意 `package _struct` 在代码中使用 `_struct.XXX` 引用

### 文件命名

- 全小写 + 下划线分隔：`beep_player.go`、`event_handler.go`
- 测试文件：`原文件名_test.go`
- 平台特定：`文件名_GOOS.go` 或 `文件名_GOOS_GOARCH.go`

### 类型命名

- **接口**：单方法接口常用 `-er` 后缀；多方法接口直接名词如 `Player`、`Menu`
  ```go
  type Player interface { ... }    // internal/player/player.go:13
  type PlaylistManager interface { ... } // internal/playlist/interfaces.go:10
  type Model interface { ... }      // internal/storage/model.go:3
  ```
- **结构体**：PascalCase，如 `Netease`、`ListLoopPlayMode`、`LocalDBManager`
- **私有结构体**：小写开头，如 `baseMenu`、`playlistManager`

### 函数命名

- **导出函数**：PascalCase，如 `NewPlaylistManager()`、`Play()`
- **未导出函数**：camelCase，如 `saveStateAsync()`、`registerPlayModes()`
- **构造函数**：统一使用 `NewXxx()` 模式
  ```go
  func NewPlaylistManager() PlaylistManager       // internal/playlist/manager.go:26
  func NewListLoopPlayMode() PlayMode             // internal/playlist/list_loop.go:13
  func NewConfigFromTomlFile(tomlPath string) (*Config, error) // internal/configs/loader.go:25
  ```

### 变量命名

- 包级变量：`AppConfig`、`DBManager`（全局变量）
- 局部变量：短名称（Go 惯例），如 `err`、`cfg`、`pm`
- 常量：PascalCase，如 `AppName`、`BeepPlayer`、`MaxPlayErrCount`
- 错误变量：`Err` 前缀，如 `ErrEmptyPlaylist`、`ErrInvalidIndex`

### 常量

定义在 `internal/types/constants.go`（全局常量）和各包内：
```go
const AppName = "musicfox"     // types/constants.go:15
const BeepPlayer = "beep"      // types/constants.go:42
const MaxPlayErrCount = 3       // types/constants.go:52
```

---

## 4. 代码格式化

### 基础工具

- **gofmt**：标准 Go 格式化
- **goimports**：import 自动管理
- **gci**：import 分组排序

### golangci-lint 配置

`.golangci.yml` 配置了：

**Linters（代码检查）**：
- `govet` - Go 官方检查
- `errcheck` - 错误检查
- `ineffassign` - 无效赋值检测
- `staticcheck` - 静态分析（all checks，排除 SA4006、SA1029）
- `unused` - 未使用代码检测

**Formatters（格式化）**：
- `gci` - import 分组（标准库 → 第三方 → go-musicfox）
- `gofmt` - 代码格式化
- `goimports` - import 管理

**Pre-commit hook**（`githooks/pre-commit`）：
```sh
make lint-fix
git add .
```

### Import 分组规则

按以下顺序分组，组间空行：
1. **标准库**
2. **第三方库**
3. **go-musicfox 内部包**（前缀 `github.com/go-musicfox/go-musicfox`）

示例（`internal/playlist/manager.go:3-14`）：
```go
import (
    "encoding/json"
    "log/slog"
    "maps"
    "slices"
    "sync"
    "time"

    "github.com/go-musicfox/go-musicfox/internal/storage"
    "github.com/go-musicfox/go-musicfox/internal/structs"
    "github.com/go-musicfox/go-musicfox/internal/types"
)
```

---

## 5. 注释与文档规范

### 注释语言

根据 `AGENTS.md` 规定：
- **代码注释**：使用英文
- **用户交互/文档**：使用中文
- **Git Commit Message**：使用英文

### 注释风格

- **包注释**：不需要（Go 1.22+）
- **导出类型/函数**：应有文档注释
  ```go
  // PlaylistManager 播放列表管理器接口
  // 提供播放列表的核心管理功能，包括播放控制、模式切换等
  type PlaylistManager interface { ... }
  ```
- **接口方法**：应有清晰的中文注释
  ```go
  // NextSong 切换到下一首歌曲
  // manual 参数表示是否为手动切换
  NextSong(manual bool) (structs.Song, error)
  ```
- **结构体字段**：可选中英文注释
- **实现细节注释**：英文，简洁
  ```go
  // 列表循环播放模式无需特殊初始化逻辑
  func (l *ListLoopPlayMode) Initialize(...) error { ... }
  ```

### Deprecated 标记

使用标准格式：
```go
// GetPlayModeName 获取当前播放模式的名称
//
// Deprecated: please use GetPlayMode().Name() instead.
GetPlayModeName() string
```

---

## 6. 错误处理模式


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-musicfox/go-musicfox](https://github.com/go-musicfox/go-musicfox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
