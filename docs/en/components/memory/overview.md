# Memory Component

The Memory component in LoongFlow provides sophisticated state management for evolutionary algorithms, enabling persistent storage, population management, and checkpoint functionality.

## Architecture Overview

LoongFlow's Memory system is organized into two main components:

### Evolution Memory
Manages the core evolutionary process including solution storage, population management, and genetic relationships. Located at `src/agentsdk/memory/evolution/`.

### Grade Memory  
Handles message history, grading results, and conversation context. Located at `src/agentsdk/memory/grade/`.

## Key Features

- **Persistent Storage**: Support for both in-memory and Redis-based storage
- **Population Management**: Island-based evolutionary algorithms
- **Checkpoint System**: Save and restore evolutionary progress
- **Message Compression**: Efficient storage of conversation history
- **Scalable Architecture**: From single-node development to distributed deployment

## When to Use

Use the Memory component when you need:
- Long-term storage of evolutionary solutions
- Checkpointing for long-running optimization tasks
- Message history for agent conversation context
- Distributed memory for scaling evolutionary algorithms

## Quick Start

```python
from agentsdk.memory import MemoryFactory

# Initialize memory with basic configuration
memory = MemoryFactory(
    storage_type="in_memory",
    population_size=100,
    num_islands=3
)
```

For detailed information about specific memory types and advanced configuration, continue reading:
- [Evolution Memory](evolution.md) - Core evolutionary state management
- [Grade Memory](grade.md) - Message and grading history
- [Configuration Guide](configuration.md) - Complete configuration options