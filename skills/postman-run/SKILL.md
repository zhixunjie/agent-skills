---
name: postman-run
description: 运行 Postman collection 或指定 folder，输出测试结果摘要。当用户想做接口回归测试、冒烟测试时使用。
user-invocable: true
disable-model-invocation: false
argument-hint: "<collection 名称> [folder: 文件夹名] [env: 环境名]"
allowed-tools: mcp__postman__getWorkspaces, mcp__postman__getCollections, mcp__postman__getCollection, mcp__postman__getEnvironments, mcp__postman__getEnvironment, mcp__postman__runCollection
---

# postman-run skill

运行 Postman collection（或其中某个 folder），输出结构化的测试结果摘要。

## 调用方式

```
/postman-run <collection 名称> [folder: 文件夹名] [env: 环境名]
```

示例：
- `/postman-run shopman`（运行整个 collection）
- `/postman-run shopman folder:auth`（只运行 auth folder）
- `/postman-run shopman env:local`（使用 local 环境变量）
- `/postman-run shopman folder:auth env:local`

## 执行步骤

### 1. 定位 Collection

- 调用 `getWorkspaces` 拿所有 workspace
- 并行调用 `getCollections(name=<collection名称>)` 搜索目标 collection
- 未找到或有歧义时提示用户确认

### 2. 解析可选参数

从 `$ARGUMENTS` 中提取：
- `folder`：指定运行某个 folder 的名称（可选）
- `env`：environment 名称（可选）

### 3. 定位 Folder ID（如有）

若用户指定了 `folder`：
- 调用 `getCollection(collectionId)`（不传 model）查看结构
- 从 itemRefs 中找到匹配名称的 folder ID
- 未找到则列出所有 folder 名称，提示用户修正

### 4. 定位 Environment ID（如有）

若用户指定了 `env`：
- 调用 `getEnvironments(workspaceId)` 列出所有 environment
- 找到匹配名称的 environment ID
- 未找到则列出可用环境名称，或询问是否不用环境继续运行

### 5. 运行 Collection

调用 `runCollection`，参数：
- `collectionId`：必填
- `folderId`：若指定了 folder（可选）
- `environmentId`：若指定了 environment（可选）

> 注意：`runCollection` 是异步操作，返回 `runId`，需轮询状态。

### 6. 输出结果摘要

运行完成后，输出结构化摘要：

```
运行结果：shopman / auth
环境：local
─────────────────────────────
总请求数：5
✅ 通过：4
❌ 失败：1

失败详情：
  POST /auth/login — 断言失败：expected status 200, got 401

耗时：1.23s
```

- 若全部通过，输出简洁成功信息
- 若有失败，列出失败请求名称 + 错误原因
- 若运行本身出错（网络、配置），直接输出错误信息

## 工具对照

| 目标 | 工具 | 关键参数 |
|------|------|---------|
| 列出 workspace | `getWorkspaces` | 无 |
| 搜索 collection | `getCollections` | `workspace`, `name` |
| 查看 collection 结构 | `getCollection` | `collectionId`，不传 model |
| 列出环境 | `getEnvironments` | `workspaceId` |
| 查看环境详情 | `getEnvironment` | `environmentId` |
| 运行 collection | `runCollection` | `collectionId`, `folderId`（可选）, `environmentId`（可选） |

## MCP 配置前提

```bash
claude mcp add --transport http postman https://mcp.postman.com/mcp \
  --header "Authorization: Bearer <YOUR_POSTMAN_API_KEY>"
```