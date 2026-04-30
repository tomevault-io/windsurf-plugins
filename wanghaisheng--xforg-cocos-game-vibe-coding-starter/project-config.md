---
trigger: always_on
description: 欢迎加入 QQ 群：795231926，一起探讨和交流 XForge 开发经验。
---

## 交流与支持
欢迎加入 QQ 群：795231926，一起探讨和交流 XForge 开发经验。

# 一、介绍

框架设计的目的是希望：<br/>
1、**对新手友好，学习曲线平滑。**<br/>
2、更好的 **多人协同** 开发体验，尽可能避免合并冲突。<br/>
3、尽可能统一的 **开发规范** ，避免口头约束，减少因为规范问题带来的CR人力成本。<br/>
4、更小的 **首屏/首页体积** ，优化新用户加载性能。<br/>
5、更小的 **增量更新体积** ，优化老用户加载性能。<br/>
6、通过 **扩展** 的方式复用通用模块，同时平滑框架学习难度。<br/>

> ⚠️大部分的框架目录点击后，都会在属性检查器页面生成它的一些说明文字，可以进行查看。<br/>
> ⚠️使用vscode推荐安装 Code Spell Checker 插件。<br/>

<br>

# 二、脚手架

## 1、 初始化项目

在空文件夹下执行：<br/>
`npx @gamex/cc-cli@latest`

如果npm源无法使用，可以尝试使用淘宝源：<br/>
`npx --registry=https://registry.npmmirror.com @gamex/cc-cli@latest`

## 2、 更新项目框架

在开发过程中，框架可能会发布新版本以修复BUG或更新特性等。<br/>
如果想更新框架，可以在项目根目录下执行快捷指令：<br/>
`npm run upgrade`

其本质还是调用```@gamex/cc-cli```，通过cli方式升级框架，可以几乎不用考虑因升级带来的一些兼容性问题，因为cli内部会处理这些问题。

## 3、 管理扩展包

扩展包中包含通用组件、控件、库等。

想要添加或删除扩展包，可以在项目根目录下，执行快捷指令：<br/>
`npm run package`

然后可以选择**安装**、**卸载**、**更新**、**退出**选项。

<br>

# 三、目录结构 [(视频)](https://www.bilibili.com/video/BV1nP8peeE9B/)
```
├─assets
│   ├─app
│   │  ├─app.ts     // app单例(框架中的所有单例都绑定在app下)
│   │  ├─handle.ts  // 初始化回调
│   │  └─setting.ts // 配置文件
│   │
│   ├─app-appinit   // 首屏
│   │
│   ├─app-builtin
│   │  ├─app-admin  // 框架自动生成的内容
│   │  ├─app-manager// 管理器
│   │  ├─app-control// 控制器
│   │  └─app-model  // 数据
│   │
│   ├─app-bundle    // 框架资源包
│   │  ├─app-view   // 界面
│   │  └─app-sound  // 声音
│   │
│   ├─app-scene     // 初始场景
│   │
│   ├─res-bundle    // Bundle动态资源目录
│   └─res-native    // 静态资源目录
│
├─app               // 框架代码
│
└─pkg               // 扩展包
```
<br/>

# 四、快速上手
### [点击查看wiki](https://gitee.com/cocos2d-zp/cococs-creator-frame-3d/wikis/pages?sort_id=9433202&doc_id=5075526)

<br/>

# 五、详细介绍 [(视频)](https://www.bilibili.com/video/BV138bXeNEUt/)

## 1、 首屏

**游戏的首屏位于assets/app-appinit文件夹内。**

  - res目录用于存放资源。

  - view目录用于存放预制体和脚本。

**AppInit继承自BaseAppInit。**

- 需要注意的是在BaseAppInit中是通过Cocos的生命周期start来进行框架的初始化工作，所以如果在AppInit中重写了start方法，一定要调用super.start或this.startInit。
  ```ts
  /**
    * [可以重写] 默认start调用startInit，可以重写后自定义时机
    */
  protected start(): any { 
    this.startInit(); 
  }
  ```

