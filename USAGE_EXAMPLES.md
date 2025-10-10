# 使用示例

本文档展示如何在不同编程语言和工具中使用 `X-Factory-API-Key` 请求头来访问 droid2api 服务。

## 认证方式说明

droid2api 支持两种主要的认证方式：

1. **客户端提供密钥**（推荐）- 通过 `X-Factory-API-Key` 请求头提供
2. **服务器共享密钥** - 在服务器环境变量中配置 `FACTORY_API_KEY`

### 优先级

```
客户端 X-Factory-API-Key > 服务器 FACTORY_API_KEY > refresh_token > 客户端 authorization
```

## 示例代码

### cURL

```bash
# 基本请求
curl http://localhost:3000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "X-Factory-API-Key: your_api_key_here" \
  -d '{
    "model": "claude-opus-4-1-20250805",
    "messages": [
      {"role": "user", "content": "Hello, how are you?"}
    ]
  }'

# 流式响应
curl http://localhost:3000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "X-Factory-API-Key: your_api_key_here" \
  -d '{
    "model": "claude-opus-4-1-20250805",
    "messages": [
      {"role": "user", "content": "写一首诗"}
    ],
    "stream": true
  }'
```

### Python

#### 使用 requests 库

```python
import requests

api_key = "your_api_key_here"
base_url = "http://localhost:3000"

headers = {
    "Content-Type": "application/json",
    "X-Factory-API-Key": api_key
}

data = {
    "model": "claude-opus-4-1-20250805",
    "messages": [
        {"role": "user", "content": "你好，请介绍一下自己"}
    ],
    "stream": False
}

response = requests.post(
    f"{base_url}/v1/chat/completions",
    headers=headers,
    json=data
)

print(response.json())
```

#### 使用 OpenAI SDK

```python
from openai import OpenAI

# 配置客户端
client = OpenAI(
    base_url="http://localhost:3000/v1",
    api_key="your_api_key_here",  # 会自动转换为 Authorization 头
    default_headers={
        "X-Factory-API-Key": "your_api_key_here"  # 直接使用 Factory API Key
    }
)

# 发送请求
response = client.chat.completions.create(
    model="claude-opus-4-1-20250805",
    messages=[
        {"role": "user", "content": "你好，世界"}
    ]
)

print(response.choices[0].message.content)
```

#### 流式响应

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:3000/v1",
    api_key="your_api_key_here",
    default_headers={
        "X-Factory-API-Key": "your_api_key_here"
    }
)

# 流式响应
stream = client.chat.completions.create(
    model="claude-opus-4-1-20250805",
    messages=[
        {"role": "user", "content": "讲一个故事"}
    ],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### JavaScript / TypeScript

#### 使用 fetch

```javascript
const apiKey = "your_api_key_here";
const baseUrl = "http://localhost:3000";

async function chat(message) {
  const response = await fetch(`${baseUrl}/v1/chat/completions`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Factory-API-Key": apiKey
    },
    body: JSON.stringify({
      model: "claude-opus-4-1-20250805",
      messages: [
        { role: "user", content: message }
      ]
    })
  });

  const data = await response.json();
  return data.choices[0].message.content;
}

// 使用
chat("你好，请介绍一下自己").then(console.log);
```

#### 使用 OpenAI SDK

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "http://localhost:3000/v1",
  apiKey: "your_api_key_here",
  defaultHeaders: {
    "X-Factory-API-Key": "your_api_key_here"
  }
});

async function chat(message: string) {
  const response = await client.chat.completions.create({
    model: "claude-opus-4-1-20250805",
    messages: [
      { role: "user", content: message }
    ]
  });

  return response.choices[0].message.content;
}

// 使用
chat("你好，世界").then(console.log);
```

#### 流式响应

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "http://localhost:3000/v1",
  apiKey: "your_api_key_here",
  defaultHeaders: {
    "X-Factory-API-Key": "your_api_key_here"
  }
});

async function streamChat(message: string) {
  const stream = await client.chat.completions.create({
    model: "claude-opus-4-1-20250805",
    messages: [
      { role: "user", content: message }
    ],
    stream: true
  });

  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content;
    if (content) {
      process.stdout.write(content);
    }
  }
}

// 使用
streamChat("讲一个故事");
```

### Go

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
)

type ChatRequest struct {
    Model    string    `json:"model"`
    Messages []Message `json:"messages"`
    Stream   bool      `json:"stream"`
}

type Message struct {
    Role    string `json:"role"`
    Content string `json:"content"`
}

type ChatResponse struct {
    Choices []struct {
        Message Message `json:"message"`
    } `json:"choices"`
}

func main() {
    apiKey := "your_api_key_here"
    baseURL := "http://localhost:3000"

    reqBody := ChatRequest{
        Model: "claude-opus-4-1-20250805",
        Messages: []Message{
            {Role: "user", Content: "你好，请介绍一下自己"},
        },
        Stream: false,
    }

    jsonData, _ := json.Marshal(reqBody)
    
    req, _ := http.NewRequest("POST", baseURL+"/v1/chat/completions", bytes.NewBuffer(jsonData))
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("X-Factory-API-Key", apiKey)

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        panic(err)
    }
    defer resp.Body.Close()

    body, _ := io.ReadAll(resp.Body)
    
    var chatResp ChatResponse
    json.Unmarshal(body, &chatResp)
    
    fmt.Println(chatResp.Choices[0].Message.Content)
}
```

### Rust

```rust
use reqwest;
use serde::{Deserialize, Serialize};
use serde_json::json;

