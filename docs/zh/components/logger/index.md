# 日志器组件

日志器组件为LoongFlow智能体提供全面的日志系统，支持结构化日志记录、上下文管理和灵活的输出格式。

## 架构设计

### 多层日志系统
- **应用日志**：高级智能体执行日志
- **组件日志**：各组件活动日志
- **调试日志**：详细调试信息
- **错误日志**：错误和异常跟踪

### 日志级别
- **DEBUG**：详细调试信息
- **INFO**：一般操作信息
- **WARNING**：警告条件
- **ERROR**：错误条件
- **CRITICAL**：严重条件

## 核心类

### Logger (`logger.py`)
提供日志接口的主要日志类：

```python
from agentsdk.logger import get_logger

# 获取日志器实例
logger = get_logger("my_component")

# 不同级别日志记录
logger.debug("详细调试信息")
logger.info("操作成功完成")
logger.warning("检测到潜在问题")
logger.error("操作失败", exc_info=True)
```

### ContextLogger (`context.py`)
具有自动上下文传播的上下文感知日志：

```python
from agentsdk.logger import ContextLogger

# 创建上下文日志器
context_logger = ContextLogger(
    component="planner",
    request_id="req_123",
    iteration=5
)

# 带上下文的日志记录
context_logger.info("计划迭代开始")
```

### MessageLogger (`message_logger.py`)
专门用于消息处理的日志记录：

```python
from agentsdk.logger import MessageLogger

message_logger = MessageLogger()
message_logger.log_message_sent(message, recipient)
message_logger.log_message_received(message, sender)
```

## 配置

### 基础配置
```yaml
logging:
  level: "INFO"
  format: "json"  # 或 "text"
  output: "file"  # 或 "console", "both"
  
  # 文件日志
  file_path: "./logs/evolux.log"
  max_file_size: "100MB"
  backup_count: 5
  
  # 控制台日志
  console_format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
```

### 高级配置
```yaml
logging:
  # 组件特定级别
  loggers:
    agentsdk.memory: "DEBUG"
    agentsdk.tools: "INFO"
    evolux.planner: "WARNING"
  
  # 结构化日志
  structured: true
  include_context: true
  include_traceback: true
  
  # 性能设置
  buffer_size: 1000
  flush_interval: 30  # 秒
```

## 使用示例

### 基础日志记录
```python
from agentsdk.logger import get_logger

logger = get_logger(__name__)

class MyComponent:
    async def process(self, data):
        logger.info("开始数据处理", extra={"data_size": len(data)})
        
        try:
            result = await self._complex_operation(data)
            logger.debug("操作完成", extra={"result": result})
            return result
            
        except Exception as e:
            logger.error("处理失败", exc_info=True)
            raise
```

### 上下文感知日志
```python
from agentsdk.logger import ContextLogger

async def process_request(request_id, data):
    # 为此请求创建上下文日志器
    context_logger = ContextLogger(
        request_id=request_id,
        component="request_processor",
        user_id=data.get('user_id')
    )
    
    context_logger.info("请求处理开始")
    
    # 此上下文中的所有日志都将包含request_id
    result = await process_data(data)
    context_logger.info("请求完成", extra={"result": result})
```

### 结构化日志
```python
# JSON格式的结构化日志
logger.info("进化迭代完成", extra={
    "iteration": 42,
    "population_size": 100,
    "best_score": 0.95,
    "duration_seconds": 120.5,
    "component": "evolve_agent"
})
```

## 高级功能

### 自定义日志处理器
```python
import logging
from agentsdk.logger import get_logger

# 创建自定义处理器
class CustomHandler(logging.Handler):
    def emit(self, record):
        # 自定义发射逻辑（例如发送到外部系统）
        pass

# 注册自定义处理器
logger = get_logger("custom_component")
logger.addHandler(CustomHandler())
```

### 日志过滤
```python
class ComponentFilter(logging.Filter):
    def filter(self, record):
        # 基于组件或其他条件过滤日志
        return hasattr(record, 'component') and record.component == 'memory'

logger.addFilter(ComponentFilter())
```

### 性能监控
```python
from agentsdk.logger import performance_logger

@performance_logger
async def expensive_operation(data):
    # 此调用将自动记录时间信息
    result = await do_expensive_work(data)
    return result
```

## 最佳实践

### 日志记录指南
1. **有意义的消息**：包含上下文和相关数据
2. **适当级别**：细节使用DEBUG，失败使用ERROR
3. **结构化数据**：使用extra参数进行机器可读数据
4. **安全性**：避免记录敏感信息

### 性能考虑
```python
# 昂贵日志操作的延迟评估
if logger.isEnabledFor(logging.DEBUG):
    # 仅在启用DEBUG时计算昂贵的调试信息
    debug_info = compute_expensive_debug_info()
    logger.debug("详细信息: %s", debug_info)
```

### 错误处理
```python
# 错误场景中的稳健日志记录
try:
    risky_operation()
except Exception as e:
    logger.error(
        "操作失败，出现意外错误",
        exc_info=True,
        extra={
            "operation": "risky_operation",
            "error_type": type(e).__name__
        }
    )
    # 重新抛出或适当处理
```

## 故障排除

### 常见问题

**日志文件轮转**
- 确保日志文件有足够的磁盘空间
- 配置适当的max_file_size和backup_count
- 监控日志目录大小

**性能影响**
- 生产环境中使用适当的日志级别
- 高吞吐量场景考虑异步日志记录
- 除非需要，否则禁用调试日志记录

**缺少上下文**
- 确保上下文信息正确传递给日志器
- 请求范围日志记录使用ContextLogger
- 验证包含extra参数

日志器组件为开发和生成环境提供灵活而强大的日志基础设施。