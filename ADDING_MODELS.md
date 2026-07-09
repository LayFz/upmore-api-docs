# 添加新模型文档

## 目录结构约定

```
# 指南
guides/model-api/{provider}/{model-name}.mdx
zh/guides/model-api/{provider}/{model-name}.mdx

# API 参考 — 单端点模型（对话）
api-reference/model-api/{provider}/{model-name}.mdx
api-reference/model-api/{provider}/openapi/{model-name}/
  └── openapi.yaml

zh/api-reference/model-api/{provider}/{model-name}.mdx
zh/api-reference/model-api/{provider}/openapi/{model-name}/
  └── openapi.yaml

# API 参考 — 多端点模型（视频/图片生成）
api-reference/model-api/{provider}/{model-name}/
  ├── openapi.yaml
  ├── create.mdx
  └── status.mdx
```

> **为什么 openapi.yaml 要放在 `openapi/` 子目录？**
> Mintlify 规则：同级下不能同时存在 `foo.mdx` 和 `foo/` 目录，否则生产环境 Playground 不渲染。
> 所以 `claude-opus-4-20250514.mdx` 和 `openapi/claude-opus-4-20250514/openapi.yaml` 不会冲突。

**URL 对应关系：**
- 指南：`/guides/model-api/{provider}/{model-name}`
- API 参考（对话）：`/api-reference/model-api/{provider}/{model-name}`
- API 参考（视频）：`/api-reference/model-api/{provider}/{model-name}/create`

---

## 步骤

### 1. 创建 openapi.yaml

在 `api-reference/model-api/{provider}/openapi/{model-name}/` 下新建 `openapi.yaml`。

**对话模型模板**（参考 `api-reference/model-api/anthropic/openapi/claude-3-5-sonnet-20241022/openapi.yaml`）：

```yaml
openapi: 3.1.0
info:
  title: {Model Display Name}
  description: {Provider} {Model} via Upmore OpenAI-compatible API
  version: 1.0.0
servers:
  - url: https://api.upmore.net   # 注意：带 www
security:
  - bearerAuth: []
paths:
  /v1/chat/completions:
    post:
      operationId: createChatCompletion{ModelId}
      summary: Chat Completion
      ...
      properties:
        model:
          type: string
          enum:
            - {model-id}          # 实际调用时传入的模型名
```

**视频生成模型模板**（参考 `api-reference/model-api/bytedance/openapi/seedance/openapi.yaml`）。

同理，在 `zh/api-reference/model-api/{provider}/openapi/{model-name}/` 下创建中文版 `openapi.yaml`，将 `summary` 和 `description` 等字段翻译为中文。

### 2. 创建 API 参考 MDX

**英文**（`api-reference/model-api/{provider}/{model-name}.mdx`）：

```yaml
---
title: '{model-id}'
openapi: '/api-reference/model-api/{provider}/openapi/{model-name}/openapi.yaml POST /v1/chat/completions'
---
```

**中文**（`zh/api-reference/model-api/{provider}/{model-name}.mdx`）：

```yaml
---
title: '{model-id}'
openapi: '/zh/api-reference/model-api/{provider}/openapi/{model-name}/openapi.yaml POST /v1/chat/completions'
---
```

> 中文版 MDX 引用 `zh/` 下的中文 openapi.yaml，不复用英文版。

多端点模型（视频）MDX 放在文件夹内：`{model-name}/create.mdx`、`{model-name}/status.mdx`。

### 3. 创建指南页

**英文**（`guides/model-api/{provider}/{model-name}.mdx`）：

```yaml
---
title: "{Model Name}"
description: "..."
---
```

内容参考 `guides/model-api/anthropic/claude-3-5-sonnet.mdx`，包含：核心能力、快速示例（cURL / Python / Streaming）、参数说明、底部 API Reference Card。

Card 的 href 指向 API 参考页：

```jsx
<Card
  title="API Reference"
  icon="code"
  href="/api-reference/model-api/{provider}/{model-name}"
>
  View the interactive API playground.
</Card>
```

**中文**（`zh/guides/model-api/{provider}/{model-name}.mdx`）同理，内容翻译为中文，Card href 加 `/zh` 前缀。

### 4. 添加提供商图标（新提供商）

