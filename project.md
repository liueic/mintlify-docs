# AI Gateway 项目详细文档

## 项目概述

AI Gateway 是一个基于适配器模式的多Agent统一接口平台，旨在为前端提供标准化的AI服务接口。该项目通过抽象基类和统一的数据模型，实现了对多个AI平台（FastGPT、Dify、Coze等）的兼容性，让开发者可以轻松切换不同的AI后端服务。

## 项目特点

### 1. 适配器模式设计
- **统一接口**：所有Agent都继承自`BaseAgent`基类，提供标准化的接口
- **灵活切换**：前端只需修改请求头中的`agent`参数即可切换不同的AI平台
- **向后兼容**：保持与各平台原有API的兼容性

### 2. 标准化响应格式
- **UnifiedChatResponse**：统一的响应模型，支持标准模式和详细模式
- **FastGPT兼容**：响应格式完全兼容FastGPT API，便于前端集成
- **流式支持**：支持SSE（Server-Sent Events）流式响应

### 3. 文件上传支持
- **多格式支持**：支持图片、文档等多种文件类型
- **统一接口**：所有Agent共享相同的文件上传接口
- **S3/MinIO集成**：支持对象存储服务

### 4. 动态加载机制
- **自动发现**：系统自动发现和加载agent目录下的所有Agent模块
- **环境变量配置**：通过环境变量控制Agent的启用和配置
- **注册表管理**：使用注册表模式管理所有可用的Agent实例

## 支持的Agent平台

### 1. FastGPT Agent

**特点：**
- 完整的RAG（检索增强生成）支持
- 支持知识库检索和引用
- 支持多模态输入（文本+图片+文档）
- 支持对话上下文管理

**环境变量配置：**
```bash
FASTGPT_BASE_URL=https://your-fastgpt-instance.com
FASTGPT_API_KEY=your-api-key
FASTGPT_APP_ID=your-app-id
```

**请求示例：**
```bash
curl -X POST 'http://localhost:3000/api/v1/chat' \
  -H 'Content-Type: application/json' \
  -H 'agent: fastgpt' \
  -d '{
    "query": "你好，请介绍一下胰腺癌的基本知识",
    "user": "test_user",
    "stream": false,
    "detail": false
  }'
```

**文件上传支持：**
```bash
curl -X POST 'http://localhost:3000/api/v1/upload' \
  -H 'agent: fastgpt' \
  -F 'file=@document.pdf' \
  -F 'created_by=test_user'
```

### 2. Dify Agent

**特点：**
- 支持工作流和对话管理
- 支持变量替换和上下文管理
- 支持流式和非流式响应
- 支持文件上传和处理

**环境变量配置：**
```bash
DIFY_BASE_URL=https://your-dify-instance.com
DIFY_API_KEY=your-api-key
```

**请求示例：**
```bash
curl -X POST 'http://localhost:3000/api/v1/chat' \
  -H 'Content-Type: application/json' \
  -H 'agent: dify' \
  -d '{
    "query": "请分析这个文档",
    "user": "test_user",
    "conversation_id": "conv_123",
    "stream": false
  }'
```

### 3. Coze Agent

**特点：**
- 基于Coze平台的Bot服务
- 支持国内和海外版本
- 支持流式和非流式对话
- 支持多轮对话上下文

**环境变量配置：**
```bash
COZE_API_TOKEN=your-api-token
COZE_BOT_ID=your-bot-id
COZE_BASE_URL=https://api.coze.cn  # 可选，默认为国内版
```

**请求示例：**
```bash
curl -X POST 'http://localhost:3000/api/v1/chat' \
  -H 'Content-Type: application/json' \
  -H 'agent: coze' \
  -d '{
    "query": "你好，请介绍一下自己",
    "user": "test_user",
    "conversation_id": "conv_123",
    "stream": false
  }'
```

## API接口文档

### 1. 聊天接口

**端点：** `POST /api/v1/chat`

**请求头：**
- `Content-Type: application/json`
- `agent: {agent_name}` - 指定使用的Agent（必填）

**请求参数：**
```json
{
  "query": "用户输入的问题",           // 必填
  "user": "用户标识",                // 必填（Dify）
  "uid": "用户ID",                  // 可选
  "stream": false,                  // 是否流式返回
  "detail": false,                  // 是否返回详细信息
  "app_id": "应用ID",              // 可选（FastGPT）
  "chat_id": "会话ID",             // 可选
  "conversation_id": "对话ID",      // 可选
  "files": ["file_url1", "file_url2"]  // 文件URL列表
}
```

**响应格式：**
```json
{
  "id": "会话ID",
  "model": "使用的模型",
  "usage": {
    "prompt_tokens": 100,
    "completion_tokens": 200,
    "total_tokens": 300
  },
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "AI回复内容"
      },
      "finish_reason": "stop",
      "index": 0
    }
  ]
}
```

### 2. 文件上传接口

**端点：** `POST /api/v1/upload`

**请求头：**
- `agent: {agent_name}` - 指定使用的Agent（必填）

**请求参数：**
- `file`: 上传的文件（multipart/form-data）
- `agent`: Agent名称（可选，也可通过header传递）
- `created_by`: 创建者标识（可选）

