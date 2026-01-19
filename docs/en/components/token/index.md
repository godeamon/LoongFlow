# Token Component

The Token component provides token management, counting, and budgeting capabilities for LLM interactions within LoongFlow. It ensures efficient token usage and cost control across agent operations.

## Architecture

### Token Counting Layer
- **Multi-Provider Support**: Different tokenization schemes for various LLM providers
- **Accurate Counting**: Precise token counting for prompts and responses
- **Cost Estimation**: Real-time cost estimation based on token usage
- **Budget Enforcement**: Token budget management and enforcement

### Budget Management
- **Usage Tracking**: Track token usage across different operations
- **Budget Allocation**: Allocate tokens to different components and tasks
- **Threshold Monitoring**: Monitor and alert on budget thresholds
- **Cost Optimization**: Strategies for minimizing token usage

## Core Classes

### BaseTokenCounter (`base.py`)
Abstract base class for token counting:

```python
from agentsdk.token import BaseTokenCounter

class CustomTokenCounter(BaseTokenCounter):
    def count_tokens(self, text: str) -> int:
        # Implement custom token counting logic
        pass
```

### SimpleTokenCounter (`simple.py`)
Basic token counter using simple heuristics:

```python
from agentsdk.token import SimpleTokenCounter

counter = SimpleTokenCounter()
token_count = counter.count_tokens("Hello, world!")
```

### TokenBudget (`base.py`)
Token budget management:

```python
from agentsdk.token import TokenBudget

budget = TokenBudget(total_tokens=10000)
remaining = budget.get_remaining()
```

## Supported Tokenization Methods

### OpenAI tiktoken
```python
# Uses OpenAI's tiktoken library for accurate counting
from agentsdk.token import OpenAITokenCounter

counter = OpenAITokenCounter(model="gpt-4")
tokens = counter.count_tokens("Your text here")
```

### Hugging Face Tokenizers
```python
# Support for Hugging Face tokenizers
from agentsdk.token import HuggingFaceTokenCounter

counter = HuggingFaceTokenCounter(model_name="gpt2")
tokens = counter.count_tokens("Your text here")
```

### Custom Tokenizers
```python
# Custom tokenization logic
class CustomTokenCounter(BaseTokenCounter):
    def __init__(self, vocabulary: List[str]):
        self.vocabulary = vocabulary
    
    def count_tokens(self, text: str) -> int:
        # Custom tokenization logic
        words = text.split()
        return len([w for w in words if w in self.vocabulary])
```

## Configuration

### Token Counter Configuration
```yaml
token:
  # Default token counter
  default_counter: "openai"
  
  # Provider-specific settings
  openai:
    model: "gpt-4"
    encoding: "cl100k_base"
    
  huggingface:
    model_name: "gpt2"
    tokenizer_path: "./tokenizers/gpt2"
    
  # Budget settings
  budget:
    total_tokens: 1000000
    warning_threshold: 0.8  # 80% usage
    enforcement: "soft"     # or "hard"
```

### Component-Level Budgets
```yaml
token_budgets:
  planner:
    monthly_tokens: 100000
    per_request: 1000
    
  executor:
    monthly_tokens: 500000
    per_request: 5000
    
  summary:
    monthly_tokens: 200000
    per_request: 2000
```

## Usage Examples

### Basic Token Counting
```python
from agentsdk.token import get_token_counter

# Get default token counter
counter = get_token_counter()

# Count tokens in text
text = "The quick brown fox jumps over the lazy dog"
token_count = counter.count_tokens(text)
print(f"Token count: {token_count}")

# Count tokens in messages
messages = [
    {"role": "user", "content": "Hello!"},
    {"role": "assistant", "content": "Hi there!"}
]
message_tokens = counter.count_message_tokens(messages)
```

### Budget Management
```python
from agentsdk.token import TokenBudget

# Create budget with limits
budget = TokenBudget(
    total_tokens=10000,
    budget_id="monthly_budget"
)

# Check usage
usage = budget.get_usage()
remaining = budget.get_remaining()
print(f"Used: {usage}, Remaining: {remaining}")

# Consume tokens
if budget.can_consume(500):
    budget.consume(500)
    print("Tokens consumed successfully")
else:
    print("Budget exceeded!")
```

### Cost Estimation
```python
from agentsdk.token import CostEstimator

estimator = CostEstimator(
    input_cost_per_token=0.00001,  # $0.01 per 1K tokens input
    output_cost_per_token=0.00003   # $0.03 per 1K tokens output
)

# Estimate cost for a request
estimated_cost = estimator.estimate_cost(
    prompt_tokens=1000,
    completion_tokens=500
)
print(f"Estimated cost: ${estimated_cost:.4f}")
```

## Advanced Features

