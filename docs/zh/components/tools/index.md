# 工具组件

工具组件为智能体提供可扩展的工具接口，支持文件操作、代码执行、Shell命令等常用功能，是可插拔架构的核心。

## 架构设计

### 工具分类系统
- **文件操作**: 读取、写入、列出和管理文件
- **代码执行**: 在受控环境中执行Python代码
- **Shell命令**: 安全运行系统命令
- **自定义工具**: 领域特定工具的可扩展接口

### 工具注册机制
- **动态注册**: 运行时工具注册和发现
- **权限控制**: 基于上下文的工具权限管理
- **错误处理**: 统一的工具错误处理和恢复

## 核心类

### Toolkit (`toolkit.py`)
工具管理核心类：

```python
from agentsdk.tools import Toolkit

# 创建工具包
toolkit = Toolkit()

# 注册工具
from agentsdk.tools import FileReadTool, FileWriteTool
toolkit.register_tool(FileReadTool())
toolkit.register_tool(FileWriteTool())
```

### BaseTool (`base_tool.py`)
工具抽象基类：

```python
from agentsdk.tools import BaseTool

class CustomTool(BaseTool):
    name = "custom_tool"
    description = "自定义工具描述"
    
    async def execute(self, context, **kwargs):
        # 工具执行逻辑
        return {"result": "执行成功"}
```

### ToolExecutor (`executor.py`)
工具执行器：

```python
from agentsdk.tools import ToolExecutor

executor = ToolExecutor(toolkit)
result = await executor.execute_tool("file_read", {"path": "data.txt"})
```

## 内置工具

### 文件操作工具
```python
from agentsdk.tools import FileReadTool, FileWriteTool, ListFilesTool

# 文件读取
file_content = await FileReadTool().execute(path="example.txt")

# 文件写入
await FileWriteTool().execute(path="output.txt", content="Hello World")

# 列出文件
files = await ListFilesTool().execute(directory=".")
```

### 代码执行工具
```python
from agentsdk.tools import PythonExecuteTool

# 执行Python代码
result = await PythonExecuteTool().execute(code="print('Hello')")

# 带上下文的代码执行
result = await PythonExecuteTool().execute(
    code="x + y",
    context={"x": 10, "y": 20}
)
```

### Shell命令工具
```python
from agentsdk.tools import ShellExecuteTool

# 执行Shell命令
result = await ShellExecuteTool().execute(command="ls -la")

# 带工作目录的命令
result = await ShellExecuteTool().execute(
    command="python script.py",
    working_dir="/path/to/project"
)
```

## 配置

### 工具权限配置
```yaml
tools:
  permissions:
    file_read: true
    file_write: false  # 禁用文件写入
    shell_execute: true
    python_execute: true
```

### 执行环境配置
```yaml
tools:
  python:
    timeout: 30
    sandbox: true
  shell:
    timeout: 60
    allowed_commands:
      - "ls"
      - "cat"
      - "python"
```

## 使用示例

### 基础工具使用
```python
from agentsdk.tools import Toolkit, FileReadTool, PythonExecuteTool

# 创建工具包并注册工具
toolkit = Toolkit()
toolkit.register_tool(FileReadTool())
toolkit.register_tool(PythonExecuteTool())

# 执行工具
file_content = await toolkit.execute("file_read", {"path": "data.txt"})
python_result = await toolkit.execute("python_execute", {"code": "2 + 2"})
```

### 自定义工具开发
```python
from agentsdk.tools import BaseTool

class CalculatorTool(BaseTool):
    name = "calculator"
    description = "执行数学计算"
    
    async def execute(self, context, expression: str):
        try:
            result = eval(expression)
            return {"result": result, "success": True}
        except Exception as e:
            return {"error": str(e), "success": False}

# 注册自定义工具
toolkit.register_tool(CalculatorTool())
```

### 工具链组合
```python
async def data_processing_pipeline(toolkit, input_file, output_file):
    # 读取文件
    data = await toolkit.execute("file_read", {"path": input_file})
    
    # 处理数据
    processed = await toolkit.execute("python_execute", {
        "code": f"process_data({data})"
    })
    
    # 写入结果
    await toolkit.execute("file_write", {
        "path": output_file,
        "content": processed
    })
    
    return processed
```

## 高级功能

### 工具权限控制
```python
from agentsdk.tools import PermissionManager

# 创建权限管理器
permissions = PermissionManager({
    "file_read": True,
    "file_write": False,
    "shell_execute": ["ls", "cat"]
})

# 检查权限
if permissions.check("file_write"):
    await toolkit.execute("file_write", {...})
```

### 工具监控
```python
from agentsdk.tools import ToolMonitor

monitor = ToolMonitor()

# 监控工具执行
async with monitor.track("file_read"):
    result = await toolkit.execute("file_read", {"path": "data.txt"})

# 获取执行统计
stats = monitor.get_statistics()
```

## 最佳实践

### 安全考虑
1. **沙箱执行**: 代码执行在隔离环境中进行
2. **输入验证**: 严格验证工具参数
3. **权限最小化**: 只授予必要的工具权限

### 性能优化
```python
# 批量工具执行
async def batch_tool_execution(toolkit, operations):
    results = {}
    for op_name, params in operations.items():
        results[op_name] = await toolkit.execute(op_name, params)
    return results
```

## 故障排除

### 常见问题

**权限错误**
- 检查工具权限配置
- 验证执行上下文

**执行超时**
- 调整超时设置
- 优化工具执行逻辑

**工具未找到**
- 验证工具注册是否正确
- 检查工具名称拼写

工具组件为智能体提供强大而安全的操作能力，是构建复杂智能体的基础。