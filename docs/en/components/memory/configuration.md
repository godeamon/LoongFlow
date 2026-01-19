# Memory Configuration Guide

Complete configuration reference for LoongFlow Memory components, including both Evolution Memory and Grade Memory.

## Evolution Memory Configuration

### Basic Configuration

```yaml
# Evolution memory basic setup
database:
  storage_type: "in_memory"  # or "redis"
  population_size: 100       # Maximum solutions per island
  num_islands: 3             # Number of parallel evolution islands
  checkpoint_interval: 10    # Save checkpoint every N iterations
```

### Advanced Configuration

```yaml
# Complete evolution memory configuration
database:
  storage_type: "redis"
  population_size: 500
  num_islands: 5
  checkpoint_interval: 25
  max_solution_age: 100      # Maximum generations to keep solutions
  exploration_rate: 0.1      # Default exploration rate
  elitism_count: 5           # Best solutions to preserve
  
  # Redis-specific configuration
  redis:
    url: "redis://localhost:6379/0"
    max_connections: 20
    socket_timeout: 5.0
    retry_on_timeout: true
    
  # Checkpoint configuration
  checkpoint:
    auto_save: true
    compression: true
    include_metadata: true
```

### Island-Specific Configuration

```yaml
# Per-island configuration
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

## Grade Memory Configuration

### Basic Configuration

```yaml
# Grade memory basic setup
memory:
  compression: "default"      # Compression algorithm
  storage_backend: "in_memory" # or "file"
  max_history_length: 1000    # Maximum stored conversations
```

### Advanced Configuration

```yaml
# Complete grade memory configuration
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
    cleanup_interval: 86400   # Daily cleanup
    
  grading:
    retention_days: 30
    score_threshold: 0.5      # Minimum score to store
    feedback_analysis: true
    
  # File storage specific
  file_storage:
    format: "json"
    compression: "gzip"
    backup_count: 5
```

### Compression Configuration

```yaml
# Detailed compression settings
memory:
  compression:
    default:
      target_ratio: 0.3       # Target compression ratio
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

## Python Configuration

### Programmatic Configuration

```python
from agentsdk.memory import MemoryConfig

# Evolution memory configuration
evolution_config = MemoryConfig(
    storage_type="redis",
    population_size=100,
    num_islands=3,
    checkpoint_interval=10,
    redis_url="redis://localhost:6379/0"
)

# Grade memory configuration  
grade_config = MemoryConfig(
    compression_type="default",
    storage_backend="file",
    max_history_length=1000,
    storage_path="./memory_data/"
)
```

### Factory Configuration

```python
from agentsdk.memory import MemoryFactory

# Configure memory factory with custom settings
memory = MemoryFactory(
    evolution_config=evolution_config,
    grade_config=grade_config,
    
    # Common settings
    auto_cleanup=True,
    monitoring_enabled=True,
    log_level="INFO"
)
```

## Environment-Based Configuration

### Environment Variables

```bash
# Evolution memory
export MEMORY_STORAGE_TYPE="redis"
export MEMORY_POPULATION_SIZE="200"
export MEMORY_NUM_ISLANDS="4"
export REDIS_URL="redis://redis-server:6379/0"

# Grade memory
export GRADE_COMPRESSION_TYPE="default"
export GRADE_STORAGE_BACKEND="file"
export GRADE_MAX_HISTORY="5000"
```

### Configuration Files

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

## Performance Tuning

### Memory Optimization

```yaml
# Performance-optimized configuration
database:
  storage_type: "redis"
  population_size: 1000
  optimization:
    batch_size: 100          # Batch operations size
    cache_size: 10000        # In-memory cache size
    async_operations: true   # Enable async operations
    
  redis:
    connection_pool_size: 50
    socket_keepalive: true
```

### Scalability Configuration

```yaml
# Large-scale deployment
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

## Monitoring and Maintenance

### Monitoring Configuration

```yaml
# Monitoring and metrics
monitoring:
  enabled: true
  metrics:
    - memory_usage
    - operation_latency
    - population_stats
    - compression_ratio
    
  alerts:
    memory_threshold: 0.8    # Alert at 80% memory usage
    latency_threshold: 1000  # Alert at 1s latency
```

### Maintenance Configuration

```yaml
# Automated maintenance
maintenance:
  auto_cleanup: true
  cleanup_schedule: "0 2 * * *"  # Daily at 2 AM
  retention_policy:
    solutions: 30            # Keep solutions for 30 days
    conversations: 7         # Keep conversations for 7 days
    checkpoints: 10          # Keep last 10 checkpoints
```

## Troubleshooting Configuration

### Debug Configuration

```yaml
# Debug configuration for troubleshooting
debug:
  enabled: true
  log_level: "DEBUG"
  trace_operations: true
  performance_profiling: true
  
  # Detailed logging
  log:
    operations: true
    memory_usage: true
    compression_stats: true
```

### Error Handling Configuration

```yaml
# Error handling and recovery
error_handling:
  retry_attempts: 3
  retry_delay: 1.0
  fallback_strategy: "degrade"
  
  # Recovery options
  recovery:
    auto_recover: true
    checkpoint_rollback: true
    data_validation: true
```

## Best Practices

### Configuration Management
1. **Environment-Specific Configs**: Use different configurations for development, staging, and production
2. **Version Control**: Keep configurations in version control with sensitive data in environment variables
3. **Validation**: Validate configurations at startup to catch errors early

### Performance Best Practices
1. **Start Simple**: Begin with in-memory storage for development
2. **Gradual Scaling**: Increase population size gradually as needed
3. **Monitoring**: Enable monitoring to track performance and identify bottlenecks

## See Also

- [Memory Overview](../overview.md) - Overall architecture
- [Evolution Memory](../evolution.md) - Evolutionary state management  
- [Grade Memory](../grade.md) - Message and grading storage