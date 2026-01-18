# API 使用文档

## 目录

- [基础信息](#基础信息)
- [认证流程](#认证流程)
- [完整 API 接口一览](#完整-api-接口一览)
- [业务接口说明](#业务接口说明)
- [错误处理](#错误处理)
- [示例代码](#示例代码)
- [Swagger 与相关文档](#swagger-与相关文档)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [更新日志](#更新日志)
- [技术支持](#技术支持)

---

## 基础信息

### 基础 URL

```
开发环境: http://localhost:5200
API 前缀: /api/v1
```

### 完整 API 地址示例

```
http://localhost:5200/api/v1/auth/register
```

### 响应格式

由 `TransformInterceptor` 统一包装，所有成功响应为：

**成功响应：**
```json
{
  "success": true,
  "data": { }
}
```

- `data`：业务数据。列表类接口通常为 `{ items, total, page, pageSize }`。

**错误响应：**（由 `HttpExceptionFilter` 处理）
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "错误描述"
  }
}
```

- `message` 可能为字符串或字符串数组（如 class-validator 多条校验结果）。
- 开发环境下 `error` 中可能包含 `stack`。

### 认证方式

除注册、登录、刷新令牌外，其余接口均需 JWT，请求头：

```
Authorization: Bearer <your-access-token>
```

---

## 认证流程

### 1. 用户注册

**接口：** `POST /api/v1/auth/register`

**请求体：**
```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecurePass123!",
  "displayName": "John Doe"
}
```

**字段说明：**
- `username`: 用户名，3-50个字符，只能包含字母、数字和下划线
- `email`: 邮箱地址，必须符合邮箱格式
- `password`: 密码，至少8位，必须包含大小写字母和数字
- `displayName`: 显示名称（可选），最多100个字符

**响应示例：**
```json
{
  "success": true,
  "data": {
    "user": {
      "userId": "u_1705123456789_abc123",
      "username": "john_doe",
      "email": "john@example.com",
      "displayName": "John Doe",
      "avatar": null
    },
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

### 2. 用户登录

**接口：** `POST /api/v1/auth/login`

**请求体：**
```json
{
  "emailOrUsername": "john@example.com",
  "password": "SecurePass123!"
}
```

**字段说明：**
- `emailOrUsername`: 邮箱或用户名
- `password`: 密码

**响应示例：**
```json
{
  "success": true,
  "data": {
    "user": {
      "userId": "u_1705123456789_abc123",
      "username": "john_doe",
      "email": "john@example.com",
      "displayName": "John Doe",
      "avatar": null
    },
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

### 3. 刷新令牌

当 Access Token 过期时，使用 Refresh Token 获取新的 Access Token。

**接口：** `POST /api/v1/auth/refresh`

**请求体：**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**响应示例：**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

### 4. 获取当前用户信息

**接口：** `GET /api/v1/auth/me`

**请求头：**
```
Authorization: Bearer <your-access-token>
```

**响应示例：**
```json
{
  "success": true,
  "data": {
    "userId": "u_1705123456789_abc123",
    "username": "john_doe",
    "email": "john@example.com",
    "displayName": "John Doe",
    "avatar": null,
    "bio": null,
    "settings": {},
    "createdAt": "2024-01-15T10:30:00.000Z"
  }
}
```

### 5. 用户登出

**接口：** `POST /api/v1/auth/logout`

**请求头：**
```
Authorization: Bearer <your-access-token>
```

**请求体：**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**响应：**
- 状态码：`204 No Content`
- 无响应体

---

## 完整 API 接口一览

除特别说明外，路径均以 `/api/v1` 为前缀，需认证接口需加 `Authorization: Bearer <accessToken>`。

### 认证 (auth)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| POST | `/auth/register` | 用户注册 | 否 |
| POST | `/auth/login` | 用户登录 | 否 |
| POST | `/auth/refresh` | 刷新令牌 | 否 |
| POST | `/auth/logout` | 用户登出 | 是 |
| GET | `/auth/me` | 获取当前用户 | 是 |

### 工作空间 (workspaces)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| POST | `/workspaces` | 创建工作空间 | 是 |
| GET | `/workspaces` | 工作空间列表（分页：page, pageSize） | 是 |
| GET | `/workspaces/:workspaceId` | 工作空间详情 | 是 |
| PATCH | `/workspaces/:workspaceId` | 更新工作空间 | 是 |
| DELETE | `/workspaces/:workspaceId` | 删除工作空间（软删除） | 是 |
| POST | `/workspaces/:workspaceId/members` | 邀请成员 | 是 |
| GET | `/workspaces/:workspaceId/members` | 成员列表 | 是 |
| PATCH | `/workspaces/:workspaceId/members/:userId` | 更新成员角色 | 是 |
| DELETE | `/workspaces/:workspaceId/members/:userId` | 移除成员 | 是 |

### 文档 (documents)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| POST | `/documents` | 创建文档 | 是 |
| GET | `/documents` | 文档列表（workspaceId, status, visibility, parentId, tags, category, sortBy, sortOrder, page, pageSize） | 是 |
| GET | `/documents/search` | 搜索文档（query 必填；workspaceId, status, tags, 分页） | 是 |
| GET | `/documents/:docId` | 文档详情 | 是 |
| GET | `/documents/:docId/content` | 文档内容/渲染树（?version） | 是 |
| PATCH | `/documents/:docId` | 更新文档元数据 | 是 |
| POST | `/documents/:docId/publish` | 发布文档 | 是 |
| POST | `/documents/:docId/move` | 移动文档 | 是 |
| DELETE | `/documents/:docId` | 删除文档（软删除） | 是 |
| GET | `/documents/:docId/revisions` | 修订历史 | 是 |
| GET | `/documents/:docId/diff` | 版本对比 | 是 |
| POST | `/documents/:docId/revert` | 回滚到指定版本 | 是 |
| POST | `/documents/:docId/snapshots` | 创建快照 | 是 |

### 块 (blocks)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| POST | `/blocks` | 创建块 | 是 |
| PATCH | `/blocks/:blockId/content` | 更新块内容 | 是 |
| POST | `/blocks/:blockId/move` | 移动块 | 是 |
| DELETE | `/blocks/:blockId` | 删除块（软删除） | 是 |
| GET | `/blocks/:blockId/versions` | 块版本历史（分页） | 是 |
| POST | `/blocks/batch` | 批量操作（create/update/delete/move） | 是 |

### 标签 (tags)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| POST | `/tags` | 创建标签 | 是 |
| GET | `/tags` | 标签列表（workspaceId 必填，分页） | 是 |
| GET | `/tags/:tagId` | 标签详情 | 是 |
| GET | `/tags/:tagId/usage` | 标签使用统计 | 是 |
| PATCH | `/tags/:tagId` | 更新标签 | 是 |
| DELETE | `/tags/:tagId` | 删除标签 | 是 |

### 收藏 (favorites)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| POST | `/favorites` | 添加收藏（Body: docId） | 是 |
| GET | `/favorites` | 收藏列表（分页） | 是 |
| DELETE | `/favorites/:docId` | 取消收藏 | 是 |

### 评论 (comments)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| POST | `/comments` | 创建评论（docId, content；可选 blockId, mentions, parentCommentId） | 是 |
| GET | `/comments` | 评论列表（docId 必填，可选 blockId，分页） | 是 |
| GET | `/comments/:commentId` | 评论详情 | 是 |
| PATCH | `/comments/:commentId` | 更新评论（仅本人） | 是 |
| DELETE | `/comments/:commentId` | 删除评论（软删除，仅本人） | 是 |

### 搜索 (search)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| GET | `/search` | 全局搜索（query 必填；workspaceId, type=doc\|block\|all, 分页） | 是 |
| POST | `/search/advanced` | 高级搜索（query, workspaceId, tags, startDate, endDate, createdBy, sortBy, sortOrder, 分页） | 是 |

### 活动日志 (activities)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| GET | `/activities` | 活动列表（workspaceId 必填；userId, action, entityType, startDate, endDate, 分页） | 是 |

### 资产 (assets)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| POST | `/assets/upload` | 上传文件（multipart: workspaceId, file；默认 ≤10MB） | 是 |
| GET | `/assets` | 资产列表（workspaceId 必填，分页） | 是 |
| GET | `/assets/:assetId/file` | 下载/预览文件流 | 是 |
| DELETE | `/assets/:assetId` | 删除资产 | 是 |

### 安全 (security)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| GET | `/security/events` | 安全日志（eventType, userId, ip, startDate, endDate, 分页） | 是 |
| GET | `/security/audit` | 审计日志（userId, action, resourceType, resourceId, startDate, endDate, 分页） | 是 |

### Token 说明

- **Access Token**：访问受保护接口，默认约 24 小时有效。
- **Refresh Token**：刷新 Access Token，默认约 7 天有效。
- 登录/注册返回 `data.accessToken`、`data.refreshToken`；刷新接口返回新的双 Token。

---

## 业务接口说明

### 工作空间

- **创建** `POST /workspaces`  
  Body: `{ name, description?, icon? }`  
  返回：`workspaceId`、`name`、`userRole` 等；创建者自动为 owner。

- **列表** `GET /workspaces`  
  Query: `page`, `pageSize`（默认 1, 20）。  
  返回：`{ items, total, page, pageSize }`，每项含 `userRole`。

- **邀请成员** `POST /workspaces/:workspaceId/members`  
  Body: `{ email, role }`（role: admin | editor | viewer）。需当前用户为 owner 或 admin。

### 文档

- **创建** `POST /documents`  
  Body: `{ workspaceId, title, icon?, cover?, visibility?, parentId?, tags?, category? }`。  
  返回：`docId`、`rootBlockId`、`title` 等；同时创建根块。

- **列表** `GET /documents`  
  Query: `workspaceId`（可选，不传则查有权限的所有空间）、`status`、`visibility`、`parentId`、`tags`、`category`、`sortBy`、`sortOrder`、`page`、`pageSize`。

- **搜索文档** `GET /documents/search`  
  Query: `query`（必填）、`workspaceId?`、`status?`（draft/normal/archived）、`tags?`、`page`、`pageSize`。按文档标题与 searchVector 全文检索。

- **内容** `GET /documents/:docId/content`  
  Query: `version`（可选，默认最新）。返回 `{ docId, docVer, title, tree }`。

- **发布** `POST /documents/:docId/publish`  
  将 `publishedHead` 置为当前 `head`。

- **移动** `POST /documents/:docId/move`  
  Body: `{ parentId?, sortOrder? }`。

### 块

- **创建** `POST /blocks`  
  Body: `{ docId, type, payload, parentId?, sortKey?, indent?, collapsed? }`。  
  返回：`blockId`、`docId`、`type`、`version`、`payload`。

- **更新内容** `PATCH /blocks/:blockId/content`  
  Body: `{ payload, plainText? }`。

- **移动** `POST /blocks/:blockId/move`  
  Body: `{ parentId, sortKey, indent? }`。

- **批量** `POST /blocks/batch`  
  Body: `{ docId, operations }`。`operations` 元素按 `type` 区分：
  - `create`: `{ type: 'create', data: CreateBlockDto }`
  - `update`: `{ type: 'update', blockId, data: UpdateBlockDto }`
  - `delete`: `{ type: 'delete', blockId }`
  - `move`: `{ type: 'move', blockId, parentId, sortKey, indent? }`

### 标签

- **创建** `POST /tags`  
  Body: `{ workspaceId, name, color? }`。同一工作空间下 `name` 不可重复。

- **列表** `GET /tags`  
  Query: `workspaceId`（必填）、`page`、`pageSize`。

### 收藏

- **添加** `POST /favorites`  
  Body: `{ docId }`。同一用户同一文档不可重复收藏。

- **列表** `GET /favorites`  
  Query: `page`、`pageSize`。返回带 `document` 的收藏项，已删除文档会过滤。

### 评论

- **创建** `POST /comments`  
  Body: `{ docId, content, blockId?, mentions?, parentCommentId? }`。

- **列表** `GET /comments`  
  Query: `docId`（必填）、`blockId?`、`page`、`pageSize`。

### 搜索

- **全局** `GET /search`  
  Query: `query`（必填）、`workspaceId?`、`type?`（doc/block/all，默认 all）、`page`、`pageSize`。  
  返回：文档与块的匹配结果（结构见 Swagger）。

- **高级** `POST /search/advanced`  
  Body: `query`、`workspaceId?`、`type?`、`tags?`、`startDate?`、`endDate?`、`createdBy?`、`sortBy?`（rank/updatedAt/createdAt）、`sortOrder?`、`page`、`pageSize`。

### 活动日志

- **列表** `GET /activities`  
  Query: `workspaceId`（必填）、`userId?`、`action?`、`entityType?`、`startDate?`、`endDate?`、`page`、`pageSize`。  
  需具备该工作空间访问权限。

### 资产

- **上传** `POST /assets/upload`  
  `Content-Type: multipart/form-data`，字段：`workspaceId`、`file`。默认限制 10MB，可在配置调整。

- **文件** `GET /assets/:assetId/file`  
  返回文件流，`Content-Disposition: inline` 可预览。

### 安全与审计

- **安全日志** `GET /security/events`  
  Query: `eventType`、`userId`、`ip`、`startDate`、`endDate`、`page`、`pageSize`。通常需管理员权限。

- **审计日志** `GET /security/audit`  
  Query: `userId`、`action`、`resourceType`、`resourceId`、`startDate`、`endDate`、`page`、`pageSize`。

---

## 错误处理

### 错误响应结构

```json
{
  "success": false,
  "error": {
    "code": "string",
    "message": "string | string[]"
  }
}
```

- `code`：来自 `HttpException` 的 `response.code` 或异常名（如 `BadRequestException`）；业务错误常用 `src/common/errors/error-codes.ts` 中 `ErrorCode` 枚举值。
- `message`：`class-validator` 校验失败时可能为字符串数组；其余多为字符串。
- 开发环境下 `error` 可能包含 `stack`。

### 常见错误码与 HTTP 状态

| 含义 | HTTP | 典型 code / 说明 |
|------|------|------------------|
| 参数校验失败 | 400 | `BadRequestException` 或 `VAL_4001`；`message` 常为数组 |
| 未授权 / Token 无效或过期 | 401 | `AUTH_1002`、`AUTH_1003`、`AUTH_1004`；登录失败为 `AUTH_1001` |
| 权限不足 | 403 | `PERM_2001`、`ForbiddenException` |
| 资源不存在 | 404 | `NotFoundException`、`RES_3001` |
| 资源已存在 / 冲突 | 409 | `ConflictException`、`RES_3002` |
| 全局限流 | 429 | `RATE_6001`、`RATE_6002`（@nestjs/throttler） |
| 服务器错误 | 500 | `INTERNAL_ERROR`、`SYS_9001` |

### 错误响应示例

**校验错误（message 可能为数组）：**
```json
{
  "success": false,
  "error": {
    "code": "BadRequestException",
    "message": ["用户名只能包含字母、数字和下划线", "密码至少8位"]
  }
}
```

**认证失败：**
```json
{
  "success": false,
  "error": {
    "code": "AUTH_1001",
    "message": "用户名或密码错误"
  }
}
```

**资源不存在：**
```json
{
  "success": false,
  "error": {
    "code": "NotFoundException",
    "message": "文档不存在"
  }
}
```

---

## 示例代码

### JavaScript / TypeScript (Fetch API)

#### 注册用户

```typescript
async function register() {
  const response = await fetch('http://localhost:5200/api/v1/auth/register', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      username: 'john_doe',
      email: 'john@example.com',
      password: 'SecurePass123!',
      displayName: 'John Doe',
    }),
  });

  const data = await response.json();
  
  if (data.success) {
    // 保存 token
    localStorage.setItem('accessToken', data.data.accessToken);
    localStorage.setItem('refreshToken', data.data.refreshToken);
    console.log('注册成功:', data.data.user);
  } else {
    console.error('注册失败:', data.error);
  }
}
```

#### 用户登录

```typescript
async function login() {
  const response = await fetch('http://localhost:5200/api/v1/auth/login', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      emailOrUsername: 'john@example.com',
      password: 'SecurePass123!',
    }),
  });

  const data = await response.json();
  
  if (data.success) {
    // 保存 token
    localStorage.setItem('accessToken', data.data.accessToken);
    localStorage.setItem('refreshToken', data.data.refreshToken);
    console.log('登录成功:', data.data.user);
  } else {
    console.error('登录失败:', data.error);
  }
}
```

#### 获取当前用户信息

```typescript
async function getCurrentUser() {
  const token = localStorage.getItem('accessToken');
  
  const response = await fetch('http://localhost:5200/api/v1/auth/me', {
    method: 'GET',
    headers: {
      'Authorization': `Bearer ${token}`,
    },
  });

  const data = await response.json();
  
  if (data.success) {
    console.log('当前用户:', data.data);
  } else {
    console.error('获取失败:', data.error);
  }
}
```

#### 刷新 Token

```typescript
async function refreshToken() {
  const refreshToken = localStorage.getItem('refreshToken');
  
  const response = await fetch('http://localhost:5200/api/v1/auth/refresh', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      refreshToken: refreshToken,
    }),
  });

  const data = await response.json();
  
  if (data.success) {
    // 更新 token
    localStorage.setItem('accessToken', data.data.accessToken);
    localStorage.setItem('refreshToken', data.data.refreshToken);
    console.log('Token 刷新成功');
  } else {
    console.error('刷新失败:', data.error);
    // Token 无效，需要重新登录
    localStorage.removeItem('accessToken');
    localStorage.removeItem('refreshToken');
  }
}
```

#### 自动处理 Token 过期

```typescript
async function apiRequest(url: string, options: RequestInit = {}) {
  let token = localStorage.getItem('accessToken');
  const headers: Record<string, string> = {
    'Content-Type': 'application/json',
    ...(token && { 'Authorization': `Bearer ${token}` }),
    ...(options.headers as Record<string, string>),
  };

  let response = await fetch(url, { ...options, headers });

  if (response.status === 401) {
    const refreshToken = localStorage.getItem('refreshToken');
    if (refreshToken) {
      const refreshResponse = await fetch('http://localhost:5200/api/v1/auth/refresh', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ refreshToken }),
      });
      const refreshData = await refreshResponse.json();
      if (refreshData.success) {
        localStorage.setItem('accessToken', refreshData.data.accessToken);
        localStorage.setItem('refreshToken', refreshData.data.refreshToken);
        headers['Authorization'] = `Bearer ${refreshData.data.accessToken}`;
        response = await fetch(url, { ...options, headers });
      } else {
        localStorage.removeItem('accessToken');
        localStorage.removeItem('refreshToken');
        throw new Error('请重新登录');
      }
    }
  }
  return response.json();
}
```

#### 业务接口示例：创建工作空间与文档

```typescript
// 需先登录并取得 accessToken
async function createWorkspaceAndDoc(accessToken: string) {
  const base = 'http://localhost:5200/api/v1';
  const headers = {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${accessToken}`,
  };

  const ws = await fetch(`${base}/workspaces`, {
    method: 'POST',
    headers,
    body: JSON.stringify({ name: '我的空间', description: '示例', icon: '📁' }),
  }).then(r => r.json());
  if (!ws.success) throw new Error(ws.error?.message || '创建工作空间失败');
  const workspaceId = ws.data.workspaceId;

  const doc = await fetch(`${base}/documents`, {
    method: 'POST',
    headers,
    body: JSON.stringify({
      workspaceId,
      title: '第一篇文档',
      visibility: 'workspace',
      tags: ['示例'],
    }),
  }).then(r => r.json());
  if (!doc.success) throw new Error(doc.error?.message || '创建文档失败');
  return { workspaceId, docId: doc.data.docId, rootBlockId: doc.data.rootBlockId };
}
```

### cURL 示例

#### 注册用户

```bash
curl -X POST http://localhost:5200/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "email": "john@example.com",
    "password": "SecurePass123!",
    "displayName": "John Doe"
  }'
```

#### 用户登录

```bash
curl -X POST http://localhost:5200/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "emailOrUsername": "john@example.com",
    "password": "SecurePass123!"
  }'
```

#### 获取当前用户

```bash
curl -X GET http://localhost:5200/api/v1/auth/me \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

#### 刷新 Token

```bash
curl -X POST http://localhost:5200/api/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{"refreshToken": "YOUR_REFRESH_TOKEN"}'
```

#### 创建工作空间（需先登录取得 ACCESS_TOKEN）

```bash
curl -X POST http://localhost:5200/api/v1/workspaces \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -d '{"name": "我的空间", "description": "示例", "icon": "📁"}'
```

### Axios 示例

```typescript
import axios from 'axios';

// 创建 axios 实例
const api = axios.create({
  baseURL: 'http://localhost:5200/api/v1',
  headers: {
    'Content-Type': 'application/json',
  },
});

// 请求拦截器：添加 Token
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// 响应拦截器：处理 Token 过期
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    // 如果是 401 错误且未重试过
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        const refreshToken = localStorage.getItem('refreshToken');
        const response = await axios.post(
          'http://localhost:5200/api/v1/auth/refresh',
          { refreshToken }
        );

        const { accessToken, refreshToken: newRefreshToken } = response.data.data;
        localStorage.setItem('accessToken', accessToken);
        localStorage.setItem('refreshToken', newRefreshToken);

        // 重试原始请求
        originalRequest.headers.Authorization = `Bearer ${accessToken}`;
        return api(originalRequest);
      } catch (refreshError) {
        // 刷新失败，清除 token 并跳转到登录页
        localStorage.removeItem('accessToken');
        localStorage.removeItem('refreshToken');
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);

// 使用示例
async function getCurrentUser() {
  try {
    const response = await api.get('/auth/me');
    console.log('当前用户:', response.data.data);
  } catch (error) {
    console.error('获取失败:', error.response?.data);
  }
}
```

---

## Swagger 与相关文档

### Swagger

启动应用后访问：

```
http://localhost:5200/api/docs
```

可查看全部接口、请求/响应 Schema、并在浏览器中调试。

### 相关文档

- [API 设计文档](./API_DESIGN.md) — 接口与数据结构详细设计
- [用户行为 E2E 测试说明](./E2E_USER_JOURNEY.md) — 串联调用示例与运行方式
- [设置文档](./SETUP.md) — 环境与数据库配置
- [安全机制说明](./SECURITY.md) — 认证、限流、审计等

---

## 最佳实践

### 1. Token 存储

- **Web 应用**: 使用 `localStorage` 或 `sessionStorage` 存储 Token
- **移动应用**: 使用安全的存储方案（如 Keychain/Keystore）
- **不要**将 Token 存储在 Cookie 中（除非设置了 `httpOnly` 和 `secure` 标志）

### 2. Token 刷新策略

- 在 Access Token 过期前（如剩余 5 分钟）自动刷新
- 实现自动重试机制，当收到 401 错误时自动刷新 Token 并重试请求
- 如果 Refresh Token 也过期，引导用户重新登录

### 3. 错误处理

- 始终检查响应中的 `success` 字段
- 根据 `error.code` 进行不同的错误处理
- 向用户显示友好的错误消息

### 4. 安全性

- 生产环境使用 HTTPS
- 不在日志或前端存储中暴露 Token
- 定期更新密码
- 服务端已启用全局限流（@nestjs/throttler，可配置），客户端可对 429 做重试或提示

---

## 常见问题

### Q: Token 过期后怎么办？

A: 用 Refresh Token 调 `POST /auth/refresh` 获取新的 Access Token 与 Refresh Token。

### Q: 如何判断 Token 是否过期？

A: 接口返回 401 且 `error.code` 为 `AUTH_1002`、`AUTH_1003`、`AUTH_1004` 等时，可视为需刷新或重新登录。

### Q: Refresh Token 也会过期吗？

A: 会，默认约 7 天，过期后需重新登录。

### Q: 可以同时有多个有效 Token 吗？

A: 可以，多设备/多会话各自独立；登出时 Body 传 `{ token: accessToken }` 仅销毁当前会话。

### Q: 收到 429 是什么原因？

A: 触发了全局限流（如 60 秒内超过 100 次请求），可稍后重试或联系管理员调整配置。

### Q: 文档、块、工作空间等 ID 的格式？

A: 均为服务端生成的字符串，如 `doc_`、`b_`、`ws_`、`u_` 等前缀，见 `src/common/utils/id-generator.util.ts`。

---

## 更新日志

### 2026-01
- 认证：注册、登录、刷新、登出、当前用户
- 工作空间：CRUD、成员邀请/列表/改角色/移除
- 文档：CRUD、列表、搜索、内容、发布、移动、修订、diff、回滚、快照
- 块：创建、更新内容、移动、删除、版本历史、批量
- 标签：CRUD、列表、使用统计
- 收藏：添加、列表、取消
- 评论：CRUD、列表（按文档/块）
- 搜索：全局搜索、高级搜索
- 活动日志：按工作空间查询
- 资产：上传、列表、文件流、删除
- 安全：安全日志、审计日志
- 全局限流、统一响应格式、HttpExceptionFilter、Swagger

---

## 技术支持

- Swagger: http://localhost:5200/api/docs  
- [API 设计](./API_DESIGN.md) · [E2E 测试说明](./E2E_USER_JOURNEY.md) · [设置](./SETUP.md) · [安装](./INSTALL.md)
