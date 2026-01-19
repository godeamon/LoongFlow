# 等级存储器

等级存储器组件处理消息历史、评分结果和对话上下文管理，用于智能体交互。

## 架构设计

位于 `src/agentsdk/memory/grade/`，该模块提供：

- **消息压缩**：高效存储对话历史
- **评分结果**：评估结果的持久化
- **历史上下文**：访问先前迭代用于学习

## 模块结构

### 压缩子模块 (`compressor/`)
- **基本压缩器**：消息压缩的抽象基类
- **默认压缩器**：标准压缩实现
- **提示压缩器**：专门处理提示的压缩

### 存储子模块 (`storage/`)
- **基本存储**：存储后端的抽象基类
- **文件存储**：基于文件的持久化存储
- **内存存储**：快速内存存储

## 核心组件

### 等级存储器类 (`memory.py`)

管理等级相关内存操作的主类：

```python
from agentsdk.memory.grade import GradeMemory

# 初始化等级存储器
grade_memory = GradeMemory(
    compression_type="default",
    storage_backend="in_memory",
    max_history_length=1000
)
```

### 存储器组件 (`components.py`)

支持等级存储器操作的组件：

```python
# 消息压缩工具
from agentsdk.memory.grade.compressor import DefaultCompressor

compressor = DefaultCompressor()
compressed_message = compressor.compress(full_message)
```

## 存储后端

### 内存存储

快速存储，适用于开发和测试：

```python
from agentsdk.memory.grade.storage import InMemoryStorage

storage = InMemoryStorage(max_history_length=500)
```

### 文件存储

持久化存储，适用于生产环境：

```python
from agentsdk.memory.grade.storage import FileStorage

storage = FileStorage(storage_path="./memory_data/", max_history_length=1000)
```

## 使用示例

### 基本等级存储器操作

```python
# 存储对话历史
conversation_id = await grade_memory.store_conversation(
    messages=[
        {"role": "user", "content": "查询1"},
        {"role": "assistant", "content": "响应1"}
    ],
    metadata={"task_type": "mathematical"}
)

# 检索对话历史
history = await grade_memory.get_conversation(conversation_id)

# 存储评分结果
await grade_memory.store_evaluation(
    solution_id="sol_001",
    score=0.85,
    feedback="良好解决方案，需要小改进",
    evaluator="ml_model_v1"
)
```

### 消息压缩

```python
# 压缩长对话历史
long_history = [...]
compressed_history = await grade_memory.compress_messages(long_history)

# 访问压缩上下文
context = await grade_memory.get_compressed_context(conversation_id, max_tokens=1000)
```

### 历史学习

```python
# 获取最佳表现解决方案作为上下文
successful_patterns = await grade_memory.get_successful_patterns(
    task_type="mathematical",
    min_score=0.8,
    limit=10
)

# 基于历史性能改进解决方案
improved_solution = await grade_memory.learn_from_history(
    current_solution,
    historical_solutions
)
```

## 配置

### 默认配置

```yaml
memory:
  compression: "default"
  storage_backend: "in_memory"
  max_history_length: 1000
```

### 高级配置

```yaml
memory:
  compression:
    type: "default"
    max_tokens: 2000
    preserve_important: true
  storage:
    backend: "file"
    path: "./memory_data/"
    max_history_length: 5000
  grading:
    retention_days: 30
    cleanup_interval: 86400
```

## 最佳实践

### 消息管理
1. **选择性压缩**：只压缩必要的对话历史部分
2. **上下文保留**：在压缩过程中保留重要消息内容
3. **存储优化**：基于保留策略清理旧数据

### 性能优化
1. **批量操作**：使用批量存储操作提高性能
2. **异步处理**：利用async/await实现非阻塞操作
3. **缓存策略**：为频繁访问的数据实施适当缓存

## 与进化存储器集成

等级存储器与进化存储器协同工作：

```python
# 结合进化和等级存储器实现完整的智能体内存
evolution_memory = MemoryFactory(storage_type="in_memory")
grade_memory = GradeMemory(storage_backend="in_memory")

# 存储解决方案及评分上下文
solution_id = await evolution_memory.add_solution(solution)
await grade_memory.store_evaluation(solution_id, score=0.9, feedback="优秀")
```

## 相关链接

- [存储器概述](../overview.md) - 整体架构
- [进化存储器](../evolution.md) - 进化状态管理
- [配置指南](../configuration.md) - 完整配置选项