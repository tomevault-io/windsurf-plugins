---
trigger: always_on
description: 桌面端表情包管理系统（OhMyMeme）的安卓移植版。存储结构、数据库 schema、导入/扫描/缩略图命名规则与桌面端 `D:\code\OhMyMeme` 完全一致，便于多端同步。
---

# OhMyMeme Android — AI Agent Guide

## 项目概述
桌面端表情包管理系统（OhMyMeme）的安卓移植版。存储结构、数据库 schema、导入/扫描/缩略图命名规则与桌面端 `D:\code\OhMyMeme` 完全一致，便于多端同步。

## 架构
```
MainActivity / SettingsActivity
        │ 调用
MemeDb (SQLite WAL) ──► files/memes.db
StoragePaths          ──► config:Android/data/com.ohmymeme.app/  localdata:files/
ConfigStore + CryptoUtil ──► config.json（密钥 AES-GCM 加密）
    CacheScanner / MemeImporter / Thumbnailer ──► data/cache/、data/thumbnails/
    CloudSync ──► 远端 memes/ + meme-index.json（FTP/S3/R2/WebDAV）
```

## 技术栈
- **Kotlin** + AppCompat + RecyclerView + ConstraintLayout（无 Compose）
- **AGP 9.0.0** + Gradle 9.1，依赖用 `gradle/libs.versions.toml` 版本目录管理
- **SQLite** (WAL)，schema 与桌面端 `src/database.py` 一致
- **Android Keystore** (AES-GCM) 加密配置密钥字段
- **SAF**（Storage Access Framework）批量导入，免存储权限
- minSdk 28 / targetSdk 36 / compileSdk 36

## 核心原则
- **不重构桌面端** — 桌面端 `D:\code\OhMyMeme` 仅做最小必要修改；如需同步桌面端数据层逻辑，以 `database.py`/`config.py`/`webui.py` 为唯一事实来源
- **存储结构对齐桌面端** — 表 schema、列名、重命名规则、缩略图命名、去重逻辑逐条对照，不得随意改动
- **增改同步** — 新功能/新文件必须同步更新 `README.md` 和 `AGENTS.md`
- **无 emoji**（除非用户要求）
- **代码风格** — 无冗余注释；单线程 Executor 跑数据库/IO，`runOnUiThread` 回主线程更新 UI；Kotlin 按语言惯例写类型标注

## 关键目录
```
app/src/main/
  java/com/ohmymeme/app/
    MainActivity.kt     # 主界面：导入/刷新/搜索/网格 + 空状态
    SettingsActivity.kt # 设置页：loadConfig/saveConfig/reset 接真实配置
    ChipAdapter.kt      # 分组胶囊适配器（仅 COLLECTION 样式）
    MemeGridAdapter.kt  # 表情网格，异步缩略图加载，按 meme.id 打 tag 防错位
    Meme.kt             # 数据模型（对应 memes 表，含 stego/fromStego 字段）
    MemeDb.kt           # SQLite 封装（7 表 + 索引 + 列迁移）
    ConfigStore.kt      # JSON 配置（DEFAULTS 与桌面端 config.py 一致）
    CryptoUtil.kt       # Android Keystore AES-GCM 加解密
    StoragePaths.kt     # 路径解析（base/data/cache/thumbnails/db/config）
    FileUtils.kt        # SHA-256 + 魔数识别扩展名
    CacheScanner.kt     # 缓存扫描（双重去重）
    MemeImporter.kt     # SAF 批量导入
    GifFrameDecoder.kt  # 自研最小 GIF 解码器（LZW/interlace/色板，与 Pillow 一致）
    GifEncoder.kt       # 自研最小 GIF 编码器（median cut 256 色 + LZW，与 GifFrameDecoder 严格对应）
    GifStego.kt         # STG3 隐写检测 + 7 模式解码 + encode 写入（FULL/LZMA/WebP 候选）+ 自研 PNG 编码
    AndroidGifDecoder.kt# 设备端 WebP→RGBA（反预乘 alpha）
    Thumbnailer.kt      # 缩略图生成 {id}_{size}.png
    MemeCopyProcessor.kt# 复制处理：分享前按 copy_resize_mode 缩放 WebP / 转 GIF / 转隐写 GIF
    CloudSync.kt        # 云端同步（FTP/S3/R2/WebDAV + meme-index.json 清单）
    LanClient.kt        # 局域网互联客户端（UDP 发现 + TCP 握手 + AES-GCM 会话）
    UpdateChecker.kt    # 版本更新检查（GitHub Releases API）
  res/
    layout/activity_main.xml / activity_settings.xml / item_*
    values/colors.xml   # 暗色配色（bg #0D0D0F、card #1E1E22、accent #3B82F6、muted #71717A）
    values/themes.xml   # Theme.OhMyMeme（含 values-night）
    values/strings.xml  # 含 copy_mode_options / sync_type_options
```

