---
trigger: always_on
description: ├── components/        # 自定义组件
---

# 微信小程序开发规范

## 项目结构规范

### 标准目录结构
```
services/app/
├── components/        # 自定义组件
│   ├── common/       # 通用组件
│   ├── business/     # 业务组件
│   └── ui/          # UI组件
├── pages/            # 页面文件
├── utils/           # 工具函数
├── api/             # API接口封装
├── styles/          # 公共样式
├── behaviors/       # 组件行为
├── app.js           # 小程序入口文件
├── app.json         # 全局配置
└── app.wxss         # 全局样式
```

### 文件命名规范
```
# ✅ 推荐的文件命名
search-bar/           # 组件目录：小写连字符
user-profile.js       # JS文件：小写连字符
api-client.js         # 工具文件：小写连字符
post-detail.wxml      # 页面文件：小写连字符

# ❌ 避免的命名
SearchBar/            # 避免大驼峰目录名
userProfile.js        # 避免小驼峰文件名
api_client.js         # 避免下划线（除特殊情况）
```

## JavaScript 编码规范

### 1. 基础语法规范
```javascript
// ✅ 推荐的代码风格
const userInfo = {
  openid: '',
  nickname: '',
  avatar: ''
}

// 使用const/let，避免var
const API_BASE_URL = 'https://api.nkuwiki.com'
let searchResults = []

// 函数命名：动词开头，小驼峰
function getUserInfo() {
  return wx.getStorageSync('userInfo')
}

// 异步函数优先使用async/await
async function searchKnowledge(query) {
  try {
    const result = await apiClient.post('/knowledge/search', { query })
    return result.data
  } catch (error) {
    console.error('搜索失败:', error)
    throw error
  }
}
```

### 2. 页面/组件生命周期
```javascript
// 页面生命周期标准模板
Page({
  data: {
    // 页面数据
    query: '',
    searchResults: [],
    loading: false,
    hasMore: true
  },
  
  // 页面加载
  onLoad(options) {
    console.log('页面加载:', options)
    this.initPage(options)
  },
  
  // 页面显示
  onShow() {
    this.refreshUserInfo()
  },
  
  // 页面卸载
  onUnload() {
    this.cleanup()
  },
  
  // 自定义方法
  async initPage(options) {
    const { query } = options
    if (query) {
      this.setData({ query })
      await this.performSearch(query)
    }
  },
  
  async performSearch(query) {
    if (!query.trim()) {
      wx.showToast({ title: '请输入搜索内容', icon: 'none' })
      return
    }
    
    this.setData({ loading: true })
    
    try {
      const results = await searchKnowledge(query)
      this.setData({ 
        searchResults: results,
        loading: false 
      })
    } catch (error) {
      this.setData({ loading: false })
      this.showError('搜索失败，请重试')
    }
  },
  
  // 错误处理
  showError(message) {
    wx.showToast({
      title: message,
      icon: 'none',
      duration: 2000
    })
  },
  
  // 清理资源
  cleanup() {
    // 清理定时器、取消请求等
  }
})
```

### 3. 组件定义规范
```javascript
// 自定义组件标准模板
Component({
  // 组件属性
  properties: {
    placeholder: {
      type: String,
      value: '请输入搜索内容'
    },
    disabled: {
      type: Boolean,
      value: false
    }
  },
  
  // 组件数据
  data: {
    inputValue: '',
    focused: false
  },
  
  // 组件生命周期
  lifetimes: {
    attached() {
      // 组件实例进入页面节点树时执行
      console.log('SearchBar组件已挂载')
    },
    
    detached() {
      // 组件实例被从页面节点树移除时执行
      this.cleanup()
    }
  },
  
  // 页面生命周期
  pageLifetimes: {
    show() {
      // 页面显示时执行
    },
    hide() {
      // 页面隐藏时执行
    }
  },
  
  // 组件方法
  methods: {
    onInput(e) {
      const value = e.detail.value
      this.setData({ inputValue: value })
      
      // 触发自定义事件
      this.triggerEvent('input', { value })
    },
    
    onConfirm(e) {
      const value = e.detail.value.trim()
      if (!value) {
        wx.showToast({ title: '请输入搜索内容', icon: 'none' })
        return
      }
      
      this.triggerEvent('search', { query: value })
    },
    
    onFocus() {
      this.setData({ focused: true })
      this.triggerEvent('focus')
    },
    
    onBlur() {
      this.setData({ focused: false })
      this.triggerEvent('blur')
    },
    
    // 清理方法
    cleanup() {
      // 清理定时器等资源
    }
  }
})
```

## API 调用规范

### 1. API客户端封装
```javascript
// utils/api-client.js
class ApiClient {
  constructor() {
    this.baseURL = 'https://api.nkuwiki.com'
    this.timeout = 10000
  }
  
  // 通用请求方法
  async request(options) {
    const { url, method = 'GET', data, header = {} } = options
    
    // 添加通用请求头
    const defaultHeader = {
      'Content-Type': 'application/json',
      'Authorization': this.getAuthToken()
    }
    
    const requestHeader = { ...defaultHeader, ...header }
    
    return new Promise((resolve, reject) => {
      wx.request({
        url: `${this.baseURL}${url}`,
        method,
        data,
        header: requestHeader,
        timeout: this.timeout,
        success: (res) => {
          this.handleResponse(res, resolve, reject)
        },
        fail: (error) => {
          this.handleError(error, reject)
        }
      })
    })
  }
  
  // 响应处理
  handleResponse(res, resolve, reject) {
    const { statusCode, data } = res
    
    if (statusCode === 200) {
      if (data.code === 200) {
        resolve(data)
      } else {
        reject(new Error(data.message || '请求失败'))
      }
    } else {
      reject(new Error(`HTTP ${statusCode}: ${this.getErrorMessage(statusCode)}`))
    }
  }
  
  // 错误处理
  handleError(error, reject) {
    let message = '网络请求失败'
    
    if (error.errMsg) {
      if (error.errMsg.includes('timeout')) {
        message = '请求超时，请检查网络连接'
      } else if (error.errMsg.includes('fail')) {
        message = '网络连接失败'
      }
    }
    
    reject(new Error(message))
  }
  
  // 获取认证token
  getAuthToken() {
    const userInfo = wx.getStorageSync('userInfo')

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NKU-WIKI/nkuwiki](https://github.com/NKU-WIKI/nkuwiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
