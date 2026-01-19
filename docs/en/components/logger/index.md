# Logger Component

The Logger component provides a comprehensive logging system for LoongFlow agents, offering structured logging, context management, and flexible output formats.

## Architecture

### Multi-layer Logging System
- **Application Logs**: High-level agent execution logs
- **Component Logs**: Individual component activity logs  
- **Debug Logs**: Detailed debugging information
- **Error Logs**: Error and exception tracking

### Log Levels
- **DEBUG**: Detailed debugging information
- **INFO**: General operational information
- **WARNING**: Warning conditions
- **ERROR**: Error conditions
- **CRITICAL**: Critical conditions

## Core Classes

### Logger (`logger.py`)
Main logger class providing the logging interface:

```python
from agentsdk.logger import get_logger

# Get logger instance
logger = get_logger("my_component")

# Log at different levels
logger.debug("Detailed debug information")
logger.info("Operation completed successfully")
logger.warning("Potential issue detected")
logger.error("Operation failed", exc_info=True)
```

### ContextLogger (`context.py`)
Context-aware logging with automatic context propagation:

```python
from agentsdk.logger import ContextLogger

# Create context logger
context_logger = ContextLogger(
    component="planner",
    request_id="req_123",
    iteration=5
)

# Log with context
context_logger.info("Planning iteration started")
```

### MessageLogger (`message_logger.py`)
Specialized logging for message processing:

```python
from agentsdk.logger import MessageLogger

message_logger = MessageLogger()
message_logger.log_message_sent(message, recipient)
message_logger.log_message_received(message, sender)
```

## Configuration

### Basic Configuration
```yaml
logging:
  level: "INFO"
  format: "json"  # or "text"
  output: "file"  # or "console", "both"
  
  # File logging
  file_path: "./logs/evolux.log"
  max_file_size: "100MB"
  backup_count: 5
  
  # Console logging
  console_format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
```

### Advanced Configuration
```yaml
logging:
  # Component-specific levels
  loggers:
    agentsdk.memory: "DEBUG"
    agentsdk.tools: "INFO"
    evolux.planner: "WARNING"
  
  # Structured logging
  structured: true
  include_context: true
  include_traceback: true
  
  # Performance settings
  buffer_size: 1000
  flush_interval: 30  # seconds
```

## Usage Examples

### Basic Logging
```python
from agentsdk.logger import get_logger

logger = get_logger(__name__)

class MyComponent:
    async def process(self, data):
        logger.info("Starting data processing", extra={"data_size": len(data)})
        
        try:
            result = await self._complex_operation(data)
            logger.debug("Operation completed", extra={"result": result})
            return result
            
        except Exception as e:
            logger.error("Processing failed", exc_info=True)
            raise
```

### Context-Aware Logging
```python
from agentsdk.logger import ContextLogger

async def process_request(request_id, data):
    # Create context logger for this request
    context_logger = ContextLogger(
        request_id=request_id,
        component="request_processor",
        user_id=data.get('user_id')
    )
    
    context_logger.info("Request processing started")
    
    # All logs in this context will include the request_id
    result = await process_data(data)
    context_logger.info("Request completed", extra={"result": result})
```

### Structured Logging
```python
# JSON-formatted structured logs
logger.info("Evolution iteration completed", extra={
    "iteration": 42,
    "population_size": 100,
    "best_score": 0.95,
    "duration_seconds": 120.5,
    "component": "evolve_agent"
})
```

## Advanced Features

### Custom Log Handlers
```python
import logging
from agentsdk.logger import get_logger

# Create custom handler
class CustomHandler(logging.Handler):
    def emit(self, record):
        # Custom emission logic (e.g., send to external system)
        pass

# Register custom handler
logger = get_logger("custom_component")
logger.addHandler(CustomHandler())
```

### Log Filtering
```python
class ComponentFilter(logging.Filter):
    def filter(self, record):
        # Filter logs based on component or other criteria
        return hasattr(record, 'component') and record.component == 'memory'

logger.addFilter(ComponentFilter())
```

### Performance Monitoring
```python
from agentsdk.logger import performance_logger

@performance_logger
async def expensive_operation(data):
    # This call will be automatically logged with timing information
    result = await do_expensive_work(data)
    return result
```

## Integration Patterns

### With Memory Component
```python
# Log memory operations
async def log_memory_operation(operation, solution_id, success=True):
    logger.info(
        "Memory operation completed",
        extra={
            "operation": operation,
            "solution_id": solution_id,
            "success": success,
            "timestamp": datetime.utcnow().isoformat()
        }
    )
```

### With Tools Component
```python
# Log tool executions
class LoggingToolkit:
    async def execute_tool(self, tool_name, context, **kwargs):
        start_time = time.time()
        
        logger.info("Tool execution started", extra={
            "tool": tool_name,
            "parameters": kwargs
        })
        
        result = await super().execute_tool(tool_name, context, **kwargs)
        
        duration = time.time() - start_time
        logger.info("Tool execution completed", extra={
            "tool": tool_name,
            "duration": duration,
            "success": result.success
        })
        
        return result
```

## Best Practices

### Logging Guidelines
1. **Meaningful Messages**: Include context and relevant data
2. **Appropriate Levels**: Use DEBUG for details, ERROR for failures
3. **Structured Data**: Use extra parameters for machine-readable data
4. **Security**: Avoid logging sensitive information

### Performance Considerations
```python
# Lazy evaluation for expensive log operations
if logger.isEnabledFor(logging.DEBUG):
    # Only compute expensive debug info if DEBUG is enabled
    debug_info = compute_expensive_debug_info()
    logger.debug("Detailed info: %s", debug_info)
```

### Error Handling
```python
# Robust logging in error scenarios
try:
    risky_operation()
except Exception as e:
    logger.error(
        "Operation failed with unexpected error",
        exc_info=True,
        extra={
            "operation": "risky_operation",
            "error_type": type(e).__name__
        }
    )
    # Re-raise or handle appropriately
```

## Troubleshooting

### Common Issues

**Log File Rotation**
- Ensure sufficient disk space for log files
- Configure appropriate max_file_size and backup_count
- Monitor log directory size

**Performance Impact**
- Use appropriate log levels in production
- Consider asynchronous logging for high-throughput scenarios
- Disable debug logging unless needed

**Missing Context**
- Ensure context information is properly passed to logger
- Use ContextLogger for request-scoped logging
- Verify extra parameters are included

The Logger component provides a flexible and powerful logging infrastructure suitable for both development and production environments.