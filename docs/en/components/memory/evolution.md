# Evolution Memory

The Evolution Memory component manages the core evolutionary process state, including solution storage, population management, and genetic relationships for evolutionary algorithms.

## Core Architecture

Located in `src/agentsdk/memory/evolution/`, this module provides:

- **Solution Storage**: Persistent storage of agent solutions with metadata
- **Population Management**: Island-based evolutionary algorithms
- **Parent-Child Relationships**: Tracking solution genealogy
- **Checkpoint System**: Save/restore evolutionary progress

## Core Classes

### MemoryFactory (`memory_factory.py`)

Unified interface for different memory implementations:

```python
from agentsdk.memory import MemoryFactory

# Initialize memory with configuration
memory = MemoryFactory(storage_type="in_memory", population_size=100)
```

**Storage Types:**
- `in_memory`: Fast, non-persistent storage (default)
- `redis`: Distributed persistent storage

### BaseMemory (`base_memory.py`)

Abstract base class defining memory operations:

```python
class BaseMemory:
    async def add_solution(self, solution: Solution) -> str
    def get_solutions(self, solution_ids: List[str]) -> List[Solution]
    def sample(self, island_id: int, exploration_rate: float) -> Solution
    async def save_checkpoint(self, path: str, tag: str = None)
```

### Solution Class

Represents an evolutionary solution:

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

## Implementation Backends

### InMemory Backend

Fast, non-persistent storage for development and testing:

```python
memory = MemoryFactory(
    storage_type="in_memory",
    population_size=100,
    num_islands=3
)
```

### Redis Backend

Distributed persistent storage for production:

```python
memory = MemoryFactory(
    storage_type="redis",
    redis_url="redis://redis-server:6379/0",
    population_size=1000
)
```

## Usage Examples

### Basic Memory Operations

```python
# Add a new solution
solution = Solution(
    solution_id="sol_001",
    code="def solve(): return 42",
    score=0.85,
    parent_ids=[],
    generation=1,
    island_id=0
)
await memory.add_solution(solution)

# Retrieve best solutions
best_solutions = memory.get_best_solutions(island_id=0, top_k=5)

# Sample for mutation/crossover
parent = memory.sample(island_id=0, exploration_rate=0.1)
```

### Checkpoint Management

```python
# Save checkpoint
await memory.save_checkpoint("./checkpoints/", "iteration_50")

# Load checkpoint  
memory.load_checkpoint("./checkpoints/iteration_50/")

# Check memory status
status = memory.memory_status(island_id=0)
print(f"Population size: {status['population_size']}")
```

### Island-Based Evolution

```python
# Configure multiple islands for parallel evolution
memory = MemoryFactory(
    storage_type="in_memory",
    num_islands=3,
    population_size=30
)

# Each island evolves independently
island_0_best = memory.get_best_solutions(island_id=0)
island_1_best = memory.get_best_solutions(island_id=1)
```

## Advanced Features

### Custom Memory Implementation

To create a custom memory backend:

```python
from agentsdk.memory.evolution.base_memory import BaseMemory

class CustomMemory(BaseMemory):
    def __init__(self, custom_config: Dict):
        self.config = custom_config
    
    async def add_solution(self, solution: Solution) -> str:
        # Custom implementation
        pass
    
    # Implement other required methods...
```

## Best Practices

1. **Checkpoint Frequency**: Save checkpoints every 5-10 iterations for long-running tasks
2. **Memory Monitoring**: Monitor memory usage especially with large populations
3. **Island Configuration**: Use multiple islands (3-5) for complex optimization problems

## See Also

- [Memory Overview](../overview.md) - Overall architecture
- [Grade Memory](../grade.md) - Message and grading storage
- [Configuration Guide](../configuration.md) - Complete configuration options