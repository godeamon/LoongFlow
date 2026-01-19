# Message Component

The Message component provides structured message passing and content management for agent communication within LoongFlow. It enables rich content types, serialization, and efficient message routing.

## Architecture

### Message Flow
- **Agent-to-Agent**: Communication between different agent components
- **Component-to-Component**: Internal component communication  
- **External Communication**: Interface with external systems
- **Persistence**: Message storage and retrieval

### Content Types
- **Text Messages**: Simple text content
- **Code Messages**: Executable code snippets
- **Data Messages**: Structured data payloads
- **File Messages**: File attachments and references
- **Multi-part Messages**: Combined content types

## Core Classes

### Message (`message.py`)
Base message class with rich content support:

```python
from agentsdk.message import Message

# Create a message
message = Message(
    sender="planner",
    recipient="executor",
    content="Execute this plan",
    message_type="instruction"
)

# Add content elements
message.add_element(TextElement("Plan description"))
message.add_element(CodeElement("def execute_plan(): ..."))
```

### ContentElement (`elements.py`)
Base class for different content types:

```python
from agentsdk.message import ContentElement, TextElement, CodeElement

# Text content
text_elem = TextElement("This is a text message")
text_elem.data = "Updated content"

# Code content  
code_elem = CodeElement("python", "def hello(): print('world')")
```

### MessageFactory (`message.py`)
Factory for creating different message types:

```python
from agentsdk.message import MessageFactory

factory = MessageFactory()
instruction_msg = factory.create_instruction(
    sender="planner",
    content="Please execute this task"
)
```

## Content Element Types

### TextElement
Plain text content with formatting support:

```python
text_elem = TextElement(
    content="Task description",
    format="markdown"  # or "plain", "html"
)
```

### CodeElement
Executable code with language specification:

```python
code_elem = CodeElement(
    language="python",
    code="def solution(): return 42",
    metadata={"function_name": "solution"}
)
```

### DataElement
Structured data payloads:

```python
data_elem = DataElement(
    data_type="json",
    data={"key": "value", "count": 42},
    schema={"type": "object"}
)
```

### FileElement
File references and attachments:

```python
file_elem = FileElement(
    file_path="./data/input.txt",
    mime_type="text/plain",
    size=1024
)
```

## Configuration

### Message System Configuration
```yaml
messaging:
  # Serialization settings
  serialization_format: "json"  # or "pickle", "msgpack"
  compression: true
  max_message_size: "10MB"
  
  # Routing settings
  default_timeout: 30
  retry_attempts: 3
  dead_letter_queue: "./dlq/"
  
  # Validation settings
  validate_schema: true
  required_fields: ["sender", "recipient", "content"]
```

### Content Validation
```yaml
content_validation:
  code_elements:
    allowed_languages: ["python", "javascript", "sql"]
    max_size: "1MB"
    sandbox_execution: true
    
  data_elements:
    allowed_schemas: ["json", "yaml"]
    max_depth: 10
    size_limit: "5MB"
```

## Usage Examples

### Basic Message Creation
```python
from agentsdk.message import Message, TextElement

# Create a simple message
message = Message(
    sender="user",
    recipient="agent",
    content="Please solve this problem",
    metadata={"priority": "high", "category": "math"}
)

# Add rich content
message.add_element(TextElement("Problem description in detail..."))
message.add_element(CodeElement("python", "def initial_solution(): ..."))

# Serialize for transmission
serialized = message.serialize()
```

### Message Routing
```python
from agentsdk.message import MessageRouter

router = MessageRouter()

# Register message handlers
@router.handler("instruction")
async def handle_instruction(message):
    # Process instruction messages
    result = await execute_instruction(message.content)
    return ResponseMessage(content=result)

# Route incoming message
response = await router.route(incoming_message)
```

