# agent-skills

我的 Claude Code agent skills 集合，兼容 [vercel-labs/skills](https://github.com/vercel-labs/skills) 标准。

## 安装

### 从 GitHub 安装（推荐）

```shell
# 交互式选择要安装的 skills
npx skills add zhixunjie/agent-skills

# 安装全部 skills
npx skills add zhixunjie/agent-skills --yes
```

安装后会提示选择作用域：
- **Project**：安装到 `.claude/skills/`（仅当前项目生效）
- **Global**：安装到 `~/.claude/skills/`（所有项目生效）

### 本地符号链接（开发用）

```shell
SKILL_NAME=new-llm
mkdir -p ${PATH_PROJECT_LEARN_PYTHON}/ai-llm/ai-agent/agent-skills/.claude/skills
ln -sf ${PATH_PROJECT_LEARN_PYTHON}/ai-llm/ai-agent/agent-skills-my/skills/${SKILL_NAME} \
  ${PATH_PROJECT_LEARN_PYTHON}/ai-llm/ai-agent/agent-skills/.claude/skills/${SKILL_NAME}
```

## Skills 列表

| Skill | 描述 |
|-------|------|
| `new-llm` | 为新的 LLM provider 创建接入示例 |
| `postman-explore` | 探索 Postman collection 结构，按需获取请求详情 |

## 目录结构

```
skills/
├── new-llm/
│   └── SKILL.md
└── postman-explore/
    └── SKILL.md
```

每个 skill 目录包含一个 `SKILL.md`，frontmatter 定义元数据，正文是 Claude Code 执行时的指令。

## 工作原理

`npx skills add` 会：
1. 克隆此 GitHub 仓库到临时目录
2. 扫描所有 `SKILL.md` 文件
3. 展示交互式多选界面
4. 将选中的 skills 安装到对应 agent 目录