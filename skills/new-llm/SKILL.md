---
name: new-llm
description: 为新的 LLM provider 创建接入示例。当用户想添加新的大模型接入、新建 LLM 示例代码时使用。
user-invocable: true
disable-model-invocation: false
argument-hint: <provider-name> [sdk: openai|anthropic] [model: model-name] [base_url: url]
allowed-tools: Bash, Read, Write, Glob
---

# new-llm skill

在 `ai-llm/lm-model/` 下为新的 LLM provider 创建标准接入示例。

## 调用方式

```
/new-llm <provider-name> [sdk: openai|anthropic] [model: model-name] [base_url: url]
```

示例：
- `/new-llm moonshot`
- `/new-llm siliconflow sdk:openai model:Qwen/Qwen2.5-7B-Instruct`

## 执行步骤

### 1. 解析参数

从 `$ARGUMENTS` 中提取：
- `provider`：第一个参数，provider 名称（小写）
- `sdk`：`openai` 或 `anthropic`，默认 `openai`
- `model`：模型名称，默认根据 provider 推断
- `base_url`：API 地址，默认留空让用户填写

### 2. 确认目录

目标路径：`ai-llm/lm-model/<provider>/`

用 Glob 检查该目录是否已存在，若存在则提示用户确认是否覆盖。

### 3. 创建 config.py

```python
BASE_URL = "<base_url 或 占位符>"
API_KEY = "your-api-key-here"
```

### 4. 创建 chat_completions.py

**若 sdk=openai：**

```python
from openai import OpenAI
import config

client = OpenAI(
    base_url=config.BASE_URL,
    api_key=config.API_KEY,
)

response = client.chat.completions.create(
    model="<model>",
    messages=[
        {
            "role": "system",
            "content": "You are a helpful assistant."
        },
        {
            "role": "user",
            "content": "Hello!"
        }
    ],
)

print(response.choices[0].message.content)
print(response.to_json(indent=2))
```

**若 sdk=anthropic：**

```python
from anthropic import Anthropic
import config

client = Anthropic(
    base_url=config.BASE_URL,
    api_key=config.API_KEY,
)

response = client.messages.create(
    model="<model>",
    max_tokens=1024,
    system="You are a helpful assistant.",
    messages=[
        {
            "role": "user",
            "content": "Hello!"
        }
    ],
)

print(response.content[0].text)
print(response.to_json(indent=2))
```

### 5. 完成后输出

告知用户：
- 创建的文件路径
- 需要填写的内容（API_KEY、BASE_URL）
- 如何运行：`python ai-llm/lm-model/<provider>/chat_completions.py`