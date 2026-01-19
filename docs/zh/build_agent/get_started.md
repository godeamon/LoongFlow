# 准备工作

开始使用LoongFlow构建智能体前的准备工作。

## 环境要求

### Python版本
```bash
python --version  # 需要 Python 3.12+
```

### 包管理
推荐使用 `uv` 进行依赖管理：
```bash
uv --version  # 确认uv已安装
```

## 快速设置

### 1. 创建虚拟环境
```bash
uv venv .venv --python 3.12
source .venv/bin/activate
```

### 2. 安装依赖
```bash
uv pip install -e .
```

### 3. 配置LLM
创建 `task_config.yaml`：
```yaml
llm_config:
  url: "https://your-llm-api/v1"
  api_key: "your-api-key"
  model: "deepseek-r1-250528"
```

## 验证安装

### 运行测试
```bash
uv run pytest tests/ -v
```

### 检查组件
```bash
python -c "from agentsdk import tools, memory, logger; print('Components loaded successfully')"
```

## 下一步

选择适合的构建范式开始开发：
- [通用进化智能体](general_evolve.md)
- [机器学习智能体](ml_evolve.md)
- [自定义智能体开发](custom_guide.md)