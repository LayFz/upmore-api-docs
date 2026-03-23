# 火山方舟豆包多模态适配文档

## 概述

本文档记录火山方舟（VolcEngine）豆包模型的多模态能力适配，涵盖 Chat API 和 Responses API 两种接口格式，支持文本、图片、视频、文档的理解。

### 支持的 API 端点

| 用户端点 | 上游端点 | 说明 |
|---------|---------|------|
| `POST /v1/chat/completions` | `POST /api/v3/chat/completions` | Chat 对话（文本+图片+视频） |
| `POST /v1/responses` | `POST /api/v3/responses` | Responses 对话（文本+图片+视频+文档） |

### 支持的输入方式

所有多模态内容均通过**透传**方式支持，不需要额外的请求体转换。

| 内容类型 | Chat API 格式 | Responses API 格式 |
|---------|--------------|-------------------|
| 图片 URL | `image_url` | `input_image` |
| 图片 Base64 | `image_url` | `input_image` |
| 视频 URL | `video_url` (支持 `fps`) | `input_video` (支持 `fps`) |
| 视频 Base64 | `video_url` | `input_video` |
| 文档 URL | — | `input_file` + `file_url` |
| 文档 Base64 | — | `input_file` + `file_data` + `filename` |

---

## 一、Chat API (`/v1/chat/completions`)

上游实际调用：`https://ark.cn-beijing.volces.com/api/v3/chat/completions`

### 1.1 图片理解（URL 方式）

**请求体：**

```json
{
  "model": "doubao-seed-1-6-251015",
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "image_url",
          "image_url": {
            "url": "https://ark-project.tos-cn-beijing.volces.com/doc_image/ark_demo_img_1.png"
          }
        },
        {
          "type": "text",
          "text": "图片里有什么？"
        }
      ]
    }
  ],
  "max_tokens": 100
}
```

**curl 示例：**

```bash
curl -X POST "http://localhost:3002/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-xxxxx" \
  -d '{
    "model": "doubao-seed-1-6-251015",
    "messages": [
      {
        "role": "user",
        "content": [
          {
            "type": "image_url",
            "image_url": {
              "url": "https://ark-project.tos-cn-beijing.volces.com/doc_image/ark_demo_img_1.png"
            }
          },
          {"type": "text", "text": "图片里有什么？"}
        ]
      }
    ],
    "max_tokens": 100
  }'
```

**返回示例：**

```json
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "这张图片里有3个模型，分别是Doubao-Seed-1.8、DeepSeek-V3.2、GLM-4.7。",
        "reasoning_content": "用户现在需要回答图片里有什么...",
        "role": "assistant"
      }
    }
  ],
  "created": 1774236644,
  "model": "doubao-seed-1-6-251015",
  "object": "chat.completion",
  "usage": {
    "completion_tokens": 87,
    "prompt_tokens": 1355,
    "total_tokens": 1442,
    "completion_tokens_details": {
      "reasoning_tokens": 53
    }
  }
}
```

### 1.2 图片理解（Base64 方式）

**请求体：**

```json
{
  "model": "doubao-seed-1-6-251015",
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "image_url",
          "image_url": {
            "url": "data:image/png;base64,{BASE64_IMAGE}"
          }
        },
        {
          "type": "text",
          "text": "图片里有什么？"
        }
      ]
    }
  ]
}
```

### 1.3 视频理解（URL 方式 + fps 控制）

`fps` 参数控制每秒抽帧数（默认 1，范围 0.2~5）。

**请求体：**

```json
{
  "model": "doubao-seed-1-6-251015",
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "video_url",
          "video_url": {
            "url": "https://ark-project.tos-cn-beijing.volces.com/doc_video/ark_vlm_video_input.mp4",
            "fps": 2
          }
        },
        {
          "type": "text",
          "text": "视频中出现了哪些建筑？"
        }
      ]
    }
  ],
  "max_tokens": 200
}
```

**curl 示例：**

```bash
curl -X POST "http://localhost:3002/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-xxxxx" \
  -d '{
    "model": "doubao-seed-1-6-251015",
    "messages": [
      {
        "role": "user",
        "content": [
          {
            "type": "video_url",
            "video_url": {
              "url": "https://ark-project.tos-cn-beijing.volces.com/doc_video/ark_vlm_video_input.mp4",
              "fps": 2
            }
          },
          {"type": "text", "text": "视频中出现了哪些建筑？"}
        ]
      }
    ],
    "max_tokens": 200
  }'
```

**返回示例：**

