# Models Component

The Models component provides a unified interface for Large Language Model (LLM) integration, supporting multiple LLM providers and enabling consistent model interactions across the LoongFlow framework.

## Architecture

### Provider Abstraction Layer
- **Unified Interface**: Consistent API for different LLM providers
- **Provider Plugins**: Support for OpenAI, Anthropic, Google, DeepSeek, etc.
- **Fallback Mechanisms**: Automatic provider fallback on failures
- **Rate Limiting**: Configurable rate limiting and retry logic

### Request/Response Handling
- **Structured Requests**: Standardized request format across providers
- **Response Parsing**: Consistent response parsing and error handling
- **Tool Integration**: Support for tool calling and function calling
- **Streaming**: Real-time streaming response support

## Core Classes

### BaseLLMModel (`base_llm_model.py`)
Abstract base class defining the LLM interface:

```python
from agentsdk.models import BaseLLMModel

class CustomLLM(BaseLLMModel):
    async def generate(self, request: CompletionRequest) -> CompletionResponse:
        # Implement provider-specific generation logic
        pass
```

### CompletionRequest (`llm_request.py`)
Structured request format:

```python
from agentsdk.models import CompletionRequest

request = CompletionRequest(
    messages=[
        {"role": "system", "content": "You are a helpful assistant"},
        {"role": "user", "content": "Hello, world!"}
    ],
    model="gpt-4",
    temperature=0.7,
    max_tokens=1000,
    tools=[...]  # Optional tool specifications
)
```

### CompletionResponse (`llm_response.py`)
Structured response format:

```python
from agentsdk.models import CompletionResponse

response = CompletionResponse(
    content="Hello! How can I help you?",
    model="gpt-4",
    usage={"prompt_tokens": 10, "completion_tokens": 5},
    finish_reason="stop"
)
```

## Supported Providers

### OpenAI/Compatible APIs
```python
from agentsdk.models import LiteLLMModel

model = LiteLLMModel(
    api_key="your-api-key",
    base_url="https://api.openai.com/v1",
    model="gpt-4"
)
```

### Anthropic Claude
```python
model = LiteLLMModel(
    api_key="claude-api-key",
    base_url="https://api.anthropic.com",
    model="claude-3-sonnet-20240229"
)
```

### Google Gemini
```python
model = LiteLLMModel(
    api_key="gemini-api-key", 
    base_url="https://generativelanguage.googleapis.com",
    model="gemini-pro"
)
```

### DeepSeek
```python
model = LiteLLMModel(
    api_key="deepseek-api-key",
    base_url="https://api.deepseek.com/v1",
    model="deepseek-chat"
)
```

## Configuration

### Provider Configuration
```yaml
llm_config:
  # Provider settings
  provider: "openai"  # or "anthropic", "google", "deepseek"
  api_key: "your-api-key"
  base_url: "https://api.openai.com/v1"
  model: "gpt-4"
  
  # Generation parameters
  temperature: 0.7
  max_tokens: 4000
  top_p: 1.0
  frequency_penalty: 0.0
  presence_penalty: 0.0
  
  # Advanced settings
  timeout: 30
  max_retries: 3
  retry_delay: 1.0
```

### Multi-Provider Configuration
```yaml
llm_providers:
  primary:
    provider: "openai"
    api_key: "openai-key"
    model: "gpt-4"
    
  fallback:
    provider: "anthropic" 
    api_key: "claude-key"
    model: "claude-3-haiku"
    
  budget:
    monthly_limit: 1000  # USD
    warning_threshold: 0.8
```

## Usage Examples

### Basic Text Generation
```python
from agentsdk.models import LiteLLMModel, CompletionRequest

# Initialize model
model = LiteLLMModel.from_config(llm_config)

# Create request
request = CompletionRequest(
    messages=[
        {"role": "user", "content": "Explain quantum computing"}
    ],
    model="gpt-4",
    max_tokens=500
)

# Generate response
response = await model.generate(request)
print(response.content)
```

### Tool Calling
```python
# Define tools
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"}
                }
            }
        }
    }
]

# Request with tools
request = CompletionRequest(
    messages=[{"role": "user", "content": "What's the weather in Tokyo?"}],
    tools=tools,
    tool_choice="auto"
)

response = await model.generate(request)

# Check for tool calls
if response.tool_calls:
    for tool_call in response.tool_calls:
        function_name = tool_call.function.name
        arguments = json.loads(tool_call.function.arguments)
        # Execute the tool...
```