### Token Optimization
```python
class TokenOptimizer:
    def __init__(self, token_counter):
        self.token_counter = token_counter
    
    def optimize_prompt(self, prompt, max_tokens):
        """Optimize prompt to fit within token limits"""
        tokens = self.token_counter.count_tokens(prompt)
        
        if tokens <= max_tokens:
            return prompt
        
        # Implement optimization strategies
        return self._truncate_or_summarize(prompt, max_tokens)
```

### Multi-Level Budgeting
```python
class HierarchicalBudget:
    def __init__(self, budgets):
        self.budgets = budgets  # Dict of budget levels
    
    def can_consume(self, amount, level="default"):
        budget = self.budgets.get(level)
        return budget.can_consume(amount) if budget else True
    
    def consume(self, amount, level="default"):
        budget = self.budgets.get(level)
        if budget:
            budget.consume(amount)
```

### Token Usage Analytics
```python
class TokenAnalytics:
    def __init__(self, storage_backend):
        self.storage = storage_backend
    
    def record_usage(self, operation, tokens, cost, metadata):
        record = {
            "timestamp": datetime.utcnow(),
            "operation": operation,
            "tokens": tokens,
            "cost": cost,
            "metadata": metadata
        }
        self.storage.save(record)
    
    def get_usage_report(self, start_date, end_date):
        return self.storage.query(start_date, end_date)
```

## Integration Patterns

### With Models Component
```python
# Token-aware model wrapper
class TokenAwareModel:
    def __init__(self, model, token_counter, budget):
        self.model = model
        self.token_counter = token_counter
        self.budget = budget
    
    async def generate(self, request):
        # Count tokens in request
        prompt_tokens = self.token_counter.count_message_tokens(request.messages)
        
        # Check budget
        if not self.budget.can_consume(prompt_tokens):
            raise BudgetExceededError("Prompt tokens exceed budget")
        
        # Generate response
        response = await self.model.generate(request)
        
        # Count completion tokens and update budget
        completion_tokens = self.token_counter.count_tokens(response.content)
        total_tokens = prompt_tokens + completion_tokens
        
        self.budget.consume(total_tokens)
        
        # Add token usage to response
        response.usage = {
            "prompt_tokens": prompt_tokens,
            "completion_tokens": completion_tokens,
            "total_tokens": total_tokens
        }
        
        return response
```

### With Logger Component
```python
# Token usage logging
class TokenUsageLogger:
    def __init__(self, logger, token_counter):
        self.logger = logger
        self.token_counter = token_counter
    
    def log_token_usage(self, operation, text, level="INFO"):
        tokens = self.token_counter.count_tokens(text)
        
        self.logger.log(
            level,
            f"Token usage for {operation}",
            extra={
                "operation": operation,
                "text_length": len(text),
                "token_count": tokens,
                "tokens_per_char": tokens / len(text) if text else 0
            }
        )
```

## Best Practices

### Efficient Token Usage
```python
# Strategies for reducing token usage
def optimize_conversation(messages, max_context_tokens):
    """Optimize conversation history to fit context window"""
    total_tokens = sum(count_message_tokens(msg) for msg in messages)
    
    if total_tokens <= max_context_tokens:
        return messages
    
    # Remove oldest messages first
    while total_tokens > max_context_tokens and len(messages) > 1:
        removed = messages.pop(0)
        total_tokens -= count_message_tokens(removed)
    
    return messages
```

### Budget Management Strategies
```python
class AdaptiveBudget:
    def __init__(self, base_budget, learning_rate=0.1):
        self.base_budget = base_budget
        self.learning_rate = learning_rate
        self.usage_patterns = []
    
    def update_based_on_patterns(self):
        if not self.usage_patterns:
            return
        
        # Analyze usage patterns and adjust budget
        avg_usage = sum(self.usage_patterns) / len(self.usage_patterns)
        adjustment = (avg_usage - self.base_budget.total_tokens) * self.learning_rate
        
        self.base_budget.total_tokens += adjustment
```

### Error Handling
```python
async def generate_with_budget_fallback(model, request, budgets):
    """Try multiple budgets before failing"""
    for budget in budgets:
        if budget.can_consume(estimated_tokens):
            try:
                return await model.generate(request)
            except BudgetExceededError:
                continue
    
    raise BudgetExceededError("All budgets exhausted")
```

## Troubleshooting

### Common Issues

**Token Counting Inaccuracy**
- Verify tokenizer compatibility with target model
- Test with known examples to validate counts
- Consider provider-specific tokenizers for accuracy

**Budget Exceeded Errors**
- Implement graceful degradation strategies
- Add budget monitoring and alerts
- Consider dynamic budget allocation

**Performance Issues**
- Cache token counts for repeated text
- Use approximate counting for large volumes
- Implement batch token counting

The Token component provides essential token management capabilities for cost-effective and efficient LLM operations within the LoongFlow framework.