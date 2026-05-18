---
trigger: always_on
description: 热更新系统开发规范
---


# 热更新系统开发规范

## 热更新架构设计

### 管理器单例模式
```typescript
export class HotUpdateManager {
    private static _instance: HotUpdateManager;
    
    /** 获取单例实例 */
    public static getInstance(): HotUpdateManager {
        if (!this._instance) {
            this._instance = new HotUpdateManager();
        }
        return this._instance;
    }
    
    /** 禁用直接构造 */
    private constructor() {}
    
    /** 配置属性 */
    public manifestUrl: string = "";
    public versionUrl: string = "";
    
    /** 初始化热更新 */
    public init(manifestUrl: string, versionUrl: string): void {
        this.manifestUrl = manifestUrl;
        this.versionUrl = versionUrl;
    }
}
```

### 热更新状态码定义
```typescript
export enum HotUpdateCode {
    /** 成功 */
    Succeed = 0,
    /** 已是最新版本 */
    LatestVersion = 1,
    /** 检查更新失败 */
    CheckFailed = 2,
    /** 下载失败 */
    DownloadFailed = 3,
    /** 解压失败 */
    UnzipFailed = 4,
    /** 网络错误 */
    NetworkError = 5,
    /** 空间不足 */
    NoSpace = 6,
    /** 未知错误 */
    UnknownError = 7
}
```

## Promise 结果模式

### 统一结果接口
```typescript
export interface IPromiseResult {
    /** 状态码 */
    code: HotUpdateCode;
    
    /** 消息描述 */
    message: string;
    
    /** 扩展数据 */
    data?: any;
}
```

### 热更新配置接口
```typescript
export interface IHotUpdateConfig {
    /** 版本号 */
    version: string;
    
    /** 远程manifest文件URL */
    remoteManifestUrl: string;
    
    /** 远程version文件URL */
    remoteVersionUrl: string;
    
    /** 资源包URL */
    packageUrl: string;
    
    /** 资源文件列表 */
    assets?: { [key: string]: any };
    
    /** 搜索路径 */
    searchPaths?: string[];
}
```

## 热更新核心实现

### HotUpdate 核心类
```typescript
export class HotUpdate {
    private _am: jsb.AssetsManager;
    private _updating: boolean = false;
    
    constructor(manifestUrl: string) {
        // 初始化 AssetsManager
        this._am = new jsb.AssetsManager(manifestUrl, jsb.fileUtils.getWritablePath() + 'remote-assets');
        this._am.setEventCallback(this.onUpdateEvent.bind(this));
        this._am.setVerifyCallback(this.onVerifyCallback.bind(this));
    }
    
    /**
     * 检查更新
     * @returns Promise<IPromiseResult>
     */
    public checkUpdate(): Promise<IPromiseResult> {
        return new Promise((resolve) => {
            if (this._updating) {
                resolve({ code: HotUpdateCode.UnknownError, message: "正在更新中" });
                return;
            }
            
            if (!this._am.getLocalManifest() || !this._am.getLocalManifest().isLoaded()) {
                resolve({ code: HotUpdateCode.CheckFailed, message: "本地manifest加载失败" });
                return;
            }
            
            this._am.setEventCallback((event) => {
                switch (event.getEventCode()) {
                    case jsb.EventAssetsManager.ERROR_NO_LOCAL_MANIFEST:
                        resolve({ code: HotUpdateCode.CheckFailed, message: "本地manifest不存在" });
                        break;
                    case jsb.EventAssetsManager.ERROR_DOWNLOAD_MANIFEST:
                        resolve({ code: HotUpdateCode.NetworkError, message: "下载manifest失败" });
                        break;
                    case jsb.EventAssetsManager.ALREADY_UP_TO_DATE:
                        resolve({ code: HotUpdateCode.LatestVersion, message: "已是最新版本" });
                        break;
                    case jsb.EventAssetsManager.NEW_VERSION_FOUND:
                        resolve({ code: HotUpdateCode.Succeed, message: "发现新版本" });
                        break;
                }
            });
            
            this._am.checkUpdate();
        });
    }
    
    /**
     * 执行热更新
     * @returns Promise<IPromiseResult>
     */
    public hotUpdate(): Promise<IPromiseResult> {
        return new Promise((resolve) => {
            if (this._updating) {
                resolve({ code: HotUpdateCode.UnknownError, message: "正在更新中" });
                return;
            }
            
            this._updating = true;
            
            this._am.setEventCallback((event) => {
                switch (event.getEventCode()) {
                    case jsb.EventAssetsManager.UPDATE_FINISHED:
                        this._updating = false;
                        resolve({ code: HotUpdateCode.Succeed, message: "更新完成" });
                        break;
                    case jsb.EventAssetsManager.UPDATE_FAILED:
                        this._updating = false;
                        resolve({ code: HotUpdateCode.DownloadFailed, message: "更新失败" });
                        break;
                    case jsb.EventAssetsManager.ERROR_DECOMPRESS:
                        this._updating = false;
                        resolve({ code: HotUpdateCode.UnzipFailed, message: "解压失败" });
                        break;
                }
            });
            
            this._am.update();
        });
    }
    
    /**
     * 事件回调处理
     */
    private onUpdateEvent(event: jsb.EventAssetsManager): void {
        const code = event.getEventCode();
        debug(`热更新事件: ${code}`);
        
        switch (code) {
            case jsb.EventAssetsManager.UPDATE_PROGRESSION:
                const progress = event.getPercent();
                debug(`更新进度: ${progress}%`);
                break;
            case jsb.EventAssetsManager.ASSET_UPDATED:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gongxh0901/kunpolibrary](https://github.com/gongxh0901/kunpolibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