从 [LobeHub Icons](https://icons.lobehub.com/) 获取 SVG 图标，然后调整为统一规范格式。

**步骤：**

1. 从 CDN 下载 SVG（provider 名称即图标名，如 `openai`、`anthropic`、`grok`）：

```bash
curl -sL "https://cdn.jsdelivr.net/npm/@lobehub/icons-static-svg/icons/{icon-name}.svg" \
  -o images/providers/{provider}.svg
```

2. 调整 SVG 为统一规范格式，**必须删除以下属性**（LobeHub 默认带的）：
   - `height="..."` 和 `width="..."` — 删除，由外部容器控制大小
   - `style="..."` — 删除
   - `fill-rule="evenodd"` — 删除（除非路径渲染依赖它）

3. **必须添加/保留**以下属性：
   - `fill="currentColor"` — 继承主题颜色
   - `role="img"` — 无障碍标记
   - `viewBox="0 0 24 24"` — 保持 24x24 视口

**最终规范格式**（所有 provider SVG 必须统一为此格式）：

```xml
<svg fill="currentColor" role="img" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <title>{Provider Name}</title>
  <path d="..."/>
</svg>
```

**示例对照**（以 xAI 为例）：

```xml
<!-- 下载的原始格式（❌ 不规范） -->
<svg fill="currentColor" fill-rule="evenodd" height="56" viewBox="0 0 24 24" width="56"
     xmlns="http://www.w3.org/2000/svg" style="flex: 0 0 auto; line-height: 1;">
  <title>Grok</title><path d="..."/></svg>

<!-- 调整后的规范格式（✅ 正确） -->
<svg fill="currentColor" role="img" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <title>xAI</title><path d="..."/></svg>
```

> **注意：** LobeHub 图标名可能和 provider 名不同（如 xAI 的图标名是 `grok`）。在 https://icons.lobehub.com/ 搜索确认。若 LobeHub 没有，再尝试 Simple Icons CDN：`https://cdn.simpleicons.org/{provider}`。

### 5. 更新 docs.json

需要在英文和中文两处各更新。

**Guides tab — 现有提供商新增模型：**

```json
{
  "group": "Anthropic",
  "icon": "/images/providers/anthropic.svg",
  "pages": [
    "guides/model-api/anthropic/claude-3-5-sonnet",
    "guides/model-api/anthropic/{new-model}"
  ]
}
```

**Guides tab — 新提供商：**

```json
{
  "group": "{Provider}",
  "icon": "/images/providers/{provider}.svg",
  "pages": [
    "guides/model-api/{provider}/{model-name}"
  ]
}
```

**API Reference tab — 新增模型：**

```json
{
  "group": "{Provider}",
  "icon": "/images/providers/{provider}.svg",
  "pages": [
    "api-reference/model-api/{provider}/{model-name}"
  ]
}
```

---

## 快速示例：添加 GPT-4o

```bash
# 1. 创建目录
mkdir -p api-reference/model-api/openai/openapi/gpt-4o
mkdir -p zh/api-reference/model-api/openai/openapi/gpt-4o

# 2. 复制并修改 spec
cp api-reference/model-api/openai/openapi/gpt-4/openapi.yaml \
   api-reference/model-api/openai/openapi/gpt-4o/openapi.yaml
cp zh/api-reference/model-api/openai/openapi/gpt-4/openapi.yaml \
   zh/api-reference/model-api/openai/openapi/gpt-4o/openapi.yaml
# 修改 model.enum: ["gpt-4o"]、title、operationId
```

修改 `openapi.yaml` 中：
- `info.title` → `GPT-4o`
- `operationId` → `createChatCompletionGPT4o`
- `model.enum` → `["gpt-4o"]`
- `model.example` → `gpt-4o`

创建 MDX：

```yaml
# api-reference/model-api/openai/gpt-4o.mdx
---
title: 'gpt-4o'
openapi: '/api-reference/model-api/openai/openapi/gpt-4o/openapi.yaml POST /v1/chat/completions'
---
```

在 `docs.json` 的 OpenAI group 追加（英文和中文各一处）：

```json
"pages": [
  "api-reference/model-api/openai/gpt-4",
  "api-reference/model-api/openai/gpt-4o"
]
```

---

## 注意事项

- **base URL 统一用** `https://api.upmore.net`（`upmore.net` 会 301 重定向导致 Authorization header 丢失）
- **openapi.yaml 放在 `openapi/` 子目录**，避免与同名 MDX 文件产生 Mintlify 命名冲突（`foo.mdx` + `foo/` 同级会导致生产环境 playground 不渲染）
- **MDX 里 `openapi` 路径必须以 `/` 开头**（如 `/api-reference/model-api/openai/openapi/gpt-4/openapi.yaml POST /v1/chat/completions`），否则生产环境无法解析路径，playground 空白
- **中文 spec 单独维护**，`summary`、`description`、`example` 中的自然语言翻译为中文，参数名（`model`、`messages` 等）保持英文
- **上线前**确认对应渠道已在控制台开通，否则 API 调用返回 403
