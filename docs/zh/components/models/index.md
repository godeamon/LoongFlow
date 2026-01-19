# 模型组件

模型组件提供LLM（大语言模型）的集成和管理系统，支持多种模型供应商的统一接口和工具调用功能。

## 架构设计

### 模型抽象层
- **统一接口**: 不同LLM供应商的标准化接口
- **适配器模式**: 支持OpenAI、Gemini、DeepSeek等主流模型
- **配置管理**: 灵活模型配置和切换

### 请求/响应处理
- **结构化请求**: 标准化的模型输入格式
- **响应解析**: 统一的响应处理和错误处理
- **流式支持**: 流式响应的增量处理

## 核心类

### ModelFactory (`model_factory.py`)
模型实例化工厂：

```python
from agentsdk.models import ModelFactory

# 创建模型实例
model = ModelFactory.create_model(
    provider="openai",
    model_name="gpt-4",
    api_key="your-api-key"
)
```

### BaseModel (`base_model.py`)
模型抽象基类：

```python
from agentsdk.models import BaseModel

class CustomModel(BaseModel):
    async def generate(self, messages: List[dict], **kwargs):
        # 模型生成实现
        pass
    
    async def generate_stream(self, messages: List[dict], **kwargs):
        # 流式生成实现
        pass
```

### ModelResponse (`response.py`)
模型响应处理：

```python
from agentsdk.models import ModelResponse

# 处理模型响应
response = ModelResponse(
    content="模型生成的文本",
    usage={"tokens": 100},
    finish_reason="stop"
)
```

## 配置

### 基础模型配置
```yaml
llm_config:
  provider: "openai"
  model: "gpt-4"
  api_key: "your-api-key"
  base_url: "https://api.openai.com/v1"
```

### 高级配置选项
```yaml
llm_config:
  provider: "openai"
  model: "gpt-4"
  temperature: 0.7
  max_tokens: 2000
  timeout: 30
  retry_attempts: 3
  tools:  # 工具调用配置
    - name: "file_reader"
    - name: "code_executor"
```

## 使用示例

### 基础模型调用
```python
from agentsdk.models import ModelFactory

# 创建模型实例
model = ModelFactory.create_model(
    provider="openai",
    model="gpt-4",
    api_key="your-api-key"
)

# 生成文本
messages = [
    {"role": "user", "content": "请解释人工智能"}
]
response = await model.generate(messages)
print(response.content)
```

### 工具调用集成
```python
# 使用工具调用的模型请求
tools = [
    {
        "type": "function",
        "function": {
            "name": "calculator",
            "description": "计算数学表达式",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {"type": "string"}
                }
            }
        }
    }
]

response = await model.generate(
    messages=messages,
    tools=tools,
    tool_choice="auto"
)
```

### 流式响应处理
```python
# 处理流式响应
async for chunk in model.generate_stream(messages):
    if chunk.content:
        print(chunk.content, end="", flush=True)
    if chunk.finish_reason:
        print(f"\n完成原因: {chunk.finish_reason}")
```

## 高级功能

### 自定义模型适配器
```python
from agentsdk.models import BaseModel

class CustomProviderModel(BaseModel):
    def __init__(self, config: dict):
        self.config = config
        # 自定义初始化逻辑
    
    async def generate(self, messages: List[dict], **kwargs):
        # 实现自定义模型调用逻辑
        pass
```

### 模型缓存
```python
from agentsdk.models import CachedModel

# 使用缓存模型
cached_model = CachedModel(
    base_model=model,
    cache_ttl=3600  # 缓存1小时
)
```

## 最佳实践

### 模型选择指南
1. **任务匹配**: 根据任务复杂度选择合适的模型
2. **成本优化**: 平衡模型性能和调用成本
3. **失败处理**: 实现重试机制和降级策略

### 性能优化
```python
# 批量处理消息
batch_messages = [messages1, messages2, messages3]
batch_results = await model.generate_batch(batch_messages)

# 异步并发调用
import asyncio
tasks = [model.generate(msg) for msg in messages_list]
results = await asyncio.gather(*tasks)
```

## 故障排除

### 常见问题

**认证失败**
- 验证API密钥和端点URL
- 检查网络连接和防火墙设置

**速率限制**
- 实现适当的重试和退避策略
- 考虑使用多个API密钥轮换

**响应格式错误**
- 验证请求格式符合模型要求
- 检查工具调用参数的正确性

模型组件为智能体提供强大而灵活的LLM集成能力，支持复杂的AI应用场景。