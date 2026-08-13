---
trigger: always_on
description: 纯本地处理的相册浏览器。从原生 JS（~17000 行）重构为 Vue3 + Vite + Pinia。
---

# 相册浏览器 PWA

纯本地处理的相册浏览器。从原生 JS（~17000 行）重构为 Vue3 + Vite + Pinia。
**纯本地文件处理、不联网**（File System Access API，仅 Chrome/Edge/Opera）。

## 技术栈

- **Vue 3**（`<script setup>`，纯 JS，无 TypeScript）
- **Vite 5** + 双 build：`vite.config.js`（单 HTML）/ `vite.config.pwa.js`（PWA）
- **Pinia**（setup store 风格）
- **资源全内化**（零 CDN）：`@fortawesome/fontawesome-free` / `spark-md5` / `exifr` 均 npm 装 + ES import
- **GalleryDB**（自建 IndexedDB，五 store，无第三方 KV 依赖）：`thumbnails`/`file-meta`/`user-data` 按 md5 三分类 + `roots`(多根句柄)/`scans`(扫描快照) 按 rootId KV。见 [db.js](src/services/db.js)
- **@tanstack/vue-virtual**：Gallery 按行虚拟化（万图不卡，整页滚动 + 实测行高）
- **质量基建**：`@antfu/eslint-config`（ESLint flat config）+ Vitest + `@vue/test-utils` + jsdom

## 常用命令

```bash
npm run dev        # 开发（用户跑）
npm run build      # 单 HTML(dist/index.html 自包含,可 file:// 离线)
npm run build:pwa  # PWA(SW + manifest,可安装/离线)
npm run icons      # 从 public/icon.svg 生成 192/512 PNG(@resvg/resvg-js)
npm run lint       # ESLint 检查(@antfu/eslint-config)
npm run lint:fix   # 自动修格式
npm run test       # Vitest 跑测试
npm run test:watch # 监听模式
```

## 目录结构

```
src/
├── main.js              # createApp + Pinia + 全局 CSS + font-awesome
├── App.vue              # 根布局(启动页/主界面 + 全局浮层 + 启动恢复多根)
├── config/              # CONFIG + UserSettings、FileTypes(纯数据)
├── models/              # SmartFile/SmartFolder(纯数据类+派生 getter)+ 同文件模块函数(scanFolder/enrichFolder/record/CRUD/validate,P3 函数化)
├── services/            # fileResource(资源池) / persistence+scanIntegration+folderActions(T03 拆自 filesystem) / handleStore(多根句柄) / scanCache(快照) / webkitDirectory(降级只读建树/扫描/指纹) / thumbnail+thumbnail-strategies+thumbnail-worker-pool(createImageBitmap) / fileMeta+userData(md5 索引门面) / metadata / db / recovery / operations(.trash) / fileOps / exif / gps / id3-parser
├── stores/              # Pinia: fs(含 dirtyFolders) / root(多根元数据) / favorites+notes(md5 聚合) / modal / theme / userSettings / history / contextMenu / confirm / properties / uiToast
├── composables/         # useThumbnail / useModal / useSidebar(边缘拖拽调宽) / useScrollZone / useGallerySearch / useOverlay(浮层 dismiss) / useFileActions / useMediaActions / useHoveredFile / useStorageEstimate
├── utils/               # concurrency(runConcurrent+cancelToken) / gallery-layout(虚拟化布局纯函数) / format / file(calculateMD5+md5.worker) / browser / coverFit / mediaSession
├── components/          # Gallery(按行虚拟化) / PhotoCard / Sidebar / RootSwitcher / SidebarTreeItem / MediaModal / AudioPlayer / SettingsPanel / PropertiesPanel / ContextMenu / ConfirmDialog / Toast / BrowserUnsupportedWarning
└── styles/              # 全局 CSS(main.js import,组件用其 class)
docs/superpowers/        # specs(设计 + 实现记录)+ plans(实施计划)
后续待办.md              # 跨阶段遗留事项
改造路线图.md            # 重构后的改进路线(质量基建/句柄持久化/多文件夹/约定现代化)
```

## 关键约定（请遵守）

