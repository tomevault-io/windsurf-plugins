---
trigger: always_on
description: 强类型数据绑定系统开发规范
---


# 强类型数据绑定系统规范

## 数据基类设计

### DataBase 基类模式
```typescript
export class DataBase {
    /** 数据变化监听器 */
    private _watchers: Map<string, Function[]> = new Map();
    
    /**
     * 注册属性监听器
     * @param path 属性路径
     * @param callback 回调函数
     */
    public watch(path: string, callback: Function): void {
        if (!this._watchers.has(path)) {
            this._watchers.set(path, []);
        }
        this._watchers.get(path)!.push(callback);
    }
    
    /**
     * 触发属性变化通知
     * @param path 属性路径
     * @param value 新值
     */
    protected notify(path: string, value: any): void {
        if (this._watchers.has(path)) {
            this._watchers.get(path)!.forEach(callback => {
                callback(value);
            });
        }
    }
}
```

### 数据类定义规范
```typescript
class GameData extends DataBase {
    private _level: number = 1;
    private _coins: number = 0;
    private _items: Item[] = [];
    
    // 使用 getter/setter 实现响应式
    get level(): number {
        return this._level;
    }
    
    set level(value: number) {
        if (this._level !== value) {
            this._level = value;
            this.notify('level', value);
        }
    }
    
    get coins(): number {
        return this._coins;
    }
    
    set coins(value: number) {
        if (this._coins !== value) {
            this._coins = value;
            this.notify('coins', value);
        }
    }
    
    // 复杂属性的变化通知
    addItem(item: Item): void {
        this._items.push(item);
        this.notify('items', this._items);
        this.notify('items.length', this._items.length);
    }
}
```

## 装饰器绑定系统

### 强类型属性绑定
```typescript
export namespace data {
    /**
     * 强类型属性绑定装饰器
     * @param dataClass 数据类构造函数 
     * @param selector 类型安全的路径选择器
     * @param callback 变化回调函数
     * @param immediate 是否立即触发
     */
    export function bindProp<T extends DataBase>(
        dataClass: new () => T, 
        selector: (data: T) => any, 
        callback: (item: any, value?: any, data?: T) => void, 
        immediate: boolean = false
    ) {
        return function (target: any, prop: string | symbol) {
            const path = `${dataClass.name}:${extractPathFromSelector(selector)}`;
            
            let ctor = target.constructor;
            ctor[BIND_METADATA_KEY] = ctor[BIND_METADATA_KEY] || [];
            ctor[BIND_METADATA_KEY].push({
                prop, callback, path, immediate, isMethod: false
            });
        };
    }
}
```

### 方法绑定装饰器
```typescript
/**
 * 强类型方法绑定装饰器
 * @param dataClass 数据类构造函数
 * @param selector 类型安全的路径选择器  
 * @param immediate 是否立即触发
 */
export function bindMethod<T extends DataBase>(
    dataClass: new () => T, 
    selector: (data: T) => any, 
    immediate: boolean = false
) {
    return function (target: any, method: string | symbol, descriptor?: PropertyDescriptor) {
        const path = `${dataClass.name}:${extractPathFromSelector(selector)}`;
        
        let ctor = target.constructor;
        ctor[BIND_METADATA_KEY] = ctor[BIND_METADATA_KEY] || [];
        ctor[BIND_METADATA_KEY].push({
            prop: method, 
            callback: descriptor!.value, 
            path, 
            immediate, 
            isMethod: true
        });
        return descriptor;
    };
}
```

## 使用示例

### UI 数据绑定示例
```typescript
class PlayerData extends DataBase {
    name: string = "";
    level: number = 1;
    exp: number = 0;
    maxExp: number = 100;
    
    // 计算属性
    get expProgress(): number {
        return this.exp / this.maxExp;
    }
}

@uiclass("main", "player", "PlayerPanel")  
export class PlayerPanel extends Window {
    @uiprop nameLabel: GLabel;
    @uiprop levelLabel: GLabel;
    @uiprop expBar: GProgressBar;
    
    // 绑定玩家名称到标签
    @data.bindProp(PlayerData, data => data.name, function(item, value) {
        this.nameLabel.text = value;
    })
    private _nameBinding: any;
    
    // 绑定等级显示
    @data.bindMethod(PlayerData, data => data.level)
    private onLevelChanged(value: number): void {
        this.levelLabel.text = `Lv.${value}`;
    }
    
    // 绑定经验条
    @data.bindMethod(PlayerData, data => data.expProgress)  
    private onExpChanged(progress: number): void {
        this.expBar.value = progress * 100;
    }
    
    protected onInit(): void {
        // 初始化绑定
        data.initializeBindings(this);
    }
    
    protected onClose(): void {
        // 清理绑定
        data.cleanupBindings(this);
    }
}
```

## 绑定管理器

### BindManager 设计
```typescript
export class BindManager {
    private static _bindings: Map<string, BindInfo[]> = new Map();
    
    /**
     * 添加绑定信息
     */
    public static addBinding(bindInfo: BindInfo): void {
        const key = bindInfo.path;
        if (!this._bindings.has(key)) {
            this._bindings.set(key, []);
        }
        this._bindings.get(key)!.push(bindInfo);
        
        // 如果需要立即触发
        if (bindInfo.immediate) {
            this.triggerBinding(bindInfo);
        }
    }
    
    /**
     * 清理目标对象的绑定
     */
    public static cleanup(target: any): void {
        this._bindings.forEach((bindings, path) => {
            const newBindings = bindings.filter(binding => binding.target !== target);
            if (newBindings.length === 0) {
                this._bindings.delete(path);
            } else {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gongxh0901/kunpolibrary](https://github.com/gongxh0901/kunpolibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
