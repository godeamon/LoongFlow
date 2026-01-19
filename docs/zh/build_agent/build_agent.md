# 构建智能体指南

本文档介绍了如何使用LoongFlow框架构建自定义智能体。

## 快速开始

### 环境准备
```bash
# 创建虚拟环境
uv venv .venv --python 3.12
source .venv/bin/activate

# 安装依赖
uv pip install -e .
```

### 基本配置
创建 `task_config.yaml` 文件：

```yaml
llm_config:
  url: "https://your-llm-api/v1"
  api_key: "your-api-key"
  model: "deepseek-r1-250528"

evolve:
  max_iterations: 100
  target_score: 0.95
```

## 智能体范式选择

### 通用进化范式
适合数学优化、算法问题：
- 使用进化算法和PES循环
- 需要清晰的评估指标
- 支持复杂优化场景

### ML进化范式
适合机器学习任务：
- 专为ML竞赛和AutoML设计
- 内置MLE-Bench支持
- 自动化ML管道生成

## 下一步

查看具体范式文档了解详细实现方法：
- [通用进化范式](general_evolve.md)
- [ML进化范式](ml_evolve.md)
- [自定义开发指南](custom_guide.md)