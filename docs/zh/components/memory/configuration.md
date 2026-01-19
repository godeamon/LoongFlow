# 存储器配置指南

LoongFlow Memory组件的完整配置参考，包括进化存储器和等级存储器。

## 进化存储器配置

### 基础配置

```yaml
# 进化存储器基础设置
database:
  storage_type: "in_memory"  # 或 "redis"
  population_size: 100       # 每个岛屿最大解决方案数
  num_islands: 3             # 并行进化岛屿数量
  checkpoint_interval: 10    # 每N次迭代保存检查点
```

### 高级配置

```yaml
# 完整进化存储器配置
database:
  storage_type: "redis"
  population_size: 500
  num_islands: 5
  checkpoint_interval: 25
  max_solution_age: 100      # 最大解决方案保留代数
  exploration_rate: 0.1      # 默认探索率
  elitism_count: 5           # 保留的最佳解决方案数量
  
  # Redis特定配置
  redis:
    url: "redis://localhost:6379/0"
    max_connections: 20
    socket_timeout: 5.0
    retry_on_timeout: true
    
  # 检查点配置
  checkpoint:
    auto_save: true
    compression: true
    include_metadata: true
```

### 岛屿特定配置

```yaml
# 每个岛屿的配置
database:
  islands:
    - id: 0
      population_size: 100
      exploration_rate: 0.15
      specialization: "mathematical"
    - id: 1  
      population_size: 80
      exploration_rate: 0.2
      specialization: "algorithmic"
    - id: 2
      population_size: 120
      exploration_rate: 0.05
      specialization: "optimization"
```

## 等级存储器配置

### 基础配置

```yaml
# 等级存储器基础设置
memory:
  compression: "default"      # 压缩算法
  storage_backend: "in_memory" # 或 "file"
  max_history_length: 1000    # 最大存储对话数
```

### 高级配置

```yaml
# 完整等级存储器配置
memory:
  compression:
    type: "default"
    max_tokens: 2000
    preserve_important: true
    algorithms:
      - name: "summarization"
        weight: 0.6
      - name: "extraction"
        weight: 0.4
  
  storage:
    backend: "file"
    path: "./memory_data/"
    max_history_length: 5000
    cleanup_interval: 86400   # 每日清理
    
  grading:
    retention_days: 30
    score_threshold: 0.5      # 存储的最低分数
    feedback_analysis: true
    
  # 文件存储特定配置
  file_storage:
    format: "json"
    compression: "gzip"
    backup_count: 5
```

### 压缩配置

```yaml
# 详细压缩设置
memory:
  compression:
    default:
      target_ratio: 0.3       # 目标压缩比率
      preserve_structure: true
      metadata_preservation:
        - "role"
        - "timestamp"
        - "importance"
    
    prompt_compression:
      enabled: true
      max_tokens: 1000
      preserve_system_messages: true
```

## Python配置

### 编程式配置

```python
from agentsdk.memory import MemoryConfig

# 进化存储器配置
evolution_config = MemoryConfig(
    storage_type="redis",
    population_size=100,
    num_islands=3,
    checkpoint_interval=10,
    redis_url="redis://localhost:6379/0"
)

# 等级存储器配置  
grade_config = MemoryConfig(
    compression_type="default",
    storage_backend="file",
    max_history_length=1000,
    storage_path="./memory_data/"
)
```

### 工厂配置

```python
from agentsdk.memory import MemoryFactory

# 使用自定义设置配置内存工厂
memory = MemoryFactory(
    evolution_config=evolution_config,
    grade_config=grade_config,
    
    # 通用设置
    auto_cleanup=True,
    monitoring_enabled=True,
    log_level="INFO"
)
```

## 基于环境的配置

### 环境变量

```bash
# 进化存储器
export MEMORY_STORAGE_TYPE="redis"
export MEMORY_POPULATION_SIZE="200"
export MEMORY_NUM_ISLANDS="4"
export REDIS_URL="redis://redis-server:6379/0"

# 等级存储器
export GRADE_COMPRESSION_TYPE="default"
export GRADE_STORAGE_BACKEND="file"
export GRADE_MAX_HISTORY="5000"
```

### 配置文件

**config.yaml:**
```yaml
memory:
  evolution:
    storage_type: ${MEMORY_STORAGE_TYPE:-in_memory}
    population_size: ${MEMORY_POPULATION_SIZE:-100}
    num_islands: ${MEMORY_NUM_ISLANDS:-3}
    
  grade:
    compression: ${GRADE_COMPRESSION_TYPE:-default}
    storage_backend: ${GRADE_STORAGE_BACKEND:-in_memory}
    max_history_length: ${GRADE_MAX_HISTORY:-1000}
```

## 性能调优

### 内存优化

```yaml
# 性能优化的配置
database:
  storage_type: "redis"
  population_size: 1000
  optimization:
    batch_size: 100          # 批量操作大小
    cache_size: 10000        # 内存缓存大小
    async_operations: true   # 启用异步操作
    
  redis:
    connection_pool_size: 50
    socket_keepalive: true
```

### 可扩展性配置

```yaml
# 大规模部署配置
database:
  storage_type: "redis"
  population_size: 10000
  num_islands: 10
  redis:
    cluster_mode: true
    nodes:
      - "redis-node-1:6379"
      - "redis-node-2:6379"
      - "redis-node-3:6379"
```

## 监控和维护

### 监控配置

```yaml
# 监控和指标
monitoring:
  enabled: true
  metrics:
    - memory_usage
    - operation_latency
    - population_stats
    - compression_ratio
    
  alerts:
    memory_threshold: 0.8    # 80%内存使用率时告警
    latency_threshold: 1000  # 1秒延迟时告警
```

### 维护配置

```yaml
# 自动化维护
maintenance:
  auto_cleanup: true
  cleanup_schedule: "0 2 * * *"  # 每天凌晨2点
  retention_policy:
    solutions: 30            # 解决方案保留30天
    conversations: 7         # 对话保留7天
    checkpoints: 10          # 保留最后10个检查点
```

## 故障排除配置

### 调试配置

```yaml
# 用于故障排除的调试配置
debug:
  enabled: true
  log_level: "DEBUG"
  trace_operations: true
  performance_profiling: true
  
  # 详细日志记录
  log:
    operations: true
    memory_usage: true
    compression_stats: true
```

### 错误处理配置

```yaml
# 错误处理和恢复
error_handling:
  retry_attempts: 3
  retry_delay: 1.0
  fallback_strategy: "degrade"
  
  # 恢复选项
  recovery:
    auto_recover: true
    checkpoint_rollback: true
    data_validation: true
```

## 最佳实践

### 配置管理
1. **环境特定配置**：为开发、测试和生产环境使用不同的配置
2. **版本控制**：将配置保存在版本控制中，敏感数据使用环境变量
3. **验证**：在启动时验证配置以尽早发现错误

### 性能最佳实践
1. **从简单开始**：开发阶段先使用内存存储
2. **逐步扩展**：根据需要逐步增加种群大小
3. **监控**：启用监控以跟踪性能并识别瓶颈

## 相关链接

- [存储器概述](../overview.md) - 整体架构
- [进化存储器](../evolution.md) - 进化状态管理  
- [等级存储器](../grade.md) - 消息和评分存储