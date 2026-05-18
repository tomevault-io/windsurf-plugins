---
trigger: always_on
description: 小游戏平台开发规范
---


# 小游戏平台开发规范

## 平台检测和分类

### 平台类型定义
```typescript
export enum PlatformType {
    Unknown = 0,
    Browser = 1,
    WX = 2,          // 微信小游戏
    Alipay = 3,      // 支付宝小游戏  
    Bytedance = 4,   // 字节跳动小游戏
    HuaweiQuick = 5  // 华为快游戏
}

export class Platform {
    /** 平台类型 */
    public static platform: PlatformType = PlatformType.Unknown;
    
    /** 平台标识 */
    public static isWX: boolean = false;
    public static isAlipay: boolean = false;
    public static isBytedance: boolean = false;
    public static isHuaweiQuick: boolean = false;
    public static isBrowser: boolean = false;
    
    /** 设备类型 */
    public static isNative: boolean = false;
    public static isMobile: boolean = false;
    public static isNativeMobile: boolean = false;
    
    /** 系统类型 */
    public static isAndroid: boolean = false;
    public static isIOS: boolean = false;
    public static isHarmonyOS: boolean = false;
}
```

### 平台初始化模式
```typescript
export class CocosEntry extends Component {
    private initPlatform(): void {
        // 设备类型检测
        Platform.isNative = sys.isNative;
        Platform.isMobile = sys.isMobile;
        Platform.isNativeMobile = sys.isNative && sys.isMobile;

        // 系统类型检测
        switch (sys.os) {
            case sys.OS.ANDROID:
                Platform.isAndroid = true;
                debug("系统类型 Android");
                break;
            case sys.OS.IOS:
                Platform.isIOS = true;
                debug("系统类型 IOS");
                break;
            case sys.OS.OPENHARMONY:
                Platform.isHarmonyOS = true;
                debug("系统类型 HarmonyOS");
                break;
        }

        // 平台类型检测
        switch (sys.platform) {
            case sys.Platform.WECHAT_GAME:
                Platform.isWX = true;
                Platform.platform = PlatformType.WX;
                break;
            case sys.Platform.ALIPAY_MINI_GAME:
                Platform.isAlipay = true;
                Platform.platform = PlatformType.Alipay;
                break;
            case sys.Platform.BYTEDANCE_MINI_GAME:
                Platform.isBytedance = true;
                Platform.platform = PlatformType.Bytedance;
                break;
            case sys.Platform.HUAWEI_QUICK_GAME:
                Platform.isHuaweiQuick = true;
                Platform.platform = PlatformType.HuaweiQuick;
                break;
            default:
                Platform.isBrowser = true;
                Platform.platform = PlatformType.Browser;
                break;
        }
        
        debug(`platform: ${PlatformType[Platform.platform]}`);
    }
}
```

## 平台适配器设计

### 通用适配器接口
```typescript
export interface IMiniGameAdapter {
    /** 显示分享菜单 */
    showShareMenu(): void;
    
    /** 分享应用 */
    shareAppMessage(options: ShareOptions): void;
    
    /** 显示 loading */
    showLoading(options: LoadingOptions): void;
    
    /** 隐藏 loading */
    hideLoading(): void;
    
    /** 显示 toast */
    showToast(options: ToastOptions): void;
    
    /** 获取系统信息 */
    getSystemInfo(): Promise<SystemInfo>;
    
    /** 震动反馈 */
    vibrateShort(): void;
    vibrateLong(): void;
}
```

### 微信小游戏适配
```typescript
export class WechatCommon implements IMiniGameAdapter {
    public showShareMenu(): void {
        if (Platform.isWX && wx.showShareMenu) {
            wx.showShareMenu({
                withShareTicket: true,
                menus: ['shareAppMessage', 'shareTimeline']
            });
        }
    }
    
    public shareAppMessage(options: ShareOptions): void {
        if (Platform.isWX && wx.shareAppMessage) {
            wx.shareAppMessage({
                title: options.title,
                imageUrl: options.imageUrl,
                query: options.query,
                success: options.success,
                fail: options.fail
            });
        }
    }
    
    public showLoading(options: LoadingOptions): void {
        if (Platform.isWX && wx.showLoading) {
            wx.showLoading({
                title: options.title || '加载中...',
                mask: options.mask !== false
            });
        }
    }
    
    public getSystemInfo(): Promise<SystemInfo> {
        return new Promise((resolve, reject) => {
            if (Platform.isWX && wx.getSystemInfo) {
                wx.getSystemInfo({
                    success: resolve,
                    fail: reject
                });
            } else {
                reject(new Error('不支持的平台'));
            }
        });
    }
}
```

### 支付宝小游戏适配
```typescript
export class AlipayCommon implements IMiniGameAdapter {
    public showShareMenu(): void {
        // 支付宝特定实现
    }
    
    public shareAppMessage(options: ShareOptions): void {
        if (Platform.isAlipay && my.shareAppMessage) {
            my.shareAppMessage({
                title: options.title,
                desc: options.desc,
                path: options.path,
                success: options.success,
                fail: options.fail
            });
        }
    }
    
    public showLoading(options: LoadingOptions): void {
        if (Platform.isAlipay && my.showLoading) {
            my.showLoading({
                content: options.title || '加载中...'
            });
        }
    }
}
```

### 字节跳动小游戏适配
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gongxh0901/kunpolibrary](https://github.com/gongxh0901/kunpolibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
