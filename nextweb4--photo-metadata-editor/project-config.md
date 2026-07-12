---
trigger: always_on
description: - 当前项目根目录即本文件所在目录；文档和源码不得写入开发机绝对路径。
---

# AGENTS.md

## 1. 项目结构
- 当前项目根目录即本文件所在目录；文档和源码不得写入开发机绝对路径。
- `src/photo_meta_editor/` 是应用源码：`app.py` 负责 Tkinter 界面，`i18n.py` 负责中英文文案映射，`settings.py` 负责本地语言偏好，`exiftool.py` 负责 ExifTool 调用，`fields.py` 负责字段映射和校验，`presets.py` 负责相机/GPS预设，`ocr.py` 负责OCR引擎探测和时间解析。
- `vendor/exiftool/` 存放打包运行所需的 `exiftool.exe` 和 `exiftool_files/`。
- `scripts/build_exe.ps1` 是 Windows EXE 打包入口。
- `scripts/build_msi.ps1` 是 MSI 打包入口，`setup_msi.py` 是 cx_Freeze MSI 配置。
- `scripts/build_release.ps1` 是发布前总入口：编译、单元测试、EXE、MSI、可选受信证书签名、ZIP 和隐私扫描必须集中从这里跑。
- `scripts/generate_windows_version_info.py` 从 `metadata.py` 生成 PyInstaller 使用的 `scripts/windows_version_info.txt`。
- `scripts/collect_licenses.py` 生成分发包随附的 `licenses/` 目录，至少包含项目 `LICENSE`、`THIRD_PARTY.md`、ExifTool Windows 版说明、Strawberry Perl runtime 许可证包、PyInstaller COPYING、cx_Freeze 和 tkinterdnd2 许可证；PyInstaller 目录根部也必须保留一份可见 `licenses/`。
- `scripts/check_package_privacy.py` 扫描分发产物中的本机路径、临时附件名、桌面样例名和外部 ExifTool 样例路径。
- `scripts/prune_runtime_payload.py` 负责清理打包产物中非当前平台的 `tkinterdnd2\tkdnd` 文件和 Tk demos，避免 EXE/MSI 体积膨胀。
- `scripts/sign_artifacts.ps1` 只允许使用 `PHOTO_META_EDITOR_SIGNING_CERT_THUMBPRINT` 指定的受信代码签名证书；没有证书时产物必须保持未签名，不得创建自签名证书。
- `tests/` 存放标准库 `unittest` 测试。
- `docs/OPEN_SOURCE_AUDIT.md` 和 `docs/THIRD_PARTY.md` 记录方案审计与第三方组件。
- 当前仓库不提交 GitHub Actions 发布 workflow；本次发布使用 GitHub API 直接创建 Release。若后续恢复 `.github/workflows/release.yml`，推送凭据必须具备 `workflow` scope。
- `RELEASE_NOTES.md` 是 GitHub Release 的中英文说明；`release-assets/` 只保存待上传产物且不得提交。
- 外部 ExifTool 原始目录只作为本机开发来源，不得直接修改该目录内的样例图片。

## 2. 运行命令
- 源码运行：`$env:PYTHONPATH = "src"; python -m photo_meta_editor`
- 打包产物运行：`.\dist\PhotoMetaEditor\PhotoMetaEditor.exe`

## 3. 测试命令
- 单元测试：`$env:PYTHONPATH = "src"; python -m unittest discover -s tests`
- 修改字段映射、日期/GPS 校验、预设数据、OCR时间解析或 ExifTool 参数生成后必须运行单元测试。
- 修改 ExifTool 写入参数后必须至少保留一个真实 JPEG 写入回归测试，验证 UI 可接受的日期格式会被规范化并能被 ExifTool 读回；PNG 写入路径还必须覆盖标题与成对 GPS 坐标的真实读回，避免 JPEG 标签映射回归后误称支持常见无损图片。