- 可以通过重写getUserAssetNum、onUserInit、onProgress、onFinish，来实现自定义初始化流程(一般是在onUserInit中异步加载资源，完成的回调中调用一次nextInit来触发下一步)。
  ```ts
  /**
  * [建议重写] 监听进度
  * @param {Number} completed
  * @param {Number} total
  */
  protected onProgress(completed: number, total: number): any { return completed / total; }

  /**
  * [建议重写] 监听用户初始化数据
  * @param {Number} index
  */
  protected onUserInit(index: number): any { return index; }

  /**
  * [建议重写] 获得用户资源总量，这里返回几，就需要用户自行调用几次nextInit
  */
  protected getUserAssetNum(): number { return 0; }

  /**
  * [建议重写] 初始化完成
  */
  protected onFinish() { }
  ```

## 2、管理器

**管理器位于assets/app-builtin/app-manager中。**
> 不需要也不建议手动去创建，可以通过菜单栏App->创建->Manager选项来进行创建。

**管理器继承自BaseManager。**
- 可以通过重写init方法，来初始化一些管理器内部需要用到的资源，完成后需要调用finish表示初始化完成。

- 同时也提供了onInited和onFinished生命周期回调。
  ```ts
  /**
  * [无序] 自身初始化完成, init执行完毕后被调用
  */
  protected onInited() {
  }

  /**
  * [无序] 所有manager初始化完成
  */
  protected onFinished() {
  }

  /**
  * [无序] 初始化manager，在初始化完成后，调用finish方法
  * @param {Function} finish 
  */
  protected init(finish?: Function) {
      finish && finish();
  }
  ```

**管理器一般存放游戏流程控制相关的逻辑。**

- 如登录流程控制、全局状态管理等

**创建好的管理器通过app.manager.xxx来调用。**

- 如app.manager.ui

***框架有内置的管理器，包括：UI、音频、下载、事件、定时器。***

- **UIManager**：通过app.manager.ui访问，用于控制UI显示。

  ```js
  // 显示一个UI
  app.manager.ui.show<UI类>({
      name: '自动提示UI名',
      data: 自动提示onShow方法需要的参数,
      onShow:(){},
      onHide:(result){ //result自动识别为onHide的返回值类型 },
      onError:(){}
  });
  // 关闭一个UI
  app.manager.ui.hide<UI类>({
      name: '自动提示UI名',
      data: 自动提示onHide方法需要的参数,
      onHide:(result){ //result自动识别为onHide的返回值类型 }
  })
  ```

- **SoundManager**：通过app.manager.sound访问，用于控制声音播放。

  ```js
  app.manager.sound.playMusic({name:'自动提示音频名'});

  app.manager.sound.stopMusic();

  const id = app.manager.sound.playEffect({name:'自动提示音频名'});

  app.manager.sound.stopEffect(id);

  app.manager.sound.stopAllEffects();
  ```

- **TimerManager**：通过app.manager.timer访问，用于创建全局定时器。

  ```js
  // 创建或获取定时器
  // app.manager.timer.get返回值的类型是cc.Component的子类
  const timer = app.manager.timer.get(自定义标识);

  timer.schedule // 同cc.Component.prototype.schedule
  timer.scheduleOnce // 同理
  timer.unschedule // 同理
  timer.unscheduleAllCallbacks // 同理

  // 注册每日触发器(基于本地时间，使用cc.Component.prototype.update驱动)
  timer.registerDailyTrigger('8:00:01-9:00',()=>{ 
    console.log('8点01点到9点整之间每帧触发') 
  })
  // 取消每日触发器
  timer.unregisterDailyTrigger

  // 删除定时器
  app.manager.timer.delete(自定义标识);
  // 删除所有定时器
  app.manager.timer.clear();
  ```

- **EventManager**：通过app.manager.event访问，用于创建全局事件。

  ```js
  // 创建或获取事件管理器
  // app.manager.event.get返回值的类型是cc.EventTarget
  const event = app.manager.event.get(自定义标识);

  event.on('event', ()=>{}, target) // 同cc.EventTarget.prototype.on
  event.once('event', ()=>{}, target) // 同理
  event.off('event')或event.off('event', ()=>{}) // 同理
  event.targetOff(target)  // 同理

  // 删除事件管理器
  app.manager.event.delete(自定义标识);
  // 删除所有事件管理器
  app.manager.event.clear();
  ```

- **LoaderManager**：通过app.manager.loader访问，用于加载资源。

  ```js
  app.manager.loader.load({
    bundle: 'xxx', // 不传入bundle，默认为resources
    path: 'xxx/xxxx'
    type: Asset,
    onComplete(asset){
      
    }
  })
  ```

## 3、控制器

**管理器位于assets/app-builtin/app-controller中。**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wanghaisheng) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-12 -->