## 存储布局（与桌面端对应）
```
Android/data/com.ohmymeme.app/
├── config.json                       ← 桌面端 %APPDATA%/OhMyMeme/config.json
└── files/                            ← localdata（对应桌面端 %LOCALAPPDATA%/OhMyMeme）
    ├── memes.db
    ├── cache/                        ← 导入原图，命名 {sha256前16位}{ext}
    └── thumbnails/                   ← {meme_id}_{size}.png（默认 150）
```

## 关键实现细节

### 数据库（MemeDb.kt）
- 7 表：`memes`/`tags`/`meme_tags`/`collections`/`meme_collections`/`favorites`/`recent_uses`，字段与桌面端 `database.py` 逐列一致
- `PRAGMA`：`enableWriteAheadLogging()` 替代桌面端 `journal_mode=WAL`；外键依赖 ON DELETE CASCADE
- `getCollectionDepth` 在安卓上按 `pid == 0L` 判根（SQLite parent_id 无 NULL 时以 0 存储）
- 列迁移：与桌面端相同的 `ALTER TABLE ... ADD COLUMN` 容错迁移
- 单例：`MemeDb.get(context)`，用 `applicationContext` 防泄漏

### 配置（ConfigStore.kt）
- `DEFAULTS` 逐字段照搬桌面端 `config.py`（含 `s3_path`、`webdav_timeout` 等）
- `SECRET_KEYS` 6 个密钥字段（s3_access_key/s3_secret_key/r2_access_key_id/r2_secret_access_key/ftp_password/webdav_password）写入前加密、读取后解密
- `load()` 在读取时对密钥字段先解密；`save()` 加密副本后写盘；损坏文件回退默认值；**首次运行文件不存在时自动落盘默认配置**
- 与桌面端差异：桌面端 Fernet，安卓端用 Android Keystore（硬件背书），格式不互通但字段名一致

### 首次运行存储位置（MainActivity.kt）
- `StoragePaths.isFirstRun` 检测（SharedPreferences 标记 `setup_done`），首次启动弹窗二选一：默认位置（应用专属外部目录）或「选择其他位置」（SAF `ACTION_OPEN_DOCUMENT_TREE`）
- `StoragePaths.resolveTreeUriPath` 把 SAF 树 URI 解析为真实路径（`primary:`→外部存储根，`home:`→Downloads），解析失败回退默认并提示；`setDataDir` 覆盖 localdata 目录
- 选完位置后 `markSetupDone` + `ConfigStore.invalidate` 再加载数据

### 导入（MemeImporter.kt）
- 点击标题栏「导入」弹 `menu_import.xml` 菜单：从文件导入（`pickImages`，SAF `ACTION_OPEN_DOCUMENT` 多选）/ 从手机相册导入（`pickAlbumImages`：`isPhotoPickerAvailable` 为真走 Photo Picker `PickMultipleVisualMedia`，否则回退 `ACTION_GET_CONTENT` 相册，避免库默认回退文件选择器）/ 从手机QQ缓存导入（占位 Toast「开发中，后续用 Shizuku 获取文件」）
- 两种导入共用 `doImport(uris)` → `MemeImporter.importUris`：逐文件：查哈希去重 → 魔数识别扩展名 → 拷贝到 `cache/{hash16}{ext}` → 读尺寸 → `addMeme`
- 单文件失败不影响其余文件（catch 后继续）
- **隐写 GIF 解码**（对应桌面端 `_try_decode_stego` + `gif_stego.py`）：GIF 且含 `STG3` 时 `GifStego.decode` 还原原图（7 种模式），只导入还原结果 `fromStego=1`，GIF 本身不入库；WebP 模式经 `AndroidGifDecoder.webpToRgba` 解码（反预乘 alpha）
- `GifFrameDecoder` 为自研最小 GIF 解码器（GIF87a/89a、全局/局部色板、LZW、interlace、透明索引直映射 RGB、Pillow 一致的 `(R*299+G*587+B*114+500)/1000` 灰度），与 Pillow 逐字节一致；LZMA 用 `org.tukaani:xz`（`XZInputStream`）；PNG 输出用自研 RGBA 编码器
- `CacheScanner` 不做 STG3 检测（对齐桌面端 `scan_cache`）

### 缓存扫描（CacheScanner.kt）
- 遍历 cache 目录：跳过非图片扩展名、`thumbnails` 路径、与同名 `.webp` 共存的 `.gif`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OhMyMeme/OhMyMeme-Android](https://github.com/OhMyMeme/OhMyMeme-Android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