1. **model 层纯数据 + 模块函数、副作用归 service**（`models/` 不 import Pinia/Vue、不反向依赖 store）：
   - `SmartFile`/`SmartFolder` 是**纯数据类**（字段 + 派生 getter，无实例方法）；所有行为是**同文件模块级函数**：`scanFolder` / `enrichFolder` / `folderToRecord` / `foldersFromRecordMap` / `createFolder` / `validateFolder` / `ensureBlobUrl` / `renameFile` / `moveFile` / `disposeFile` 等。`scanFolder(folder,{trust})` 纯函数——不改入参、不碰 store、不 dispose，返回增删结果集。模板用的派生 getter 保留——Vue 响应式追踪属性访问，**勿函数化**。
   - **副作用集中 service 层**：`integrateScanResult(folder,result,fs)`（写回**代理** folder + 注册/删 foldersData + `disposeFile` removedFiles + `markFolderDirty` 标脏）、`registerFolderTree`、`resetFoldersData`、`registerAndIntegrate`（P0-2：收口"set 进 Map→get 取代理→integrate"，新建 folder 必走）。⚠️ **写回必须是「代理」**（从 store 取或 `foldersData.get(path)`）；`createFolder` 返回的原始对象直接写回不触发响应式（见下方 reactive 陷阱）。
2. **service 层操作 store**：`services/` 内部 `useFsStore()` / `useToastStore()` 等直接调（在函数体内，不在模块顶层）。
3. **资源走 fileResource 池**（[fileResource.js](src/services/fileResource.js)）：blobUrl/File 集中管理（`acquire`/`destroy`/`peek`，in-flight 去重+cancel）。SmartFile 是其门面。**不要直接 `URL.createObjectURL`/`revokeObjectURL`**；size/mtime 单源在 `SmartFile._meta`（响应式），不进池。
4. **持久化走 schedulePersist(per-folder,治写放大)**:改树(增删/改名/移动/md5/duration)由 `integrateScanResult`(`markFolderDirty`)或 `afterFolderMutation(folder)` 标**该文件夹**脏 → `fs.dirtyFolders`(Set<rootId::path>) + `schedulePersist()`(1s debounce)。`persistIfDirty` 只遍历 dirty 集合、**每夹写一条 record**(`saveFolderRecord(rootId,folder)`→`scans`,非递归),`fileCount` 变才写 `roots`。**不要直接 `saveFolderRecord`/`folderToRecord`**。切根前 `flushPendingPersist` 落盘旧根(reload 用 `cancelPendingPersist` 重扫);重建走 `foldersFromRecordMap`(`loadScan` 前缀拉全 record→Map,秒切)。`visibilitychange:hidden` 触发 `flushPendingPersist` 兜底(P0-3)。
5. **CSS 全局复用**：`src/styles/` 的全局 CSS（`main.js` 全局 import）。组件**不重写这些 CSS**，模板直接用其 class（如 `.photo-card` / `.gallery-row` / `.tree-node` / `.modal-audio-player`）。组件 scoped 样式只补 CSS 里没有的。
6. **核心算法稳定**：scan 纯列表差集 + 信任名字集合短路、enrich 并发 getFile、GPS/ID3/.trash、calculateMD5（前 2MB 内容寻址缓存键——跨文件夹同图共享缩略图、md5 随快照持久化→秒切零重算；**chunkSize 锁死不动**）。**数据五分收口 GalleryDB，不再用 idb-keyval**：按 md5 的 `thumbnails`(blob LRU)/`file-meta`(duration/宽高/bitrate/capturedAt/gps/exifChecked)/`user-data`(favorites+notes 聚合) + 按 rootId 的 `roots`(多根句柄)/`scans`(per-folder record)，均懒加载、经 `db.js` kv 接口读写。duration 作 `SmartFile._meta` 运行时缓存，不随快照持久化。**EXIF 拍摄时间/GPS** 走 file-meta(md5 索引)：`loadCardMetadata` 图片懒抽一次(`exifChecked` 哨兵防反复、存量首次视窗自动回填),属性面板打开重抽**变了才更新**。改动配测试。
7. **跨组件状态进 Pinia store；组件私有状态用 `ref`/`reactive`**。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HaujetZhao/Gallery-Viewer](https://github.com/HaujetZhao/Gallery-Viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
