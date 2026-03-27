---
name: postman-explore
description: 探索 Postman collection 结构，按需获取请求详情。当用户想查看、分析 Postman collection 内容时使用。
user-invocable: true
disable-model-invocation: false
argument-hint: [collection 名称]
allowed-tools: mcp__postman__getAuthenticatedUser, mcp__postman__getWorkspaces, mcp__postman__getCollections, mcp__postman__getCollection, mcp__postman__getCollectionFolder, mcp__postman__getCollectionRequest, mcp__postman__getEnvironments
---

# postman-explore skill

探索 Postman collection 结构，避免一次性拉取全量数据导致响应过大。

## 调用方式

```
/postman-explore [collection 名称]
```

示例：
- `/postman-explore shopman`
- `/postman-explore`（不传名称则列出所有 collection）

## 执行步骤

### 1. 获取当前用户 + 所有 Workspace

并行执行：
- 调用 `getAuthenticatedUser` 获取当前用户 ID（用于后续过滤"我的" workspace）
- 调用 `getWorkspaces` 获取所有 workspace（ID + 名称）

### 2. 并行搜索 Collection

对每个 workspace **并行**调用 `getCollections`：
- 若用户传了名称，加上 `name` 参数过滤
- 若未传名称，列出所有 collection 供用户选择

> 注意：`getCollections` 必须传 `workspace` ID，无法全局搜索，必须遍历所有 workspace。

**结果处理：**
- **未找到**：提示用户检查名称拼写，列出所有可用 collection 名称
- **找到多个同名**：列出所有匹配项（含所属 workspace），让用户选择
- **找到唯一**：直接进入下一步

### 3. 获取 Collection 结构（轻量模式）

找到目标 collection 后，调用 `getCollection`：
- **不传 `model` 参数**（默认返回轻量结构图：metadata + 递归 itemRefs）
- 绝对不要用 `model=full`，会返回完整 JSON（可能 60KB+），超出工具结果显示限制

collectionId 格式必须为 `{ownerId}-{uuid}`，从 `getCollections` 返回的 `uid` 字段取。

### 4. 展示结构，按需深入

用缩进树状格式展示 folder / request 列表，例如：

```
📁 auth
  POST  /auth/login
  POST  /auth/register
📁 user
  GET   /user/info
```

询问用户是否需要查看某个请求的详情。如需查看：
- `getCollectionRequest`：查看单个请求的 method、URL、headers、body 等
- `getCollectionFolder`：查看 folder 下的子项目

如用户询问"有哪些环境"，调用 `getEnvironments(workspaceId)` 列出可用 environment。

## 工具对照

| 目标 | 工具 | 关键参数 |
|------|------|---------|
| 获取当前用户 | `getAuthenticatedUser` | 无 |
| 列出所有 workspace | `getWorkspaces` | 无必填 |
| 搜索 collection | `getCollections` | `workspace`（必填）, `name`（可选） |
| 查看 collection 结构 | `getCollection` | `collectionId`，不传 model |
| 查看单个请求详情 | `getCollectionRequest` | `collectionId`, `requestId` |
| 查看 folder 详情 | `getCollectionFolder` | `collectionId`, `folderId` |
| 查看环境列表 | `getEnvironments` | `workspaceId` |

## MCP 配置前提

使用本 skill 前需先配置 Postman MCP：

```bash
claude mcp add --transport http postman https://mcp.postman.com/mcp \
  --header "Authorization: Bearer <YOUR_POSTMAN_API_KEY>"
```

API Key 在 Postman → 右上角头像 → Settings → API keys 中生成。