```json
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "视频中出现的建筑有：\n1. **大本钟（伊丽莎白塔）**：画面中最显眼的标志性建筑；\n2. **远处的现代高层建筑群**：构成城市天际线的背景；\n3. **具有历史风格的建筑**：画面右侧可见类似英国议会大厦的哥特式风格建筑。",
        "reasoning_content": "...",
        "role": "assistant"
      }
    }
  ],
  "model": "doubao-seed-1-6-251015",
  "object": "chat.completion",
  "usage": {
    "prompt_tokens": 10393,
    "completion_tokens": 132,
    "total_tokens": 10525
  }
}
```

### 1.4 视频理解（Base64 方式）

**请求体：**

```json
{
  "model": "doubao-seed-1-6-251015",
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "video_url",
          "video_url": {
            "url": "data:video/mp4;base64,{BASE64_VIDEO}"
          }
        },
        {
          "type": "text",
          "text": "视频里有什么？"
        }
      ]
    }
  ]
}
```

---

## 二、Responses API (`/v1/responses`)

上游实际调用：`https://ark.cn-beijing.volces.com/api/v3/responses`

### 2.1 文本对话

**请求体：**

```json
{
  "model": "doubao-seed-1-6-251015",
  "input": "你好，用一句话介绍你自己"
}
```

**curl 示例：**

```bash
curl -X POST "http://localhost:3002/v1/responses" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-xxxxx" \
  -d '{
    "model": "doubao-seed-1-6-251015",
    "input": "你好，用一句话介绍你自己"
  }'
```

**返回示例：**

```json
{
  "created_at": 1774236830,
  "id": "resp_021774236829912...",
  "model": "doubao-seed-1-6-251015",
  "object": "response",
  "output": [
    {
      "type": "reasoning",
      "summary": [
        {
          "type": "summary_text",
          "text": "用户让我用一句话介绍自己..."
        }
      ],
      "status": "completed"
    },
    {
      "type": "message",
      "role": "assistant",
      "content": [
        {
          "type": "output_text",
          "text": "我是字节跳动开发的AI助手豆包，能为你提供信息查询、聊天互动、问题解答等多种帮助。"
        }
      ],
      "status": "completed"
    }
  ],
  "status": "completed",
  "usage": {
    "input_tokens": 41,
    "output_tokens": 204,
    "total_tokens": 245,
    "output_tokens_details": {
      "reasoning_tokens": 177
    }
  }
}
```

### 2.2 图片理解（URL 方式）

**请求体：**

```json
{
  "model": "doubao-seed-1-6-251015",
  "input": [
    {
      "role": "user",
      "content": [
        {
          "type": "input_image",
          "image_url": "https://ark-project.tos-cn-beijing.volces.com/doc_image/ark_demo_img_1.png"
        },
        {
          "type": "input_text",
          "text": "图片里有什么？"
        }
      ]
    }
  ]
}
```

**curl 示例：**

```bash
curl -X POST "http://localhost:3002/v1/responses" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-xxxxx" \
  -d '{
    "model": "doubao-seed-1-6-251015",
    "input": [
      {
        "role": "user",
        "content": [
          {
            "type": "input_image",
            "image_url": "https://ark-project.tos-cn-beijing.volces.com/doc_image/ark_demo_img_1.png"
          },
          {"type": "input_text", "text": "图片里有什么？"}
        ]
      }
    ]
  }'
```

### 2.3 图片理解（Base64 方式）

**请求体：**

```json
{
  "model": "doubao-seed-1-6-251015",
  "input": [
    {
      "role": "user",
      "content": [
        {
          "type": "input_image",
          "image_url": "data:image/png;base64,{BASE64_IMAGE}"
        },
        {
          "type": "input_text",
          "text": "图片里有什么？"
        }
      ]
    }
  ]
}
```

### 2.4 视频理解（URL 方式 + fps）

**请求体：**

```json
{
  "model": "doubao-seed-1-6-251015",
  "input": [
    {
      "role": "user",
      "content": [
        {
          "type": "input_video",
          "video_url": "https://ark-project.tos-cn-beijing.volces.com/doc_video/ark_vlm_video_input.mp4",
          "fps": 1
        },
        {
          "type": "input_text",
          "text": "视频里有什么？"
        }
      ]
    }
  ]
}
```

**curl 示例：**

```bash
curl -X POST "http://localhost:3002/v1/responses" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-xxxxx" \
  -d '{
    "model": "doubao-seed-1-6-251015",
    "input": [
      {
        "role": "user",
        "content": [
          {
            "type": "input_video",
            "video_url": "https://ark-project.tos-cn-beijing.volces.com/doc_video/ark_vlm_video_input.mp4",
            "fps": 1
          },
          {"type": "input_text", "text": "视频里有什么？"}
        ]
      }
    ]
  }'
```