#[derive(Serialize, Deserialize, Debug)]
struct Message {
    role: String,
    content: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct ChatRequest {
    model: String,
    messages: Vec<Message>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Choice {
    message: Message,
}

#[derive(Serialize, Deserialize, Debug)]
struct ChatResponse {
    choices: Vec<Choice>,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let api_key = "your_api_key_here";
    let base_url = "http://localhost:3000";

    let client = reqwest::Client::new();
    
    let request_body = ChatRequest {
        model: "claude-opus-4-1-20250805".to_string(),
        messages: vec![
            Message {
                role: "user".to_string(),
                content: "你好，请介绍一下自己".to_string(),
            }
        ],
    };

    let response = client
        .post(format!("{}/v1/chat/completions", base_url))
        .header("Content-Type", "application/json")
        .header("X-Factory-API-Key", api_key)
        .json(&request_body)
        .send()
        .await?;

    let chat_response: ChatResponse = response.json().await?;
    
    println!("{}", chat_response.choices[0].message.content);

    Ok(())
}
```

## 环境变量配置示例

如果你希望在服务器端配置共享密钥，而不是在每个请求中传递：

### Linux/macOS

```bash
# 临时设置（当前会话）
export FACTORY_API_KEY="your_api_key_here"

# 永久设置（添加到 ~/.bashrc 或 ~/.zshrc）
echo 'export FACTORY_API_KEY="your_api_key_here"' >> ~/.bashrc
source ~/.bashrc
```

### Windows

```powershell
# PowerShell 临时设置
$env:FACTORY_API_KEY="your_api_key_here"

# PowerShell 永久设置（当前用户）
[Environment]::SetEnvironmentVariable("FACTORY_API_KEY", "your_api_key_here", "User")

# CMD 临时设置
set FACTORY_API_KEY=your_api_key_here
```

### Docker

```yaml
# docker-compose.yml
version: '3.8'
services:
  droid2api:
    image: droid2api
    ports:
      - "3000:3000"
    environment:
      - FACTORY_API_KEY=your_api_key_here
```

## 多用户场景示例

在多用户 SaaS 应用中，每个用户可以使用自己的 API 密钥：

### Python Flask 示例

```python
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

@app.route('/api/chat', methods=['POST'])
def chat():
    # 从用户请求中获取用户的 API 密钥
    user_api_key = request.headers.get('Authorization')
    
    if not user_api_key:
        return jsonify({"error": "Missing API key"}), 401
    
    # 转发请求到 droid2api，使用用户的密钥
    response = requests.post(
        "http://localhost:3000/v1/chat/completions",
        headers={
            "Content-Type": "application/json",
            "X-Factory-API-Key": user_api_key  # 使用用户自己的密钥
        },
        json=request.json
    )
    
    return response.json(), response.status_code

if __name__ == '__main__':
    app.run(port=5000)
```

### Node.js Express 示例

```javascript
const express = require('express');
const axios = require('axios');

const app = express();
app.use(express.json());

app.post('/api/chat', async (req, res) => {
    // 从用户请求中获取用户的 API 密钥
    const userApiKey = req.headers['authorization'];
    
    if (!userApiKey) {
        return res.status(401).json({ error: 'Missing API key' });
    }
    
    try {
        // 转发请求到 droid2api，使用用户的密钥
        const response = await axios.post(
            'http://localhost:3000/v1/chat/completions',
            req.body,
            {
                headers: {
                    'Content-Type': 'application/json',
                    'X-Factory-API-Key': userApiKey  // 使用用户自己的密钥
                }
            }
        );
        
        res.json(response.data);
    } catch (error) {
        res.status(error.response?.status || 500).json({
            error: error.response?.data || 'Internal server error'
        });
    }
});

app.listen(5000, () => {
    console.log('Proxy server running on port 5000');
});
```

## 错误处理

当 API 密钥无效或缺失时，droid2api 会返回以下错误：

```json
{
  "error": "Authentication required",
  "message": "Please provide X-Factory-API-Key header with your API key, or configure server authentication."
}
```

**HTTP 状态码**：`401 Unauthorized`

## 常见问题

### Q: 我可以同时使用客户端密钥和服务器密钥吗？

A: 可以。当同时存在时，客户端提供的 `X-Factory-API-Key` 具有最高优先级，会覆盖服务器配置的 `FACTORY_API_KEY`。

### Q: X-API-Key 和 X-Factory-API-Key 有什么区别？

A: 两者都可以使用，功能完全相同。`X-Factory-API-Key` 是推荐的标准头名称，`X-API-Key` 是为了兼容性而保留的别名。

### Q: 如果不提供任何密钥会怎样？

A: 如果服务器没有配置 `FACTORY_API_KEY` 或 `DROID_REFRESH_KEY`，且客户端也没有提供密钥，系统会返回 401 错误，要求提供认证。

### Q: 我的密钥会被记录到日志中吗？

A: droid2api 只会记录"正在使用客户端提供的密钥"这样的信息，不会记录密钥的具体内容，保护你的密钥安全。

## 最佳实践

1. **生产环境**：推荐使用客户端提供密钥的方式，每个用户管理自己的密钥
2. **开发环境**：可以在服务器端配置共享密钥，简化开发流程
3. **密钥管理**：使用环境变量或密钥管理服务存储密钥，不要硬编码在代码中
4. **错误处理**：始终处理 401 错误，提示用户检查 API 密钥
5. **日志记录**：在应用层记录 API 使用情况，便于追踪和调试

## 更多信息

- [README.md](./README.md) - 完整的功能说明和配置指南
- [DOCKER_DEPLOY.md](./DOCKER_DEPLOY.md) - Docker 部署指南