**响应格式：**
```json
{
  "code": 0,
  "msg": "File uploaded successfully",
  "data": {
    "id": "文件ID",
    "name": "文件名",
    "size": 1024,
    "extension": "pdf",
    "mime_type": "application/pdf",
    "created_by": "user123",
    "created_at": 1640995200
  }
}
```

### 3. 支持的Agent查询接口

**端点：** `GET /api/v1/upload/agents`

**响应格式：**
```json
{
  "code": 0,
  "msg": "Success",
  "data": {
    "supported_agents": ["fastgpt", "dify"],
    "count": 2
  }
}
```

## 项目优势

### 1. 技术优势
- **架构清晰**：采用适配器模式，代码结构清晰，易于维护
- **扩展性强**：新增Agent只需实现BaseAgent接口即可
- **类型安全**：使用Pydantic模型确保数据验证和类型安全
- **异步支持**：基于FastAPI，支持异步处理和并发请求

### 2. 业务优势
- **统一接口**：前端只需对接一套API，无需适配多个平台
- **灵活切换**：可以根据需求动态切换不同的AI平台
- **成本控制**：可以比较不同平台的性能和成本，选择最优方案
- **风险分散**：避免单一平台依赖，提高系统稳定性

### 3. 运维优势
- **配置简单**：通过环境变量即可配置所有Agent
- **监控友好**：统一的日志格式和错误处理
- **部署便捷**：支持Docker部署，一键启动
- **文档完善**：详细的API文档和使用示例

## 部署指南

### 1. 环境要求
- Python 3.12+
- FastAPI
- 相关依赖包（见requirements.txt）

### 2. 环境变量配置
创建`.env`文件：
```bash
# FastGPT配置
FASTGPT_BASE_URL=https://your-fastgpt-instance.com
FASTGPT_API_KEY=your-api-key
FASTGPT_APP_ID=your-app-id

# Dify配置
DIFY_BASE_URL=https://your-dify-instance.com
DIFY_API_KEY=your-api-key

# Coze配置
COZE_API_TOKEN=your-api-token
COZE_BOT_ID=your-bot-id
COZE_BASE_URL=https://api.coze.cn

# S3/MinIO配置（可选，用于文件上传）
S3_ENDPOINT=https://your-s3-endpoint.com
S3_ACCESS_KEY=your-access-key
S3_SECRET_KEY=your-secret-key
S3_BUCKET=your-bucket-name
S3_REGION=your-region
S3_KEY_PREFIX=uploads
S3_ACL=public-read
S3_PUBLIC_BASE_URL=https://your-cdn-domain.com
```

### 3. 启动服务
```bash
# 安装依赖
pip install -r requirements.txt

# 启动服务
uvicorn main:app --host 0.0.0.0 --port 3000 --reload
```

### 4. Docker部署
```bash
# 构建镜像
docker build -t ai-gateway .

# 运行容器
docker run -d -p 3000:3000 --env-file .env ai-gateway
```

## 使用示例

### 1. 基本对话
```python
import requests

# 使用FastGPT
response = requests.post('http://localhost:3000/api/v1/chat', 
    headers={'agent': 'fastgpt', 'Content-Type': 'application/json'},
    json={
        'query': '你好，请介绍一下胰腺癌的基本知识',
        'user': 'test_user',
        'stream': False,
        'detail': False
    }
)
print(response.json())

# 使用Dify
response = requests.post('http://localhost:3000/api/v1/chat',
    headers={'agent': 'dify', 'Content-Type': 'application/json'},
    json={
        'query': '请分析这个文档',
        'user': 'test_user',
        'stream': False
    }
)
print(response.json())
```

### 2. 流式对话
```python
import requests

response = requests.post('http://localhost:3000/api/v1/chat',
    headers={'agent': 'fastgpt', 'Content-Type': 'application/json'},
    json={
        'query': '请详细介绍一下胰腺癌的治疗方案',
        'user': 'test_user',
        'stream': True,
        'detail': False
    },
    stream=True
)

for line in response.iter_lines():
    if line:
        print(line.decode('utf-8'))
```

### 3. 文件上传
```python
import requests

with open('document.pdf', 'rb') as f:
    response = requests.post('http://localhost:3000/api/v1/upload',
        headers={'agent': 'fastgpt'},
        files={'file': f},
        data={'created_by': 'test_user'}
    )
    print(response.json())
```

## 扩展开发

### 1. 添加新的Agent
1. 在`agent`目录下创建新的Agent文件
2. 继承`BaseAgent`类并实现必要的方法
3. 在Agent类中注册到registry
4. 配置相应的环境变量

### 2. 自定义响应格式
1. 在`agent/models.py`中添加新的响应模型
2. 在Agent的`format_response`方法中实现格式转换
3. 确保响应格式符合前端需求

### 3. 添加新功能
1. 在`BaseAgent`中添加新的抽象方法
2. 在各个Agent中实现该方法
3. 在API路由中添加相应的端点

## 总结

AI Gateway项目通过适配器模式成功实现了多AI平台的统一接口，为前端提供了标准化的服务。项目具有架构清晰、扩展性强、易于维护等优势，特别适合需要支持多个AI平台的应用场景。通过统一的接口和响应格式，开发者可以轻松切换不同的AI后端，实现成本控制和风险分散的目标。