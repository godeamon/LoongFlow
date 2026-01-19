# Token管理组件

Token管理组件负责令牌化、计数和预算管理，为智能体提供高效的令牌使用监控和优化。

## 架构设计

### 令牌化系统
- **多方案支持**: 支持不同的令牌化算法
- **精确计数**: 准确统计输入和输出的令牌数量
- **缓存优化**: 令牌化结果的缓存机制

### 预算管理
- **运行时跟踪**: 实时代理令牌使用情况
- **预算限制**: 基于预算的资源分配和控制
- **成本优化**: 令牌使用效率的优化策略

## 核心类

### Tokenizer (`tokenizer.py`)
令牌化核心类：

```python
from agentsdk.token import Tokenizer

# 创建令牌化器
tokenizer = Tokenizer(model_name="gpt-4")

# 令牌化文本
tokens = tokenizer.encode("这是一段文本")
token_count = len(tokens)

# 令牌计数
count = tokenizer.count_tokens("需要计数的文本")
```

### TokenBudget (`budget.py`)
令牌预算管理：

```python
from agentsdk.token import TokenBudget

# 创建预算管理器
budget = TokenBudget(
    total_budget=100000,
    warning_threshold=0.8  # 80%使用率时警告
)

# 记录令牌使用
budget.record_usage(input_tokens=100, output_tokens=50)
remaining = budget.get_remaining()
```

### TokenOptimizer (`optimizer.py`)
令牌使用优化：

```python
from agentsdk.token import TokenOptimizer

optimizer = TokenOptimizer()
optimized_text = optimizer.optimize_text(
    text="需要优化的长文本",
    target_tokens=500
)
```

## 配置

### 基础配置
```yaml
token:
  model: "gpt-4"  # 默认令牌化模型
  budget:
    total: 1000000
    warning_threshold: 0.9
```

### 高级配置
```yaml
token:
  models:
    gpt-4:
      tokens_per_message: 3
      tokens_per_name: 1
    claude-3:
      tokens_per_message: 4
  optimization:
    enabled: true
    compression_ratio: 0.7
```

## 使用示例

### 基础令牌操作
```python
from agentsdk.token import Tokenizer, TokenBudget

# 初始化
tokenizer = Tokenizer("gpt-4")
budget = TokenBudget(1000000)

# 消息令牌计数
messages = [
    {"role": "user", "content": "你好"},
    {"role": "assistant", "content": "你好！有什么可以帮助你的？"}
]

total_tokens = 0
for msg in messages:
    tokens = tokenizer.count_message_tokens(msg)
    total_tokens += tokens

# 记录使用
budget.record_usage(total_tokens, 0)
```

### 预算监控
```python
class BudgetAwareAgent:
    def __init__(self, budget: TokenBudget):
        self.budget = budget
    
    async def process_request(self, request):
        # 检查预算
        if self.budget.get_remaining() < 1000:
            raise Exception("令牌预算不足")
        
        # 处理请求并记录使用
        response = await self._generate_response(request)
        self.budget.record_usage(
            response.input_tokens, 
            response.output_tokens
        )
        
        return response
```

### 文本优化
```python
from agentsdk.token import TokenOptimizer

optimizer = TokenOptimizer()

# 压缩长文本
long_text = "这是一段非常长的文本内容..." * 100
compressed = optimizer.compress_text(long_text, max_tokens=500)

# 智能截断
truncated = optimizer.truncate_text(long_text, max_tokens=1000)
```

## 高级功能

### 自定义令牌化器
```python
from agentsdk.token import BaseTokenizer

class CustomTokenizer(BaseTokenizer):
    def encode(self, text: str) -> List[int]:
        # 自定义令牌化逻辑
        pass
    
    def decode(self, tokens: List[int]) -> str:
        # 自定义反令牌化逻辑
        pass
```

### 多模型支持
```python
# 为不同模型使用不同的令牌化器
gpt_tokenizer = Tokenizer("gpt-4")
claude_tokenizer = Tokenizer("claude-3")

# 根据模型动态选择
def get_tokenizer(model_name: str) -> Tokenizer:
    return Tokenizer(model_name)
```

## 最佳实践

### 预算管理策略
1. **渐进式使用**: 避免单次消耗过多令牌
2. **监控预警**: 设置合理的警告阈值
3. **成本分摊**: 为不同任务分配不同的预算额度

### 优化技巧
```python
# 批量处理优化
def optimize_batch_requests(requests, max_batch_tokens=4000):
    optimized_batch = []
    current_batch_tokens = 0
    
    for req in requests:
        req_tokens = tokenizer.count_tokens(req.text)
        if current_batch_tokens + req_tokens <= max_batch_tokens:
            optimized_batch.append(req)
            current_batch_tokens += req_tokens
    
    return optimized_batch
```

## 故障排除

### 常见问题

**令牌计数不准确**
- 验证令牌化模型与使用的LLM匹配
- 检查特殊字符和编码问题

**预算超限**
- 实现更严格的预算检查
- 添加预算重置或补充机制

**性能问题**
- 启用令牌化缓存
- 优化批量处理逻辑

Token管理组件帮助智能体高效管理令牌资源，确保在预算范围内实现最佳性能。