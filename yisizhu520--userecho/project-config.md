---
trigger: always_on
description: 安全规范和性能优化最佳实践
---


# 安全与性能规范

## 安全规范

### 前端安全

1. **数据存储安全**:
   ```typescript
   // ✅ 敏感信息使用 sessionStorage 或内存
   import { useAccessToken } from '@vben/access';
   
   // 使用状态管理，不直接存储在 localStorage
   const tokenStore = useAccessToken();
   
   // ❌ 避免
   localStorage.setItem('token', token);  // 不安全
   localStorage.setItem('password', pwd); // 绝对不要存密码
   ```

2. **XSS 防护**:
   ```vue
   <template>
     <!-- ✅ Vue 自动转义，安全 -->
     <div>{{ userInput }}</div>
     
     <!-- ❌ 危险！避免使用 v-html -->
     <div v-html="userInput"></div>
     
     <!-- ✅ 如果必须使用，先消毒 -->
     <div v-html="sanitizedHtml"></div>
   </template>
   
   <script setup lang="ts">
   import DOMPurify from 'dompurify';
   
   const sanitizedHtml = computed(() => 
     DOMPurify.sanitize(userInput.value)
   );
   </script>
   ```

3. **CSRF 防护**:
   ```typescript
   // ✅ 在请求中添加 CSRF token
   import axios from 'axios';
   
   axios.interceptors.request.use(config => {
     const csrfToken = document.querySelector('meta[name="csrf-token"]')?.content;
     if (csrfToken) {
       config.headers['X-CSRF-Token'] = csrfToken;
     }
     return config;
   });
   ```

4. **路由守卫**:
   ```typescript
   // ✅ 验证权限
   import { useAccessStore } from '@vben/access';
   
   router.beforeEach(async (to, from, next) => {
     const accessStore = useAccessStore();
     
     // 检查登录状态
     if (to.meta.requiresAuth && !accessStore.isLoggedIn) {
       next('/login');
       return;
     }
     
     // 检查权限
     if (to.meta.permission && !accessStore.hasPermission(to.meta.permission)) {
       next('/403');
       return;
     }
     
     next();
   });
   ```

### 后端安全

1. **密码安全**:
   ```python
   from pwdlib import PasswordHash
   
   pwd_context = PasswordHash.recommended()
   
   # ✅ 密码加密
   def hash_password(password: str) -> str:
       return pwd_context.hash(password)
   
   # ✅ 密码验证
   def verify_password(plain_password: str, hashed_password: str) -> bool:
       return pwd_context.verify(plain_password, hashed_password)
   
   # ❌ 绝对不要明文存储密码
   # user.password = plain_password  # 危险！
   ```

2. **JWT 安全**:
   ```python
   from datetime import datetime, timedelta
   from jose import jwt
   
   # ✅ 设置合理的过期时间
   def create_access_token(data: dict) -> str:
       to_encode = data.copy()
       expire = datetime.utcnow() + timedelta(minutes=30)  # 短期 token
       to_encode.update({'exp': expire})
       return jwt.encode(to_encode, SECRET_KEY, algorithm='HS256')
   
   def create_refresh_token(data: dict) -> str:
       to_encode = data.copy()
       expire = datetime.utcnow() + timedelta(days=7)  # 长期 refresh token
       to_encode.update({'exp': expire})
       return jwt.encode(to_encode, SECRET_KEY, algorithm='HS256')
   ```

3. **SQL 注入防护**:
   ```python
   from sqlalchemy import select
   
   # ✅ 使用 ORM 参数化查询
   async def get_user_by_email(db: AsyncSession, email: str) -> User | None:
       result = await db.execute(
           select(User).where(User.email == email)  # 自动转义
       )
       return result.scalar_one_or_none()
   
   # ❌ 避免字符串拼接
   # query = f"SELECT * FROM users WHERE email = '{email}'"  # 危险！
   ```

4. **限流保护**:
   ```python
   from fastapi import APIRouter
   from fastapi_limiter.depends import RateLimiter
   
   router = APIRouter()
   
   # ✅ 添加限流
   @router.post('/login', dependencies=[Depends(RateLimiter(times=5, minutes=1))])
   async def login(credentials: LoginSchema):
       """登录接口，每分钟最多 5 次请求"""
       pass
   ```

5. **敏感数据保护**:
   ```python
   from pydantic import BaseModel, Field, field_serializer
   
   class UserResponse(BaseModel):
       id: int
       username: str
       email: str
       
       # ✅ 序列化时隐藏敏感字段
       @field_serializer('email')
       def mask_email(self, email: str) -> str:
           name, domain = email.split('@')
           return f"{name[:2]}***@{domain}"
   ```

6. **日志安全**:
   ```python
   from loguru import logger
   
   # ✅ 不记录敏感信息
   logger.info(f'用户登录: user_id={user.id}')
   
   # ❌ 避免记录敏感数据
   # logger.info(f'用户登录: password={password}')  # 危险！
   # logger.info(f'Token: {token}')  # 危险！
   ```

## 性能优化

### 前端性能

1. **路由懒加载**:
   ```typescript
   // ✅ 路由级代码分割
   const routes = [
     {
       path: '/users',
       component: () => import('./views/Users.vue')
     },
     {
       path: '/settings',
       component: () => import('./views/Settings.vue')
     }
   ];
   ```

2. **组件懒加载**:
   ```vue
   <script setup lang="ts">
   import { defineAsyncComponent } from 'vue';
   
   // ✅ 异步组件
   const HeavyComponent = defineAsyncComponent(() => 
     import('./HeavyComponent.vue')
   );
   </script>
   ```

3. **列表优化**:
   ```vue
   <template>
     <!-- ✅ 使用虚拟滚动处理长列表 -->
     <VirtualList
       :data="items"
       :item-size="50"
       :buffer="10"
     >
       <template #default="{ item }">
         <UserItem :user="item" />
       </template>
     </VirtualList>
     
     <!-- ✅ 使用 key 优化渲染 -->
     <div v-for="item in items" :key="item.id">
       {{ item.name }}
     </div>
     
     <!-- ✅ 使用 v-memo 缓存子树 -->
     <div v-for="item in items" :key="item.id" v-memo="[item.id, item.updated_at]">
       <ExpensiveChild :item="item" />
     </div>
   </template>
   ```

4. **防抖节流**:
   ```typescript
   import { useDebounceFn, useThrottleFn } from '@vueuse/core';
   
   // ✅ 搜索框防抖

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yisizhu520/userecho](https://github.com/yisizhu520/userecho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