**返回示例：**

```json
{
  "created_at": 1774236584,
  "id": "resp_021774236581706...",
  "model": "doubao-seed-1-6-251015",
  "object": "response",
  "output": [
    {
      "type": "reasoning",
      "summary": [
        {
          "type": "summary_text",
          "text": "用户现在需要用一句话概括视频内容..."
        }
      ],
      "status": "completed"
    },
    {
      "type": "message",
      "role": "assistant",
      "content": [
        {
          "type": "output_text",
          "text": "视频展示了黄昏时分，大本钟旁的桥上车辆川流不息的城市景象。"
        }
      ],
      "status": "completed"
    }
  ],
  "status": "completed",
  "usage": {
    "input_tokens": 10393,
    "output_tokens": 82,
    "total_tokens": 10475,
    "output_tokens_details": {
      "reasoning_tokens": 63
    }
  }
}
```

### 2.5 视频理解（Base64 方式）

**请求体：**

```json
{
  "model": "doubao-seed-1-6-251015",
  "input": [
    {
      "role": "user",
      "content": [
        {
          "type": "input_video",
          "video_url": "data:video/mp4;base64,{BASE64_VIDEO}",
          "fps": 1
        }
      ]
    }
  ]
}
```

### 2.6 文档理解（URL 方式）

**请求体：**

```json
{
  "model": "doubao-seed-2-0-lite-260215",
  "input": [
    {
      "role": "user",
      "content": [
        {
          "type": "input_file",
          "file_url": "https://ark-project.tos-cn-beijing.volces.com/doc_pdf/demo.pdf"
        },
        {
          "type": "input_text",
          "text": "按段落给出文档中的文字内容"
        }
      ]
    }
  ]
}
```

### 2.7 文档理解（Base64 方式）

**请求体：**

```json
{
  "model": "doubao-seed-2-0-lite-260215",
  "input": [
    {
      "role": "user",
      "content": [
        {
          "type": "input_file",
          "file_data": "data:application/pdf;base64,{BASE64_PDF}",
          "filename": "demo.pdf"
        },
        {
          "type": "input_text",
          "text": "按段落给出文档中的文字内容"
        }
      ]
    }
  ]
}
```

---

## 三、格式对照说明

### Chat API vs Responses API 内容类型对照

| 内容 | Chat API `type` | Chat API 数据字段 | Responses API `type` | Responses API 数据字段 |
|------|----------------|------------------|---------------------|----------------------|
| 文本 | `text` | `text` | `input_text` | `text` |
| 图片 | `image_url` | `image_url.url` | `input_image` | `image_url` (string) |
| 视频 | `video_url` | `video_url.url` + `video_url.fps` | `input_video` | `video_url` (string) + `fps` |
| 文档 | — | — | `input_file` | `file_url` 或 `file_data`+`filename` |

### fps 参数说明

`fps` 控制视频抽帧频率（每秒抽取的帧数），默认为 1。

| fps 值 | 适用场景 |
|--------|---------|
| 0.2~0.5 | 画面变化不频繁，如静态监控 |
| 1 (默认) | 通用场景 |
| 2~5 | 画面变化剧烈，如动作计数、体育赛事 |

### 文件大小限制

| 传入方式 | 图片限制 | 视频限制 | 文档限制 |
|---------|---------|---------|---------|
| URL | 单张 < 10 MB | < 50 MB | < 50 MB |
| Base64 | 单张 < 10 MB，请求体 < 64 MB | < 50 MB，请求体 < 64 MB | < 50 MB，请求体 < 64 MB |

---

## 四、代码改动

### 修改文件

1. **`relay/channel/volcengine/adaptor.go`**
   - `GetRequestURL` 新增 `RelayModeResponses` 路由 → `/api/v3/responses`
   - `ConvertOpenAIResponsesRequest` 从 `not implemented` 改为透传

2. **`dto/openai_request.go`**
   - 新增 `ContentTypeInputVideo` 常量
   - `ParseContent()` 增加 `input_video` 类型处理
   - `ParseContent()` 修复 `video_url` 支持 string 和 object 两种格式
   - `ParseInput()` 增加 `input_video` 解析
   - `MediaInput` 新增 `VideoUrl` 字段
   - `GetTokenCountMeta()` 增加 `input_video` 文件元数据

### 透传原理

Chat API 的 `Message.Content` 字段类型为 `any`，Responses API 的 `Input` 字段类型为 `json.RawMessage`，序列化时会**完整保留原始 JSON 结构**，包括 `fps` 等非标准字段。因此不需要逐字段适配，所有火山方舟支持的参数都能正确透传到上游。
