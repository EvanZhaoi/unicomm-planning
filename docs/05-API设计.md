# Memo API 设计

## 一、API 设计规范

### 1.1 基础规范

**Base URL:** `/api/v1`

**认证方式:** 桌面端 Windows 用户识别 + 服务端会话令牌

> **认证流程：**
> 1. 应用启动时调用 `/api/v1/auth/desktop/verify` 验证 Windows 用户
> 2. 验证成功后，后端返回 `accessToken`
> 3. 后续 API 请求使用 `Authorization: Bearer {accessToken}`
> 4. 后端使用 Sa-Token 管理桌面端认证后的会话 Token

**统一请求头:**
```
Authorization: Bearer {accessToken}
Content-Type: application/json
```

### 1.2 统一响应格式

**成功响应:**
```json
{
  "code": 200,
  "message": "success",
  "data": { ... }
}
```

**分页响应:**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "list": [...],
    "total": 100,
    "page": 1,
    "size": 20,
    "pages": 5
  }
}
```

**错误响应:**
```json
{
  "code": 400,
  "message": "参数错误",
  "data": null
}
```

### 1.3 状态码说明

| code | 说明 |
|------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未登录 |
| 403 | 无权限 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |

---

## 二、身份认证接口

### 2.1 桌面端身份校验

**接口:** `POST /api/v1/auth/desktop/verify`

**接口说明:** 
- 桌面端启动时调用，验证 Windows 用户
- 后端根据 username 调用公司人员接口获取员工信息
- 如果人员接口返回有效员工并且状态正常，则创建会话 Token
- 如果人员接口返回失败/用户不存在/用户停用，则拒绝访问
- 后端会把人员信息写入 user_snapshot 作为缓存和审计

**请求参数:**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | Windows 用户名 |
| domain | string | 否 | Windows 域信息 |
| computerName | string | 是 | 计算机名称 |
| deviceId | string | 是 | 客户端设备标识 |
| os | string | 是 | 操作系统 |
| osVersion | string | 是 | 操作系统版本 |
| appVersion | string | 是 | 应用版本 |

**请求示例:**
```json
{
  "username": "evan.zhao",
  "domain": "COMPANY",
  "computerName": "CN-SH-001",
  "deviceId": "client-device-id",
  "os": "Windows",
  "osVersion": "Windows 11",
  "appVersion": "0.1.0"
}
```

**响应参数:**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | Windows 用户名 |
| employeeNo | string | 是 | 员工工号 |
| displayName | string | 是 | 显示名称 |
| departmentName | string | 是 | 部门名称 |
| email | string | 否 | 邮箱（如果有） |
| permissions | array | 是 | 权限列表 |
| accessToken | string | 是 | 服务端会话令牌 |

**响应示例:**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "username": "evan.zhao",
    "employeeNo": "E10001",
    "displayName": "Evan Zhao",
    "departmentName": "IT Department",
    "email": "evan.zhao@company.com",
    "permissions": ["memo:read", "memo:write"],
    "accessToken": "server-issued-token"
  }
}
```

**错误响应:**

| code | 说明 |
|------|------|
| 401 | 当前 Windows 用户未在公司人员系统中登记或已停用 |
| 403 | 用户已被禁用 |
| 500 | 认证服务异常 |

**注意事项:**
- 用户身份必须以后端调用公司人员接口返回结果为准
- 前端传来的 username 只是身份线索，不能直接信任
- 后端签发 Token 后，Memo 通过 owner_username 标识创建者，并通过 relatedUsers 控制共享可见范围

---

## 三、Memo 接口

### 2.1 分页查询 Memo

**接口:** `GET /api/v1/memos`

**请求参数:**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | int | 否 | 页码，默认 1 |
| size | int | 否 | 每页条数，默认 20 |
| groupId | long | 否 | 分组筛选 |
| status | string | 否 | 状态筛选: normal/todo/done |
| keyword | string | 否 | 搜索关键词（标题+内容） |
| isShared | boolean | 否 | 是否只查询别人共享给当前用户的 Memo |
| isFavorite | boolean | 否 | 是否收藏筛选 |
| sortBy | string | 否 | 排序字段，默认 updateTime |
| sortOrder | string | 否 | 排序方向: asc/desc，默认 desc |

**请求示例:**
```
GET /api/v1/memos?page=1&size=20&groupId=1&keyword=会议
```

