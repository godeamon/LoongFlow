# Custom Agent Development Guide

This guide shows you how to create custom agents by extending LoongFlow's components and frameworks.

## 🎯 When to Build Custom Agents

Consider building custom agents when:
- Your problem domain requires specialized components
- You need domain-specific tool integration
- Existing paradigms don't fit your use case
- You want to experiment with new architectures

## 🏗️ Custom Agent Architecture

### Basic Structure
```
custom_agent/
├── __init__.py
├── custom_agent.py          # Main agent class
├── components/              # Custom components
│   ├── custom_planner.py
│   ├── custom_executor.py
│   └── custom_summary.py
├── tools/                   # Domain-specific tools
│   └── domain_tool.py
└── config/                  # Configuration files
    └── custom_config.yaml
```

### Extending Base Classes

#### Custom Planner
```python
from evolux.base import BasePlanner

class CustomPlanner(BasePlanner):
    def __init__(self, config):
        super().__init__(config)
        self.domain_knowledge = load_domain_knowledge()
    
    async def plan(self, context):
        # Custom planning logic
        plan = await self.generate_domain_plan(context)
        return plan
```

#### Custom Executor
```python
from evolux.base import BaseExecutor

class CustomExecutor(BaseExecutor):
    async def execute(self, plan, context):
        # Domain-specific execution
        results = await self.execute_domain_plan(plan)
        return results
```

#### Custom Summary
```python
from evolux.base import BaseSummary

class CustomSummary(BaseSummary):
    async def summarize(self, results, context):
        # Domain-aware summarization
        insights = self.analyze_domain_results(results)
        return insights
```

## 🔧 Component Integration

### Tool Development
```python
from agentsdk.tools import BaseTool

class DomainSpecificTool(BaseTool):
    name = "domain_tool"
    description = "Tool for domain-specific operations"
    
    async def execute(self, context, **kwargs):
        # Implement domain logic
        result = await self.process_domain_data(kwargs['data'])
        return ToolResponse(success=True, data=result)
```

### Memory Extensions
```python
from agentsdk.memory import MemoryFactory

class DomainMemory(MemoryFactory):
    def __init__(self, domain_config):
        super().__init__()
        self.domain_config = domain_config
    
    async def add_domain_solution(self, solution, domain_metadata):
        # Custom solution storage with domain context
        enhanced_solution = self.enhance_with_domain_data(solution, domain_metadata)
        return await self.add_solution(enhanced_solution)
```

## ⚙️ Configuration Framework

### Custom Configuration Schema
```yaml
# custom_config.yaml
domain:
  specific_parameter: "value"
  resource_constraints:
    max_memory: "8GB"
    timeout: 3600

components:
  custom_planner:
    planning_strategy: "domain_aware"
    max_depth: 10
    
  custom_executor:
    execution_mode: "parallel"
    resource_allocation: "dynamic"
```

## 🚀 Implementation Examples

### Domain-Specific Agent
```python
from evolux.base import BaseAgent
from custom_components import CustomPlanner, CustomExecutor, CustomSummary

class DomainAgent(BaseAgent):
    def __init__(self, config):
        super().__init__(config)
        self.planner = CustomPlanner(config)
        self.executor = CustomExecutor(config) 
        self.summary = CustomSummary(config)
    
    async def run(self, task_description):
        # Custom run loop
        context = self.create_domain_context(task_description)
        
        for iteration in range(self.config.max_iterations):
            plan = await self.planner.plan(context)
            results = await self.executor.execute(plan, context)
            insights = await self.summary.summarize(results, context)
            
            # Domain-specific iteration handling
            context = self.update_domain_context(context, insights)
            
            if self.meets_domain_criteria(results):
                break
```

### Integration with Existing Framework
```python
# Reuse existing components with custom extensions
from evolux.evolve import EvolveAgent
from custom_tools import DomainToolkit

class HybridAgent(EvolveAgent):
    def __init__(self, config):
        super().__init__(config)
        # Extend with domain tools
        self.toolkit.register_toolkit(DomainToolkit())
```

## 📊 Testing and Validation

### Unit Testing Components
```python
import pytest
from custom_components import CustomPlanner

class TestCustomPlanner:
    def test_plan_generation(self):
        planner = CustomPlanner(test_config)
        plan = await planner.plan(test_context)
        assert plan is not None
        assert hasattr(plan, 'domain_specific_attributes')
```

### Integration Testing
```python
class TestDomainAgent:
    async def test_end_to_end(self):
        agent = DomainAgent(integration_config)
        results = await agent.run(test_task)
        assert results['success'] == True
        assert 'domain_metrics' in results
```

## 🔄 Deployment Patterns

### Configuration Management
```python
# Dynamic configuration loading
def load_domain_config(environment):
    base_config = load_base_config()
    domain_config = load_domain_specific_config(environment)
    return merge_configs(base_config, domain_config)
```

### Resource Management
```python
# Custom resource allocation
class DomainResourceManager:
    def allocate_resources(self, task_complexity):
        if task_complexity == "high":
            return {"gpus": 1, "memory": "16GB"}
        else:
            return {"gpus": 0, "memory": "8GB"}
```

## 🎯 Best Practices

### Design Principles
1. **Modularity**: Keep components independent and reusable
2. **Extensibility**: Design for easy future extensions
3. **Testing**: Comprehensive test coverage for custom logic
4. **Documentation**: Clear docstrings and usage examples

### Performance Considerations
```python
# Efficient domain operations
class OptimizedDomainTool(BaseTool):
    async def execute(self, context, large_dataset):
        # Use streaming for large data
        results = []
        async for chunk in self.stream_process(large_dataset):
            results.extend(await self.process_chunk(chunk))
        return ToolResponse(success=True, data=results)
```

### Error Handling
```python
# Robust error handling for domain operations
class SafeDomainExecutor(BaseExecutor):
    async def execute(self, plan, context):
        try:
            return await super().execute(plan, context)
        except DomainSpecificError as e:
            logger.error(f"Domain execution failed: {e}")
            return ExecutionResult(
                success=False, 
                error=f"Domain constraint violation: {e}"
            )
```

## 🔍 Advanced Patterns

### Multi-Agent Systems
```python
# Coordinate multiple domain agents
class DomainOrchestrator:
    def __init__(self, agent_configs):
        self.agents = {
            name: DomainAgent(config) 
            for name, config in agent_configs.items()
        }
    
    async def coordinate(self, complex_task):
        # Distribute subtasks to specialized agents
        results = await asyncio.gather(*[
            agent.run(subtask) 
            for agent, subtask in self.assign_tasks(complex_task)
        ])
        return self.aggregate_results(results)
```

### Domain-Specific Optimization
```python
# Custom optimization strategies
class DomainOptimizer:
    def optimize_parameters(self, domain_constraints):
        # Implement domain-aware optimization
        return self.domain_aware_optimization(domain_constraints)
```

This guide provides the foundation for building custom agents that leverage LoongFlow's architecture while addressing specific domain requirements.
