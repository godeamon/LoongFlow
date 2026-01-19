# Getting Started with Building Agents

Welcome to LoongFlow! This guide will help you understand the two main paradigms for building agents and choose the right approach for your use case.

## 🎯 Choose Your Paradigm

LoongFlow offers two distinct approaches to agent development:

### General Evolve Paradigm
**Best for**: Mathematical optimization, algorithmic problems, general AI tasks
- Uses evolutionary algorithms with Plan-Execute-Summary cycles
- Ideal for problems with clear evaluation metrics
- Supports complex optimization landscapes

### ML Evolve Paradigm  
**Best for**: Machine learning competitions, AutoML, Kaggle-style problems
- Specialized for ML/AI data science workflows
- Built-in support for MLE-Bench and Kaggle competitions
- Automated ML pipeline generation

## 📋 Prerequisites

### Environment Setup
```bash
# Create virtual environment
uv venv .venv --python 3.12
source .venv/bin/activate

# Install dependencies
uv pip install -e .
```

### LLM Configuration
You'll need access to an LLM API (OpenAI, Gemini, DeepSeek, etc.). Create a `task_config.yaml`:

```yaml
llm_config:
  url: "https://your-llm-api/v1"
  api_key: "your-api-key-here"
  model: "deepseek-r1-250528"  # or your preferred model
```

## 🚀 Quick Start Examples

### General Evolve - Math Problem
```bash
# Navigate to math example
cd agents/general_evolve/examples/math_flip

# Install dependencies
uv pip install -r requirements.txt

# Run the agent
python general_evolve_agent.py \
    --config task_config.yaml \
    --task-file description.md \
    --eval-file evaluator.py \
    --max-iterations 100
```

### ML Evolve - Classification Task
```bash
# Navigate to ML example  
cd agents/ml_evolve/examples/ml_example

# Initialize environment
./run_ml.sh init

# Run ML agent
./run_ml.sh run ml_example --background

# Monitor progress
tail -f output/logs/evolux.log
```

## 🔧 Key Configuration Files

### Task Configuration (`task_config.yaml`)
Controls the evolution process:
- LLM settings and model parameters
- Evolution parameters (iterations, population size)
- Component configurations

### Evaluation Code (`evaluator.py`)
Defines how solutions are scored:
- Python script with `evaluate()` function
- Returns score between 0.0 and 1.0
- Critical for guiding the evolutionary process

### Task Description (`description.md`)
Describes the problem to be solved:
- Clear problem statement
- Expected input/output formats
- Constraints and requirements

## 📊 Monitoring Progress

### Logs
- Check `output/logs/` for execution logs
- Monitor evolution progress and errors

### Visualization
```bash
# Start visualization server
cd visualizer
python visualizer.py --port 8888

# Open browser to http://localhost:8888
```

### Checkpoints
- Automatic checkpointing every few iterations
- Resume from interruptions using `--checkpoint-path`

## 🛠️ Next Steps

1. **Explore Examples**: Study the provided examples in `agents/*/examples/`
2. **Customize**: Modify configurations for your specific needs
3. **Extend**: Create custom evaluation functions and tools
4. **Scale**: Configure for distributed execution if needed

Choose the paradigm that best fits your problem domain and follow the specific guides for detailed implementation.
