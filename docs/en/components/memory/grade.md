# Grade Memory

The Grade Memory component handles message history, grading results, and conversation context management for agent interactions.

## Architecture

Located in `src/agentsdk/memory/grade/`, this module provides:

- **Message Compression**: Efficient storage of conversation history
- **Grading Results**: Persistence of evaluation outcomes
- **Historical Context**: Access to previous iterations for learning

## Module Structure

### Compression Submodule (`compressor/`)
- **Base Compressor**: Abstract base class for message compression
- **Default Compressor**: Standard compression implementation
- **Prompt Compressor**: Specialized compression for prompt handling

### Storage Submodule (`storage/`)
- **Base Storage**: Abstract base class for storage backends
- **File Storage**: File-based persistent storage
- **In-Memory Storage**: Fast in-memory storage

## Core Components

### Grade Memory Class (`memory.py`)

Main class for managing grade-related memory operations:

```python
from agentsdk.memory.grade import GradeMemory

# Initialize grade memory
grade_memory = GradeMemory(
    compression_type="default",
    storage_backend="in_memory",
    max_history_length=1000
)
```

### Memory Components (`components.py`)

Supporting components for grade memory operations:

```python
# Message compression utilities
from agentsdk.memory.grade.compressor import DefaultCompressor

compressor = DefaultCompressor()
compressed_message = compressor.compress(full_message)
```

## Storage Backends

### In-Memory Storage

Fast storage suitable for development and testing:

```python
from agentsdk.memory.grade.storage import InMemoryStorage

storage = InMemoryStorage(max_history_length=500)
```

### File Storage

Persistent storage for production use:

```python
from agentsdk.memory.grade.storage import FileStorage

storage = FileStorage(storage_path="./memory_data/", max_history_length=1000)
```

## Usage Examples

### Basic Grade Memory Operations

```python
# Store conversation history
conversation_id = await grade_memory.store_conversation(
    messages=[
        {"role": "user", "content": "Query 1"},
        {"role": "assistant", "content": "Response 1"}
    ],
    metadata={"task_type": "mathematical"}
)

# Retrieve conversation history
history = await grade_memory.get_conversation(conversation_id)

# Store grading results
await grade_memory.store_evaluation(
    solution_id="sol_001",
    score=0.85,
    feedback="Good solution with minor improvements needed",
    evaluator="ml_model_v1"
)
```

### Message Compression

```python
# Compress long conversation history
long_history = [...]
compressed_history = await grade_memory.compress_messages(long_history)

# Access compressed context
context = await grade_memory.get_compressed_context(conversation_id, max_tokens=1000)
```

### Historical Learning

```python
# Get best performing solutions for context
successful_patterns = await grade_memory.get_successful_patterns(
    task_type="mathematical",
    min_score=0.8,
    limit=10
)

# Update solution based on historical performance
improved_solution = await grade_memory.learn_from_history(
    current_solution,
    historical_solutions
)
```

## Configuration

### Default Configuration

```yaml
memory:
  compression: "default"
  storage_backend: "in_memory"
  max_history_length: 1000
```

### Advanced Configuration

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

## Best Practices

### Message Management
1. **Selective Compression**: Compress only necessary parts of conversation history
2. **Context Preservation**: Preserve important message content during compression
3. **Storage Optimization**: Clean up old data based on retention policies

### Performance Optimization
1. **Batch Operations**: Use batch storage operations for better performance
2. **Asynchronous Processing**: Leverage async/await for non-blocking operations
3. **Caching Strategy**: Implement appropriate caching for frequently accessed data

## Integration with Evolution Memory

Grade Memory works in conjunction with Evolution Memory:

```python
# Combine evolution and grade memory for complete agent memory
evolution_memory = MemoryFactory(storage_type="in_memory")
grade_memory = GradeMemory(storage_backend="in_memory")

# Store solution with grading context
solution_id = await evolution_memory.add_solution(solution)
await grade_memory.store_evaluation(solution_id, score=0.9, feedback="Excellent")
```

## See Also

- [Memory Overview](../overview.md) - Overall architecture
- [Evolution Memory](../evolution.md) - Evolutionary state management
- [Configuration Guide](../configuration.md) - Complete configuration options