**响应示例:**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "list": [
      {
        "id": 1,
        "ownerUsername": "evan.zhao",
        "title": "会议纪要",
        "content": "# 会议纪要\n...",
        "groupId": 1,
        "groupName": "我的备忘",
        "status": "normal",
        "isTop": false,
        "isFavorite": true,
        "isOwner": true,
        "isShared": false,
        "relatedUsers": [
          {
            "id": 1,
            "username": "leader.wang",
            "employeeNo": "E10004",
            "displayName": "Leader Wang",
            "departmentName": "Management",
            "email": "leader.wang@company.com",
            "permission": "view"
          }
        ],
        "createTime": "2024-01-01 10:00:00",
        "updateTime": "2024-01-01 10:00:00"
      }
    ],
    "total": 100,
    "page": 1,
    "size": 20,
    "pages": 5
  }
}
```

---

### 2.2 获取 Memo 详情

**接口:** `GET /api/v1/memos/{id}`

**路径参数:**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | long | 是 | Memo ID |

**响应示例:**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "ownerUsername": "evan.zhao",
    "title": "会议纪要",
    "content": "# 会议纪要\n...",
    "groupId": 1,
    "groupName": "我的备忘",
    "status": "normal",
    "isTop": false,
    "isFavorite": true,
    "isOwner": true,
    "isShared": false,
    "relatedUsers": [
      {
        "id": 1,
        "username": "leader.wang",
        "employeeNo": "E10004",
        "displayName": "Leader Wang",
        "departmentName": "Management",
        "email": "leader.wang@company.com",
        "permission": "view"
      }
    ],
    "createTime": "2024-01-01 10:00:00",
    "updateTime": "2024-01-01 10:00:00"
  }
}
```

---

### 2.3 创建 Memo

**接口:** `POST /api/v1/memos`

**请求体:**
```json
{
  "title": "新备忘录",
  "content": "# 标题\n内容...",
  "groupId": 1,
  "status": "normal",
  "relatedUsernames": ["leader.wang"]
}
```

**请求参数:**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| title | string | 是 | 标题，最大 200 字符 |
| content | string | 否 | Markdown 内容 |
| groupId | long | 否 | 分组 ID，默认创建默认分组 |
| status | string | 否 | 状态，默认 normal |
| relatedUsernames | array | 否 | 相关人用户名数组，相关人可查看该 Memo |

**响应示例:**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "title": "新备忘录",
    "content": "# 标题\n内容...",
    "groupId": 1,
    "status": "normal",
    "isTop": false,
    "isFavorite": false,
    "createTime": "2024-01-01 10:00:00",
    "updateTime": "2024-01-01 10:00:00"
  }
}
```

---

### 2.4 更新 Memo

**接口:** `PUT /api/v1/memos/{id}`

**路径参数:**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | long | 是 | Memo ID |

**请求体:**
```json
{
  "title": "更新后的标题",
  "content": "更新后的内容...",
  "groupId": 2,
  "status": "done",
  "relatedUsernames": ["leader.wang", "pm.li"]
}
```

**响应示例:**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "title": "更新后的标题",
    "content": "更新后的内容...",
    "groupId": 2,
    "status": "done",
    "isTop": false,
    "isFavorite": false,
    "createTime": "2024-01-01 10:00:00",
    "updateTime": "2024-01-01 11:00:00"
  }
}
```

---

### 2.4.1 更新 Memo 相关人

**接口:** `PUT /api/v1/memos/{id}/related-users`

**说明:** 创建者替换该 Memo 的完整相关人列表。相关人权限支持 `view`（只读）与 `edit`（可编辑标题、正文和状态）。

**路径参数:**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | long | 是 | Memo ID |

**请求体:**
```json
{
  "relatedUsernames": ["leader.wang", "pm.li"]
}
```

**响应:** 返回更新后的 `MemoVO`，包含 `relatedUsers`。

---

### 2.4.2 搜索成员

**接口:** `GET /api/v1/members/search`

**说明:** 前端相关人选择器使用该接口按姓名、用户名、工号或邮箱模糊搜索员工成员。当前阶段数据源为后端临时员工缓存，后续替换为公司人员接口。

**请求参数:**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| keyword | string | 否 | 搜索关键词，支持姓名、用户名、工号、邮箱 |
| limit | int | 否 | 返回数量，默认 10，最大 20 |

**响应示例:**
```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "username": "leader.wang",
      "employeeNo": "E10004",
      "displayName": "Leader Wang",
      "departmentName": "Management",
      "email": "leader.wang@company.com"
    }
  ]
}
```

---

### 2.5 删除 Memo（逻辑删除）

**接口:** `DELETE /api/v1/memos/{id}`

**路径参数:**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | long | 是 | Memo ID |

**响应示例:**
```json
{
  "code": 200,
  "message": "删除成功",
  "data": null
}
```

---

### 2.6 置顶/取消置顶

