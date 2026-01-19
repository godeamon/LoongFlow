# 存储器组件

LoongFlow中的Memory组件为进化算法提供复杂的状态管理，支持持久化存储、种群管理和检查点功能。

## 架构概述

LoongFlow的内存系统分为两个主要组件：

### 进化存储器
管理核心进化过程，包括解决方案存储、种群管理和遗传关系。位于 `src/agentsdk/memory/evolution/`。

### 等级存储器
处理消息历史、评分结果和对话上下文。位于 `src/agentsdk/memory/grade/`。

## 主要特性

- **持久化存储**：支持内存和Redis两种存储方式
- **种群管理**：基于岛屿的进化算法
- **检查点系统**：保存和恢复进化进度
- **消息压缩**：高效存储对话历史
- **可扩展架构**：从单节点开发到分布式部署

## 使用场景

在以下情况下使用Memory组件：
- 需要长期存储进化解决方案
- 长时间运行优化任务的检查点功能
- 智能体对话上下文的Message历史
- 需要扩展进化算法的分布式内存

## 快速开始

```python
from agentsdk.memory import MemoryFactory

# 使用基本配置初始化内存
memory = MemoryFactory(
    storage_type="in_memory",
    population_size=100,
    num_islands=3
)
```

有关特定内存类型和高级配置的详细信息，请继续阅读：
- [进化存储器](evolution.md) - 核心进化状态管理
- [等级存储器](grade.md) - 消息和评分历史
- [配置指南](configuration.md) - 完整配置选项