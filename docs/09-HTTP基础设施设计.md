# UniComm HTTP 基础设施设计

## 一、目标

建立统一的 HTTP 请求基础设施，作为后续所有模块（Notify、IM、AI、Plugin、Workspace）的基础。

**禁止**：每个 Feature 自己写 fetch / axios。

---

## 二、当前状态

现有 `src/services/request.ts` 是基础，但需要升级：

```
问题：
- request.ts 直接 import authStore，耦合紧密
- 没有统一的错误处理体系
- 没有 Response 类型定义
- 没有 Retry / Timeout 策略
- Auth interceptor 和 Response interceptor 混在一起
```

---

## 三、目录结构

```
src/
├── core/
│   ├── http/                    # HTTP 基础设施（第一优先级）
│   │   ├── client.ts            # Axios 实例配置
│   │   ├── interceptors/
│   │   │   ├── request.ts       # 请求拦截器（Token 注入）
│   │   │   └── response.ts      # 响应拦截器（错误处理）
│   │   ├── errors/
│   │   │   ├── AppError.ts      # 基础错误类
│   │   │   ├── ApiError.ts      # API 错误
│   │   │   ├── NetworkError.ts  # 网络错误
│   │   │   └── AuthError.ts     # 认证错误
│   │   ├── types/
│   │   │   └── api.ts           # 统一 API Response 类型
│   │   └── index.ts             # 导出
│   │
│   └── runtime/                 # Desktop Runtime（第三优先级）
│       ├── tray.ts               # 托盘管理
│       ├── window.ts             # 窗口生命周期
│       ├── notification.ts        # 桌面通知
│       ├── updater.ts            # 自动更新
│       └── index.ts
│
├── shared/                       # 共享层（可选过渡）
│   └── http/                     # 兼容旧代码
│
├── desktop/
│   ├── adapter/                  # 桌面适配层
│   │   ├── api/                   # Tauri API 抽象
│   │   │   ├── invoke.ts          # invoke 封装
│   │   │   ├── notify.ts          # 通知
│   │   │   └── tray.ts            # 托盘
│   │   └── index.ts
│   │
│   └── runtime/                  # 运行时能力
│       ├── device/
│       ├── user/
│       └── index.ts
│
├── features/
│   ├── auth/
│   │   ├── api/
│   │   │   ├── verifyDesktopApi.ts
│   │   │   └── index.ts
│   │   ├── store/
│   │   │   └── authStore.ts
│   │   ├── types/
│   │   │   └── auth.types.ts
│   │   └── components/
│   │
│   └── memo/
│       └── ...
│
└── services/                      # 旧代码迁移后删除
    └── request.ts
```

---

## 四、HTTP Infrastructure 详细设计

### 4.1 统一 API Response 类型

所有后端响应格式：

```typescript
// src/core/http/types/api.ts

/**
 * 统一 API 响应结构
 * 
 * 后端返回格式：
 * {
 *   code: number;      // 200=成功, 4xx=客户端错误, 5xx=服务端错误
 *   message: string;   // "success" 或错误消息
 *   data: T | null;   // 响应数据，成功时有值
 * }
 */
export interface ApiResponse<T = unknown> {
  code: number;
  message: string;
  data: T | null;
}

/**
 * API 错误响应（后端返回错误格式）
 */
export interface ApiErrorResponse {
  code: number;
  message: string;
  data?: null;
}

/**
 * 分页响应
 */
export interface PagedResponse<T> {
  code: number;
  message: string;
  data: {
    items: T[];
    total: number;
    page: number;
    pageSize: number;
  };
}

/**
 * 判断是否为成功响应
 */
export function isSuccess<T>(response: ApiResponse<T>): boolean {
  return response.code >= 200 && response.code < 300;
}
```

### 4.2 错误体系

