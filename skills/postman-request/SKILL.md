---
name: postman-request
description: 在 Postman collection 中新建或更新请求/文件夹。当用户新增 API 路由、修改接口后想同步到 Postman 时使用。
user-invocable: true
disable-model-invocation: false
argument-hint: "<collection 名称> <method> <path> [folder: 文件夹名]"
allowed-tools: mcp__postman__getWorkspaces, mcp__postman__getCollections, mcp__postman__getCollection, mcp__postman__getCollectionFolder, mcp__postman__getCollectionRequest, mcp__postman__createCollectionFolder, mcp__postman__createCollectionRequest, mcp__postman__updateCollectionRequest, mcp__postman__updateCollectionFolder
---

# postman-request skill

在 Postman collection 中新建或更新请求/文件夹，避免每次都要打开 Postman UI 手动操作。

## 调用方式

```
/postman-request <collection 名称> <method> <path> [folder: 文件夹名]
```

示例：
- `/postman-request shopman POST /shopman_base/app/auth/login folder:auth`
- `/postman-request shopman GET /shopman_base/app/user/info`
- `/postman-request shopman`（不传参数则交互引导）

## 执行步骤

### 1. 定位 Collection

复用 `postman-explore` 的步骤 1-2：
- 调用 `getWorkspaces` 拿所有 workspace
- 并行调用 `getCollections(name=<collection名称>)` 搜索目标 collection
- 未找到或有歧义时，提示用户确认

### 2. 解析参数

从 `$ARGUMENTS` 中提取：
- `collection`：collection 名称
- `method`：HTTP 方法（GET / POST / PUT / DELETE / PATCH），大写
- `path`：完整 URL 路径（如 `/shopman_base/app/auth/login`）
- `folder`：可选，文件夹名称（如 `auth`）；不传则放在 collection 根目录
- 若参数缺失，询问用户补充

### 3. 查看现有结构（轻量）

调用 `getCollection(collectionId)`（不传 model）获取结构图，判断：
- `folder` 是否已存在：查找同名 folder 的 ID
- 目标请求是否已存在：查找同 method + path 的请求

### 4. 确认 Folder

- **folder 已存在**：直接使用其 ID
- **folder 不存在且用户指定了 folder**：调用 `createCollectionFolder` 创建
- **无 folder 参数**：放在 collection 根目录（不传 `folderId`）

### 5. 新建或更新请求

**请求体模板**（根据 method 智能填充）：

```json
{
  "name": "<method> <path 最后一段>",
  "request": {
    "method": "<METHOD>",
    "header": [
      { "key": "Content-Type", "value": "application/json" }
    ],
    "url": {
      "raw": "{{base_url}}<path>",
      "host": ["{{base_url}}"],
      "path": ["<path 各段>"]
    }
  }
}
```

- GET/DELETE：不附 body
- POST/PUT/PATCH：附 `"body": { "mode": "raw", "raw": "{}", "options": { "raw": { "language": "json" } } }`
- 若用户提供了请求体示例，填入 `raw` 字段

**判断新建 vs 更新：**
- 请求不存在：调用 `createCollectionRequest(collectionId, folderId?, body)`
- 请求已存在：告知用户当前内容，确认后调用 `updateCollectionRequest(collectionId, requestId, body)`

### 6. 完成后输出

告知用户：
- 操作类型（新建 / 更新）
- 请求名称、所在 folder、collection
- 如需查看详情可使用 `/postman-explore`

## 工具对照

| 目标 | 工具 | 关键参数 |
|------|------|---------|
| 列出 workspace | `getWorkspaces` | 无 |
| 搜索 collection | `getCollections` | `workspace`, `name` |
| 查看 collection 结构 | `getCollection` | `collectionId`，不传 model |
| 查看 folder 详情 | `getCollectionFolder` | `collectionId`, `folderId` |
| 查看请求详情 | `getCollectionRequest` | `collectionId`, `requestId` |
| 创建 folder | `createCollectionFolder` | `collectionId`, `body.name` |
| 创建请求 | `createCollectionRequest` | `collectionId`, `folderId`（可选）, `body` |
| 更新请求 | `updateCollectionRequest` | `collectionId`, `requestId`, `body` |

## MCP 配置前提

```bash
claude mcp add --transport http postman https://mcp.postman.com/mcp \
  --header "Authorization: Bearer <YOUR_POSTMAN_API_KEY>"
```