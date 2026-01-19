# 进化存储器

进化存储器组件管理核心进化过程状态，包括解决方案存储、种群管理和遗传关系，为进化算法提供基础支持。

## 核心架构

位于 `src/agentsdk/memory/evolution/`，该模块提供：

- **解决方案存储**：持久化存储智能体解决方案及元数据
- **种群管理**：基于岛屿的进化算法
- **父子关系追踪**：跟踪解决方案的遗传关系
- **检查点系统**：保存/恢复进化进度

## 核心类

### MemoryFactory (`memory_factory.py`)

不同内存实现的统一接口：

```python
from agentsdk.memory import MemoryFactory

# 使用配置初始化内存
memory = MemoryFactory(storage_type="in_memory", population_size=100)
```

**存储类型：**
- `in_memory`：快速、非持久化存储（默认）
- `redis`：分布式持久化存储

### BaseMemory (`base_memory.py`)

定义内存操作的抽象基类：

```python
class BaseMemory:
    async def add_solution(self, solution: Solution) -> str
    def get_solutions(self, solution_ids: List[str]) -> List[Solution]
    def sample(self, island_id: int, exploration_rate: float) -> Solution
    async def save_checkpoint(self, path: str, tag: str = None)
```

### Solution类

表示进化解决方案：

```python
@dataclass
class Solution:
    solution_id: str
    code: str
    score: float
    parent_ids: List[str]
    generation: int
    island_id: int
    metadata: Dict[str, Any]
```

## 实现后端

### 内存后端

快速、非持久化存储，适用于开发和测试：

```python
memory = MemoryFactory(
    storage_type="in_memory",
    population_size=100,
    num_islands=3
)
```

### Redis后端

分布式持久化存储，适用于生产环境：

```python
memory = MemoryFactory(
    storage_type="redis",
    redis_url="redis://redis-server:6379/0",
    population_size=1000
)
```

## 使用示例

### 基本内存操作

```python
# 添加新解决方案
solution = Solution(
    solution_id="sol_001",
    code="def solve(): return 42",
    score=0.85,
    parent_ids=[],
    generation=1,
    island_id=0
)
await memory.add_solution(solution)

# 检索最佳解决方案
best_solutions = memory.get_best_solutions(island_id=0, top_k=5)

# 采样用于突变/交叉
parent = memory.sample(island_id=0, exploration_rate=0.1)
```

### 检查点管理

```python
# 保存检查点
await memory.save_checkpoint("./checkpoints/", "iteration_50")

# 加载检查点
memory.load_checkpoint("./checkpoints/iteration_50/")

# 检查内存状态
status = memory.memory_status(island_id=0)
print(f"种群大小: {status['population_size']}")
```

### 基于岛屿的进化

```python
# 配置多个岛屿进行并行进化
memory = MemoryFactory(
    storage_type="in_memory",
    num_islands=3,
    population_size=30
)

# 每个岛屿独立进化
island_0_best = memory.get_best_solutions(island_id=0)
island_1_best = memory.get_best_solutions(island_id=1)
```

## 高级功能

### 自定义内存实现

创建自定义内存后端：

```python
from agentsdk.memory.evolution.base_memory import BaseMemory

class CustomMemory(BaseMemory):
    def __init__(self, custom_config: Dict):
        self.config = custom_config
    
    async def add_solution(self, solution: Solution) -> str:
        # 自定义实现
        pass
    
    # 实现其他必需方法...
```

## 最佳实践

1. **检查点频率**：长时间运行任务每5-10次迭代保存检查点
2. **内存监控**：特别监控大种群时的内存使用情况
3. **岛屿配置**：复杂优化问题使用多个岛屿（3-5个）

## 相关链接

- [存储器概述](../overview.md) - 整体架构
- [等级存储器](../grade.md) - 消息和评分存储
- [配置指南](../configuration.md) - 完整配置选项