```typescript
// src/core/http/errors/AppError.ts

/**
 * 应用错误基类
 * 
 * 所有业务错误都继承此类，确保统一处理。
 */
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number,
    public detail?: string
  ) {
    super(message);
    this.name = 'AppError';
  }
}

// src/core/http/errors/ApiError.ts

import { AppError } from './AppError';

/**
 * API 错误
 * 
 * 当后端返回错误响应时抛出。
 */
export class ApiError extends AppError {
  constructor(
    message: string,
    statusCode: number,
    public response?: unknown
  ) {
    super(message, 'API_ERROR', statusCode);
    this.name = 'ApiError';
  }
}

// src/core/http/errors/NetworkError.ts

import { AppError } from './AppError';

/**
 * 网络错误
 * 
 * 当网络请求失败（超时、无网络等）时抛出。
 */
export class NetworkError extends AppError {
  constructor(message: string, public code?: string) {
    super(message, 'NETWORK_ERROR', 0);
    this.name = 'NetworkError';
  }
}

// src/core/http/errors/AuthError.ts

import { AppError } from './AppError';

/**
 * 认证错误
 * 
 * 当 401/403 时抛出。
 */
export class AuthError extends AppError {
  constructor(message: string, public statusCode: 401 | 403) {
    super(message, 'AUTH_ERROR', statusCode);
    this.name = 'AuthError';
  }
}
```

### 4.3 客户端配置

```typescript
// src/core/http/client.ts

import axios, { type AxiosInstance, type AxiosRequestConfig } from 'axios';
import { ApiResponse } from './types/api';
import { requestInterceptor } from './interceptors/request';
import { responseInterceptor } from './interceptors/response';

/**
 * 默认配置
 */
const DEFAULT_CONFIG: AxiosRequestConfig = {
  baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:28080/api/v1',
  timeout: 15000,
  headers: {
    'Content-Type': 'application/json',
  },
};

/**
 * 创建 HTTP 客户端
 * 
 * @param config - 额外的 Axios 配置
 * @returns 配置好的 Axios 实例
 */
export function createClient(config?: AxiosRequestConfig): AxiosInstance {
  const client = axios.create({ ...DEFAULT_CONFIG, ...config });
  
  // 添加拦截器
  client.interceptors.request.use(requestInterceptor);
  client.interceptors.response.use(
    (response) => response,
    responseInterceptor
  );
  
  return client;
}

/**
 * 默认客户端实例
 */
export const client = createClient();
```

### 4.4 请求拦截器

```typescript
// src/core/http/interceptors/request.ts

import type { InternalAxiosRequestConfig } from 'axios';
import { useAuthStore } from '@/features/auth/store/authStore';

/**
 * 请求拦截器
 * 
 * 自动注入 Authorization 头。
 */
export function requestInterceptor(config: InternalAxiosRequestConfig): InternalAxiosRequestConfig {
  // 从 authStore 获取 token
  const token = useAuthStore.getState().accessToken;
  
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  
  // 添加请求 ID（用于日志追踪）
  config.headers['X-Request-ID'] = crypto.randomUUID();
  
  return config;
}
```

### 4.5 响应拦截器

```typescript
// src/core/http/interceptors/response.ts

import type { AxiosError, AxiosResponse } from 'axios';
import { useAuthStore } from '@/features/auth/store/authStore';
import { ApiError } from '../errors/ApiError';
import { NetworkError } from '../errors/NetworkError';
import { AuthError } from '../errors/AuthError';
import type { ApiResponse } from '../types/api';

/**
 * 响应拦截器
 * 
 * 统一处理：
 * - 2xx: 直接返回 data
 * - 401/403: 清除 Session，抛出 AuthError
 * - 4xx: 抛出 ApiError
 * - 5xx: 抛出 ApiError
 * - 网络错误: 抛出 NetworkError
 */
export function responseInterceptor(error: AxiosError): Promise<never> {
  const { response, code, message } = error;
  
  // 无响应（网络错误、超时）
  if (!response) {
    if (code === 'ECONNABORTED' || code === 'ERR_NETWORK') {
      throw new NetworkError('网络连接失败，请检查网络', code);
    }
    throw new NetworkError(message || '网络错误');
  }
  
  const status = response.status;
  
  // 认证错误 (401/403)
  if (status === 401 || status === 403) {
    // 清除本地 Session
    useAuthStore.getState().clearSession();
    throw new AuthError('认证失败，请重新登录', status);
  }
  
  // 其他 HTTP 错误
  const responseData = response.data as ApiResponse | undefined;
  const errorMessage = responseData?.message || `请求失败 (${status})`;
  
  throw new ApiError(errorMessage, status, response.data);
}

/**
 * 响应成功拦截器（可选）
 * 
 * 用于统一处理成功响应。
 */
export function successInterceptor<T>(response: AxiosResponse<ApiResponse<T>>): T {
  const { code, data, message } = response.data;
  
  // 后端返回业务错误
  if (code !== 200) {
    throw new ApiError(message, code, response.data);
  }
  
  return data as T;
}
```