**接口:** `PATCH /api/v1/memos/{id}/top`

**路径参数:**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | long | 是 | Memo ID |

**请求体:**
```json
{
  "isTop": true
}
```

**响应示例:**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "isTop": true
  }
}
```

---

### 2.7 收藏/取消收藏

**接口:** `PATCH /api/v1/memos/{id}/favorite`

**请求体:**
```json
{
  "isFavorite": true
}
```

**响应示例:**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "isFavorite": true
  }
}
```

---

## 三、Memo Group 接口

### 3.1 获取分组列表

**接口:** `GET /api/v1/memo-groups`

**响应示例:**
```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "id": 1,
      "name": "我的备忘",
      "color": "#6B7280",
      "icon": "📁",
      "sortOrder": 0,
      "memoCount": 20
    },
    {
      "id": 2,
      "name": "工作笔记",
      "color": "#2563EB",
      "icon": "💼",
      "sortOrder": 1,
      "memoCount": 8
    }
  ]
}
```

---

### 3.2 创建分组

**接口:** `POST /api/v1/memo-groups`

**请求体:**
```json
{
  "name": "新分组",
  "color": "#2563EB",
  "icon": "📁"
}
```

**响应示例:**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 3,
    "name": "新分组",
    "color": "#2563EB",
    "icon": "📁",
    "sortOrder": 10
  }
}
```

---

### 3.3 更新分组

**接口:** `PUT /api/v1/memo-groups/{id}`

**请求体:**
```json
{
  "name": "更新后的分组名",
  "color": "#DC2626",
  "icon": "🔥",
  "sortOrder": 5
}
```

---

### 3.4 删除分组

**接口:** `DELETE /api/v1/memo-groups/{id}`

**说明:** 删除分组后，该分组下的 Memo 自动移至"我的备忘"

**响应示例:**
```json
{
  "code": 200,
  "message": "删除成功",
  "data": null
}
```

---

### 3.5 分组排序

**接口:** `PATCH /api/v1/memo-groups/sort`

**请求体:**
```json
{
  "groupIds": [2, 1, 3, 4]
}
```

---

## 五、DTO 定义

### 5.1 创建 DTO

```java
public class MemoCreateDTO {
    // 标题允许为空；空标题保存为空字符串，由前端显示"请输入标题"占位符
    @Size(max = 200, message = "标题最多200字符")
    private String title;
    
    private String content;
    
    private Long groupId;
    
    @Pattern(regexp = "^(normal|todo|done)$", message = "状态值不合法")
    private String status = "normal";
    
}
```

### 5.2 更新 DTO

```java
public class MemoUpdateDTO {
    @Size(max = 200, message = "标题最多200字符")
    private String title;
    
    private String content;
    
    private Long groupId;
    
    @Pattern(regexp = "^(normal|todo|done)$", message = "状态值不合法")
    private String status;
    
}
```

### 5.3 查询 DTO

```java
public class MemoQueryDTO {
    private Integer page = 1;
    private Integer size = 20;
    private Long groupId;
    private String status;
    private String keyword;
    private Boolean isFavorite;
    private String sortBy = "updateTime";
    private String sortOrder = "desc";
}
```

### 5.4 VO 定义

```java
public class MemoVO {
    private Long id;
    private String title;
    private String content;
    private Long groupId;
    private String groupName;
    private String status;
    private Boolean isTop;
    private Boolean isFavorite;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}
```

---

## 六、接口汇总

| 接口 | Method | Path | 说明 |
|------|--------|------|------|
| 分页查询 Memo | GET | /api/v1/memos | |
| 获取 Memo 详情 | GET | /api/v1/memos/{id} | |
| 创建 Memo | POST | /api/v1/memos | |
| 更新 Memo | PUT | /api/v1/memos/{id} | |
| 更新 Memo 相关人 | PUT | /api/v1/memos/{id}/related-users | 创建者维护相关人和 view/edit 权限 |
| 搜索成员 | GET | /api/v1/members/search | 按姓名、用户名、工号、邮箱模糊搜索 |
| 删除 Memo | DELETE | /api/v1/memos/{id} | |
| 置顶/取消置顶 | PATCH | /api/v1/memos/{id}/top | |
| 收藏/取消收藏 | PATCH | /api/v1/memos/{id}/favorite | |
| 获取分组列表 | GET | /api/v1/memo-groups | |
| 创建分组 | POST | /api/v1/memo-groups | |
| 更新分组 | PUT | /api/v1/memo-groups/{id} | |
| 删除分组 | DELETE | /api/v1/memo-groups/{id} | |
| 分组排序 | PATCH | /api/v1/memo-groups/sort | |
