# 消息组件

消息组件提供智能体间通信的消息传递和内容管理系统，支持结构化消息格式和高效序列化。

## 架构设计

### 消息类型系统
- **基础消息**: 标准对话、思考和观察消息
- **工具调用消息**: 智能体与工具交互的结构化通信
- **执行消息**: 代码执行和分析的结果消息

### 内容元素
支持丰富的内容类型，包括文本、代码、数据和多媒体内容。

## 核心类

### Message (`message.py`)
基础消息类，所有消息的基类：

```python
from agentsdk.message import Message

# 创建基础消息
message = Message(
    role="assistant",
    content="这是一条消息",
    timestamp=datetime.now()
)
```

### ToolMessage (`tool_message.py`)
工具调用和分析消息：

```python
from agentsdk.message import ToolMessage

# 工具调用消息
tool_message = ToolMessage(
    tool_name="file_reader",
    arguments={"path": "data.txt"},
    result="文件内容..."
)
```

### ContentElement (`content_element.py`)
消息内容元素：

```python
from agentsdk.message import TextContent, CodeContent

# 文本内容
text_content = TextContent(text="这是文本内容")

# 代码内容
code_content = CodeContent(
    language="python",
    code="def hello(): print('world')"
)
```

## 使用示例

### 基础消息处理
```python
from agentsdk.message import Message, TextContent

# 创建消息
message = Message(
    role="user",
    content=TextContent(text="请帮助我解决这个数学问题"),
    metadata={"priority": "high"}
)

# 序列化消息
serialized = message.serialize()

# 反序列化
new_message = Message.deserialize(serialized)
```

### 工具消息交互
```python
from agentsdk.message import ToolMessage

# 工具调用消息
tool_call = ToolMessage(
    tool_name="calculator",
    arguments={"expression": "2 + 2"},
    metadata={"request_id": "req_123"}
)

# 工具响应消息
tool_response = ToolMessage(
    tool_name="calculator",
    result="4",
    metadata={"request_id": "req_123"}
)
```

### 复杂消息构建
```python
from agentsdk.message import Message, TextContent, CodeContent, DataContent

# 包含多种内容的消息
complex_message = Message(
    role="assistant",
    content=[
        TextContent(text="这是分析结果："),
        CodeContent(
            language="python",
            code="result = analyze_data(data)",
            metadata={"execution_time": "0.5s"}
        ),
        DataContent(data={"score": 0.95, "confidence": 0.99})
    ],
    metadata={"analysis_id": "analyze_001"}
)
```

## 最佳实践

### 消息设计原则
1. **单一职责**: 每条消息应专注于单个主题或操作
2. **结构清晰**: 使用适当的消息类型和内容元素
3. **元数据丰富**: 包含足够的上下文信息

### 性能优化
```python
# 消息压缩
from agentsdk.message import compress_message, decompress_message

# 压缩大消息
compressed = compress_message(large_message)
# 解压缩
decompressed = decompress_message(compressed)
```

消息组件为智能体间通信提供可靠的基础设施，支持复杂的协作场景。