### 4.6 统一导出

```typescript
// src/core/http/index.ts

export { client, createClient } from './client';
export { ApiResponse, isSuccess, PagedResponse } from './types/api';
export { AppError } from './errors/AppError';
export { ApiError } from './errors/ApiError';
export { NetworkError } from './errors/NetworkError';
export { AuthError } from './errors/AuthError';
```

---

## 五、Feature API 模块

每个 Feature 的 API 调用统一管理：

```
src/features/{feature}/api/
├── index.ts              # 统一导出
├── someApi.ts            # 单个 API 调用
└── types.ts              # API 请求/响应类型
```

示例：

```typescript
// src/features/auth/api/index.ts

export { verifyDesktopApi } from './verifyDesktopApi';

// src/features/auth/api/verifyDesktopApi.ts

import { client } from '@/core/http';
import type { ApiResponse } from '@/core/http/types/api';
import type { DesktopUserInfo } from '../types/auth.types';

export interface VerifyRequest {
  username: string;
  domain?: string;
  computerName: string;
  deviceId: string;
  os: string;
  osVersion?: string;
  appVersion?: string;
}

export interface VerifyResponse {
  username: string;
  employeeNo: string;
  displayName: string;
  departmentName: string;
  permissions: string[];
  accessToken: string;
}

export async function verifyDesktopApi(request: VerifyRequest): Promise<VerifyResponse> {
  const response = await client.post<ApiResponse<VerifyResponse>>('/auth/desktop/verify', request);
  return response.data as VerifyResponse; // 拦截器已处理错误
}
```

---

## 六、Token 自动携带 + 401 处理（核心）

### 6.1 流程

```
[组件调用 API]
    │
    ▼
[request interceptor 自动注入 Bearer token]
    │
    ▼
[后端验证 Token]
    │
    ├─── 成功 ──→ 返回业务数据
    │
    └─── 失败 (401/403) ──→ [response interceptor]
                              │
                              ▼
                         [clearSession()] 清除本地 token
                              │
                              ▼
                         [AuthError 抛出]
                              │
                              ▼
                         [组件 catch AuthError，显示错误页面]
                              │
                              ▼
                         [用户看到"认证失败"页面]
                              │
                              ▼
                         [用户关闭重开 App → 重新 verify]
```

### 6.2 关键点

**没有登录页面**：
- 401/403 后不要跳页面
- 直接显示"认证失败"视图
- 用户关闭重开 App 即可

**自动重试**：
- 401 后不自动重试（需要重新 verify）
- 网络错误可以重试（retryable 属性）

---

## 七、错误处理 UI

在 `App.tsx` 中统一处理：

```tsx
// 错误边界或 AuthErrorView 处理

if (authStatus === 'rejected' || authError) {
  return <AuthErrorView error={authError} onRetry={verifyDesktopUser} />;
}

if (authStatus === 'offline') {
  return <OfflineView onRetry={verifyDesktopUser} />;
}
```

---

## 八、实施步骤

### Phase 1: 核心 HTTP（第一优先级）

1. **创建 `src/core/http/` 目录结构**
2. **实现 `types/api.ts`** - 统一 Response 类型
3. **实现错误类** - AppError, ApiError, NetworkError, AuthError
4. **实现 `client.ts`** - Axios 实例
5. **实现请求拦截器** - Token 注入
6. **实现响应拦截器** - 401/403 处理 + 错误转换
7. **实现 Auth API** - 使用新 client

### Phase 2: 迁移

1. **将 `src/services/request.ts` 迁移到 `src/core/http/`**
2. **更新 authStore** - 使用新的 error 类型
3. **更新 App.tsx** - 使用新的错误处理

### Phase 3: 完善

1. **添加 Retry 逻辑**
2. **添加请求日志**
3. **添加 Timeout 处理**

---

## 九、不做的事情

- ~~不要做 axios 封装成一个巨大函数~~ → 分离拦截器
- ~~不要每个 feature 单独 axios 实例~~ → 使用统一 client
- ~~不要在组件里处理 401~~ → 拦截器统一处理
- ~~不要跳登录页~~ → 显示错误视图即可

---

## 十、验收标准

1. 所有 API 调用都通过 `core/http/client`
2. Token 自动注入（不需要组件手动添加）
3. 401/403 自动清除 Session
4. 所有错误都是 `AppError` 类型（可捕获判断）
5. 没有登录页面跳转