### Complex Message Scenarios
```python
# Multi-part message with different content types
complex_message = Message(
    sender="analysis_agent",
    recipient="report_agent",
    message_type="analysis_report"
)

# Add text summary
complex_message.add_element(TextElement(
    "Analysis completed with the following findings...",
    format="markdown"
))

# Add data results
complex_message.add_element(DataElement(
    data_type="json",
    data={"metrics": {"accuracy": 0.95, "f1": 0.92}}
))

# Add code for reproduction
complex_message.add_element(CodeElement(
    "python",
    "def reproduce_analysis(): ..."
))
```

## Advanced Features

### Message Serialization
```python
# Custom serialization
class CustomSerializer:
    def serialize(self, message):
        # Custom serialization logic
        return json.dumps(message.to_dict(), cls=CustomEncoder)
    
    def deserialize(self, data):
        return Message.from_dict(json.loads(data))

# Use custom serializer
message.serializer = CustomSerializer()
```

### Message Validation
```python
from agentsdk.message import MessageValidator

validator = MessageValidator()

# Custom validation rules
@validator.rule
def validate_sender(message):
    if not message.sender:
        raise ValidationError("Message must have a sender")

# Validate message
try:
    validator.validate(message)
    print("Message is valid")
except ValidationError as e:
    print(f"Validation failed: {e}")
```

### Message Transformations
```python
# Chain of message transformers
class MessageTransformer:
    def __init__(self):
        self.transformers = [
            ContentCompressor(),
            EncryptionTransformer(),
            MetadataEnricher()
        ]
    
    async def transform(self, message):
        for transformer in self.transformers:
            message = await transformer.transform(message)
        return message
```

## Integration Patterns

### With Memory Component
```python
# Store messages in memory
async def store_conversation(messages, memory):
    for message in messages:
        await memory.add_message(
            message.serialize(),
            metadata={
                "timestamp": message.timestamp,
                "participants": [message.sender, message.recipient]
            }
        )
```

### With Logger Component
```python
# Log message flow
class LoggingMessageHandler:
    async def handle_message(self, message):
        logger.info(
            "Message received",
            extra={
                "sender": message.sender,
                "recipient": message.recipient,
                "type": message.message_type,
                "size": message.size
            }
        )
        
        # Process message
        result = await self.process_message(message)
        
        logger.info("Message processed", extra={"result": result.status})
        return result
```

### With Tools Component
```python
# Message-driven tool execution
class MessageDrivenToolkit:
    async def process_tool_message(self, message):
        if message.message_type == "tool_execution":
            tool_name = message.metadata.get("tool")
            parameters = message.content
            
            result = await self.execute_tool(tool_name, parameters)
            
            # Create response message
            response = Message(
                sender="toolkit",
                recipient=message.sender,
                content=result.data,
                message_type="tool_response"
            )
            
            return response
```

## Best Practices

### Message Design
1. **Clear Purpose**: Each message should have a single, clear purpose
2. **Appropriate Size**: Keep messages focused and reasonably sized
3. **Structured Content**: Use appropriate content elements for different data types
4. **Metadata**: Include relevant metadata for routing and processing

### Performance Considerations
```python
# Efficient message handling
async def process_messages_batch(messages):
    # Process messages in batches for efficiency
    batch_size = 10
    for i in range(0, len(messages), batch_size):
        batch = messages[i:i + batch_size]
        await asyncio.gather(*[process_message(msg) for msg in batch])
```

### Security Practices
```python
# Secure message handling
class SecureMessageProcessor:
    async def process(self, message):
        # Validate message source
        if not self.is_trusted_sender(message.sender):
            raise SecurityError("Untrusted sender")
        
        # Sanitize content
        sanitized_content = self.sanitize_content(message.content)
        message.content = sanitized_content
        
        return await super().process(message)
```

## Troubleshooting

### Common Issues

**Serialization Errors**
- Ensure all message content is serializable
- Use custom serializers for complex objects
- Test serialization/deserialization round trips

**Message Size Limits**
- Implement message chunking for large content
- Use compression for large payloads
- Consider external storage for very large files

**Routing Problems**
- Verify recipient addresses are correct
- Implement dead letter queue for undeliverable messages
- Add message tracing for debugging routing issues

The Message component provides a robust foundation for agent communication with support for rich content types and flexible processing patterns.