## 4. 构建命令
- EXE 构建：`powershell -ExecutionPolicy Bypass -File .\scripts\build_exe.ps1`
- MSI 构建：`powershell -ExecutionPolicy Bypass -File .\scripts\build_msi.ps1`
- 完整发布构建：`powershell -ExecutionPolicy Bypass -File .\scripts\build_release.ps1`
- 构建产物目录：`dist\PhotoMetaEditor\`
- 构建脚本会排除 PaddleOCR/PyTorch/EasyOCR/winsdk 等重型或可选 OCR 依赖，避免 EXE/MSI 体积和启动复杂度失控。
- 构建后会运行 `scripts/prune_runtime_payload.py`；修改 PyInstaller 或 cx_Freeze 配置后必须确认只保留当前 Windows 平台所需的 `tkdnd` 目录。
- PyInstaller 构建前必须拒绝 `build/`、`dist/`、`build/pyinstaller-spec/` 及其 `PhotoMetaEditor/licenses` 子目录中的 Windows reparse point，并确认解析后的路径仍在项目根目录内；生成的 `.spec` 必须写入受保护的 `build/pyinstaller-spec/`，构建脚本不得通过目录联接删除或覆盖项目外文件。
- 对 `build/`、`dist/` 或其子目录执行递归删除/替换前，必须同时拒绝目标目录内部嵌套的 Windows reparse point；不能只检查顶层目录。
- Python 发布辅助脚本执行 `shutil.rmtree` 前必须调用 `scripts/path_safety.py` 的 reparse point 检查；脚本既要支持被测试导入，也要支持 `python scripts\*.py` 直接执行。
- PyInstaller 使用独立 `--specpath` 后，`--add-binary`、`--add-data`、`--paths`、版本文件和入口脚本必须传入项目根目录推导出的绝对路径；不得依赖 spec 目录改变前的相对路径解析。
- MSI 必须在 CostInitialize 前通过 Type 51 CustomAction 把安装会话的 `PersonalFolder` 设为 `[LocalAppDataFolder]`、把 `ROOTDRIVE` 设为 `[WindowsVolume]`；不得修改用户注册表。否则重定向到不可用盘的 Shell Folder 可能令真实安装在 `CostFinalize` 返回 1606/1603。
- cx_Freeze MSI 的非根 Directory identifier 必须由 `BuildMsiCommand` 生成为稳定的 `PME_DIR_*`；不得直接使用 ExifTool runtime 的 `Text`、`File` 等目录名作为 MSI 属性，否则可能与 Windows Installer 全局状态冲突并导致 CostFinalize 1606。

## 5. 代码风格
- 当前未发现 lint / format 命令。
- 新增代码时优先保持少依赖、清晰模块边界，不把 GUI、文件解析、ExifTool 调用混在同一个大文件中。
- Python 代码使用类型标注；业务逻辑优先放在可单测函数中。
- `tkinterdnd2` 是拖拽导入所需运行依赖；OCR依赖保持可选探测，不允许运行时自动下载模型，PaddleOCR 只能使用本地缓存模型。
- 默认分发包不内置 `winsdk`、PaddleOCR、EasyOCR、PyTorch；Windows OCR 只在源码环境安装了 `winsdk` 时可用。
- 应用元信息、作者、邮箱、网站必须统一来自 `src/photo_meta_editor/metadata.py`，不要在 UI、打包脚本和 MSI 配置中各自硬编码不同值。
- 不要手改 `scripts/windows_version_info.txt` 的作者、版本、邮箱或网站；应修改 `metadata.py` 后运行 `scripts/generate_windows_version_info.py` 或完整构建。
- 对外发布会改变已安装 MSI 行为、EXE 版本资源或分发内容时，必须先递增 `metadata.py` 的数字化 `APP_VERSION`，同步 `pyproject.toml`、README 产物名并通过 `tests/test_metadata.py`；cx_Freeze 的 ProductCode 每次生成，保持旧版本号会让安装升级语义不可靠。
- README 和中文文档必须保持 UTF-8 可读中文，不允许保存成 mojibake；修改 README 后必须运行 `python -m unittest discover -s tests -p test_metadata.py`。
- UI 新增或修改可见文案时必须同时补齐 `i18n.py` 的英文翻译；语言切换不得改变当前文件、编辑值、预设或忙碌状态。
- 语言偏好只能写入用户 AppData 的 `PhotoMetadataEditor/settings.json`，不得写入项目目录或发起联网请求。

## 6. 模块边界
- GUI 只负责用户交互、状态展示和调用服务层。
- 图片元数据读写必须通过 ExifTool 调用封装层完成。
- ExifTool 调用必须以命令行前缀 `-config "" -@ -` 禁用外部默认配置，并把其余参数以 UTF-8 C-string 输入流传递；中文路径、空格路径、反斜杠和多行元数据必须通过真实读写回归测试。
- 元数据字段读取、写入标签映射和校验必须放在 `fields.py`，不要散落到 GUI 事件里。
- iPhone“照片信息”卡片同类字段（文件名、格式、尺寸、像素、文件大小、镜头、ISO、焦距、曝光补偿、光圈、快门）必须在左侧信息区直接显示；不可写入的事实字段必须标记为只读，并在 `build_tag_assignments` 中跳过。
- UI 中可写字段和只读照片参数必须在左侧同一个信息面板内分区展示，不允许把照片参数藏在只有切换标签页才看到的位置，也不允许混成一条长表单导致用户误判哪些字段能保存。
- 相机型号和GPS预设只能放在 `presets.py`，不要在 GUI 回调中硬编码。
- OCR文本识别和时间解析只能放在 `ocr.py`，GUI 只调用服务函数。
- PaddleOCR 调用前必须强制 `PADDLE_PDX_MODEL_SOURCE=LOCAL` 并确认本地模型缓存存在；不得被外部环境变量改成联网模型源。
- 用户输入的拍摄时间可以接受 `YYYY:MM:DD`、`YYYY-MM-DD`、`YYYY/MM/DD`、`YYYY.MM.DD` 和中文年月日格式，但写入 ExifTool 前必须统一规范化为 `YYYY:MM:DD HH:MM:SS`。
- 编辑器可把文件名作为空标题的显示回填，但 ExifTool 写入后的读回校验必须比较原始存储字段；清空标题时不得把该显示回填误判成写入失败。
- 编辑器可根据镜头信息推断 iPhone 厂商和型号用于显示，但 ExifTool 写入后的读回校验必须跳过该推断；清空实际 `Make`/`Model` 时不得误报失败。
- GPS 读取必须正确处理十进制度数、DMS 度分秒格式、DM 度加小数分格式、半球方向和 ISO6709 组合坐标；不得用“取前两个数字”的方式解析带单位的组合坐标。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NextWeb4/photo-metadata-editor](https://github.com/NextWeb4/photo-metadata-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
