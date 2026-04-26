# Labs 动手实践环境配置

> 每个 notebook 对应一个考试域。当前版本按官方 Task 拆细：每个 Task 都有一段讲解和一个独立代码示例。

---

## 环境配置

### 1. 安装依赖

```bash
pip install jupyter
```

### 2. 设置 API Key（可选）

当前示例默认使用本地模拟代码，不需要真实 API Key。若你要把示例扩展成真实 Claude API 调用，可在项目根目录创建 `.env` 文件：

```
ANTHROPIC_API_KEY=your_api_key_here
```

或通过环境变量设置：

```bash
export ANTHROPIC_API_KEY="your_api_key_here"
```

### 3. 启动 Jupyter

```bash
jupyter notebook
```

---

## Notebook 列表

| 文件 | 对应 Domain | 核心示例 |
|------|------------|---------|
| [01-agentic-architecture.ipynb](01-agentic-architecture.ipynb) | D1 (27%) | 7 个 Task：智能体循环、多智能体、Hooks、会话管理 |
| [02-tool-design-mcp.ipynb](02-tool-design-mcp.ipynb) | D2 (18%) | 5 个 Task：工具接口、MCP 错误、tool_choice、内置工具 |
| [03-claude-code.ipynb](03-claude-code.ipynb) | D3 (20%) | 5 个 Task：CLAUDE.md、命令、规则、CLI 模式、Skill |
| [04-prompt-engineering.ipynb](04-prompt-engineering.ipynb) | D4 (20%) | 6 个 Task：明确标准、Few-shot、Schema、验证重试、批处理、多轮审查 |
| [05-context-management.ipynb](05-context-management.ipynb) | D5 (15%) | 6 个 Task：上下文保留、升级、错误传播、代码库探索、人工审核、来源链 |

---

## 费用说明

所有示例默认不调用外部 API，因此完整运行 notebooks 不产生 API 费用。
