# Tools Component

The Tools component provides an extensible interface for agent actions, allowing agents to interact with files, execute code, run commands, and perform domain-specific operations in a safe and controlled manner.

## Architecture

Tools follow a unified interface pattern with built-in safety features and context management. All tools inherit from the base tool classes and can be dynamically registered and discovered.

## Core Classes

### BaseTool (`base_tool.py`)
Abstract base class defining the tool interface:

```python
from agentsdk.tools import BaseTool

class MyCustomTool(BaseTool):
    name = "my_tool"
    description = "Description of what this tool does"
    
    async def execute(self, context: ToolContext, **kwargs) -> ToolResponse:
        # Tool implementation
        pass
```

### ToolContext (`tool_context.py`)
Provides execution context for tools:

```python
@dataclass
class ToolContext:
    workspace_path: str
    logger: Logger
    memory: Optional[MemoryInterface]
    request_id: str
    user_data: Dict[str, Any] = field(default_factory=dict)
```

### ToolResponse (`tool_response.py`)
Structured response from tool execution:

```python
@dataclass
class ToolResponse:
    success: bool
    data: Any
    error_message: Optional[str] = None
    metadata: Dict[str, Any] = field(default_factory=dict)
```

## Built-in Tools

### File Operations
- **ReadTool**: Read files from the filesystem
- **WriteTool**: Write content to files  
- **ListFilesTool**: List directory contents

### Code Execution
- **ExecuteCodeTool**: Execute Python code in a controlled environment
- **ShellTool**: Run shell commands with sandboxing

### Agent Operations
- **AgentTool**: Interface for agent-to-agent communication
- **TodoReadTool**: Read and parse TODO files
- **TodoWriteTool**: Write TODO files

## Configuration

### Tool Registration
```python
from agentsdk.tools import Toolkit

# Create toolkit instance
toolkit = Toolkit()

# Register built-in tools
toolkit.register_tool(ReadTool())
toolkit.register_tool(WriteTool())
toolkit.register_tool(ExecuteCodeTool())

# Register custom tool
toolkit.register_tool(MyCustomTool())
```

### Tool Security
```yaml
tools:
  security:
    allow_shell_commands: false  # Enable/disable shell access
    allowed_directories: ["./workspace"]  # Restricted paths
    max_execution_time: 30  # Seconds
```

## Usage Examples

### Basic Tool Usage
```python
# Initialize toolkit
toolkit = Toolkit()
toolkit.register_default_tools()

# Execute a tool
context = ToolContext(workspace_path="./workspace", logger=logger)
response = await toolkit.execute_tool("read_file", context, file_path="data.txt")

if response.success:
    content = response.data
    print(f"File content: {content}")
```

### File Operations
```python
# Read file
response = await toolkit.execute_tool(
    "read_file", 
    context, 
    file_path="config.yaml"
)

# Write file  
response = await toolkit.execute_tool(
    "write_file",
    context,
    file_path="output.py",
    content="print('Hello World')"
)
```

### Code Execution
```python
# Execute Python code
response = await toolkit.execute_tool(
    "execute_code",
    context,
    code="""
def calculate():
    return 2 + 2
    
result = calculate()
print(result)
"""
)
```

## Advanced Features

### Tool Chaining
```python
# Chain multiple tools together
read_response = await toolkit.execute_tool("read_file", context, file_path="input.txt")
if read_response.success:
    content = read_response.data
    processed = content.upper()
    
    write_response = await toolkit.execute_tool(
        "write_file", 
        context, 
        file_path="output.txt", 
        content=processed
    )
```

### Custom Tool Development
```python
from agentsdk.tools import BaseTool, ToolContext, ToolResponse

class DataProcessorTool(BaseTool):
    name = "process_data"
    description = "Process data with custom algorithms"
    
    async def execute(self, context: ToolContext, data: str, algorithm: str = "default") -> ToolResponse:
        try:
            # Custom processing logic
            if algorithm == "uppercase":
                result = data.upper()
            elif algorithm == "lowercase":
                result = data.lower()
            else:
                result = data
                
            return ToolResponse(success=True, data=result)
        except Exception as e:
            return ToolResponse(success=False, error_message=str(e))
```

## Best Practices

### Security Considerations
1. **Sandboxing**: Always use tool context with restricted permissions
2. **Validation**: Validate all inputs before processing
3. **Timeout**: Set reasonable execution timeouts
4. **Logging**: Log all tool executions for audit trails

### Performance Optimization
1. **Tool Caching**: Cache tool instances for repeated use
2. **Context Reuse**: Reuse ToolContext objects when possible
3. **Batch Operations**: Use batch processing for multiple operations

### Error Handling
```python
async def safe_tool_execution(toolkit, context, tool_name, **kwargs):
    try:
        response = await toolkit.execute_tool(tool_name, context, **kwargs)
        if not response.success:
            logger.error(f"Tool {tool_name} failed: {response.error_message}")
            # Fallback logic or retry
        return response
    except Exception as e:
        logger.error(f"Tool execution error: {e}")
        return ToolResponse(success=False, error_message=str(e))
```

## Integration Patterns

### With Memory Component
```python
# Use memory to store tool execution history
async def execute_with_memory(toolkit, context, tool_name, **kwargs):
    response = await toolkit.execute_tool(tool_name, context, **kwargs)
    
    # Store in memory
    if context.memory:
        await context.memory.add_tool_execution(
            tool_name=tool_name,
            parameters=kwargs,
            result=response.data if response.success else None,
            error=response.error_message
        )
    
    return response
```

### With Logger Component
```python
# Enhanced logging for tool executions
class LoggingToolkit(Toolkit):
    async def execute_tool(self, tool_name, context, **kwargs):
        context.logger.info(f"Executing tool: {tool_name} with params: {kwargs}")
        
        start_time = time.time()
        response = await super().execute_tool(tool_name, context, **kwargs)
        duration = time.time() - start_time
        
        context.logger.info(
            f"Tool {tool_name} completed in {duration:.2f}s - "
            f"Success: {response.success}"
        )
        
        return response
```

This modular approach allows for flexible tool development while maintaining security and performance standards.