### Streaming Responses
```python
async for chunk in model.generate_stream(request):
    if chunk.content:
        print(chunk.content, end="", flush=True)
    
    if chunk.finish_reason:
        print(f"\nFinished: {chunk.finish_reason}")
```

## Advanced Features

### Response Formatting
```python
from agentsdk.models.formatter import LiteLLMFormatter

formatter = LiteLLMFormatter()

# Convert LoongFlow messages to provider format
provider_messages = formatter.convert_messages(request.messages)

# Parse provider response back to LoongFlow format
response = formatter.parse_response(provider_response)
```

### Custom Provider Implementation
```python
from agentsdk.models import BaseLLMModel

class CustomProviderModel(BaseLLMModel):
    def __init__(self, custom_endpoint, auth_token):
        self.endpoint = custom_endpoint
        self.auth_token = auth_token
    
    async def generate(self, request):
        # Implement custom API call
        payload = self._prepare_payload(request)
        response = await self._call_api(payload)
        return self._parse_response(response)
```

### Cost Tracking
```python
class CostAwareModel(BaseLLMModel):
    def __init__(self, wrapped_model, cost_tracker):
        self.wrapped_model = wrapped_model
        self.cost_tracker = cost_tracker
    
    async def generate(self, request):
        response = await self.wrapped_model.generate(request)
        
        # Track cost
        cost = self._calculate_cost(request, response)
        self.cost_tracker.record_cost(cost)
        
        return response
```

## Integration Patterns

### With Tools Component
```python
# Model-driven tool execution
class ModelToolOrchestrator:
    def __init__(self, model, toolkit):
        self.model = model
        self.toolkit = toolkit
    
    async def execute_with_tools(self, query):
        # Get tool specifications
        tools = self.toolkit.get_tool_specifications()
        
        # Generate plan with tools
        request = CompletionRequest(
            messages=[{"role": "user", "content": query}],
            tools=tools
        )
        
        response = await self.model.generate(request)
        
        # Execute tool calls
        if response.tool_calls:
            return await self._execute_tool_calls(response.tool_calls)
        
        return response.content
```

### With Memory Component
```python
# Context-aware generation with memory
class ContextAwareModel:
    def __init__(self, model, memory):
        self.model = model
        self.memory = memory
    
    async def generate_with_context(self, query, context_id):
        # Retrieve conversation history
        history = await self.memory.get_conversation_history(context_id)
        
        # Build messages with context
        messages = history + [{"role": "user", "content": query}]
        
        request = CompletionRequest(messages=messages)
        return await self.model.generate(request)
```

## Best Practices

### Error Handling
```python
async def safe_generate(model, request, max_retries=3):
    for attempt in range(max_retries):
        try:
            return await model.generate(request)
        except RateLimitError:
            await asyncio.sleep(2 ** attempt)  # Exponential backoff
        except APIError as e:
            if attempt == max_retries - 1:
                raise
            logger.warning(f"API error (attempt {attempt+1}): {e}")
    
    raise Exception("Max retries exceeded")
```

### Performance Optimization
```python
# Batch processing for multiple requests
async def batch_generate(model, requests):
    # Process requests in parallel
    tasks = [model.generate(req) for req in requests]
    return await asyncio.gather(*tasks, return_exceptions=True)
```

### Security Considerations
```python
# Input validation and sanitization
class SafeModel:
    async def generate(self, request):
        # Validate input
        self._validate_input(request.messages)
        
        # Sanitize sensitive data
        sanitized_messages = self._sanitize_messages(request.messages)
        
        safe_request = request.copy(update={"messages": sanitized_messages})
        return await self.wrapped_model.generate(safe_request)
```

## Troubleshooting

### Common Issues

**Rate Limiting**
- Implement exponential backoff
- Monitor usage and adjust rate limits
- Consider provider switching for high-volume usage

**Token Limits**
- Monitor token usage in responses
- Implement chunking for long conversations
- Use summarization for context management

**Provider Compatibility**
- Test with different provider configurations
- Implement fallback mechanisms
- Monitor provider status and updates

The Models component provides a robust foundation for LLM integration with support for multiple providers, tool calling, and advanced features needed for agent development.