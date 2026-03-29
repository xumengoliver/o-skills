# 模型路由与缓存策略

## 模型能力矩阵

| 模型 | 标识符 | 最擅长 | 降级参考 | 缓存支持 |
|---|---|---|---|---|
| **Claude Opus 4.6** | `claude-opus-4-6` | 深度推理、架构、brainstorm、长上下文(200K) | → o3 → gpt-5.3-codex | ✅ cache_control, ≥1024t |
| **Claude Sonnet 4.6** | `claude-sonnet-4-6` | 全栈开发、代码生成、API 设计 | → qwen3.5-plus → deepseek-v3 | ✅ |
| **Claude Haiku 4.5** | `claude-haiku-4-5` | 快速分类、格式转换、高频调用 | → qwen3.5-plus → deepseek-v3 | ✅ |
| **GPT-5.3-Codex** | `gpt-5.3-codex` | 前端 UI/UX、视觉理解、多模态 | → claude-sonnet-4-6 → qwen3.5-plus | ✅ |
| **o3** | `o3` | 数学、算法、深度逻辑推理、约束求解 | → claude-opus-4-6 → deepseek-r1 | ✅ |
| **Gemini 3.1 Pro** | `gemini-3.1-pro` | 超长上下文(2M)、代码库全局分析 | → claude-opus-4-6 → kimi-k2.5 | ✅ |
| **DeepSeek R1** | `deepseek-r1` | 后端架构、数据库、中文技术文档、低成本推理 | → claude-sonnet-4-6 → qwen3.5-plus | ✅ |
| **DeepSeek V3** | `deepseek-v3` | 中文后端、低成本高质量、SQL 优化 | → deepseek-r1 → claude-sonnet-4-6 | ✅ |
| **GLM-5** | `glm-5` | 中文对话、工具调用、企业知识库 | → glm-4-plus → claude-sonnet-4-6 | ❌ |
| **MiniMax M2.5** | `minimax-m2.5` | 长上下文(100万)、中文创作、多轮对话 | → abab6.5s-chat → gemini-3.1-pro | ❌ |
| **Kimi K2.5** | `kimi-k2.5` | 超长文档(128K)、中文阅读理解、PDF 解析 | → moonshot-v1-128k → gemini-3.1-pro | ❌ |
| **Qwen3.5-Plus** | `qwen3.5-plus` | 中文代码、数学推理、指令跟随、全栈 | → qwen-max → claude-sonnet-4-6 | ❌ |

## 任务类型 → 推荐模型

| 任务类型 | 首选 | 备选 |
|---|---|---|
| 界面设计 / UI 原型 | gpt-5.4 | claude-sonnet-4-6 |
| 前端开发 / UI 实现 | gpt-5.3-codex | claude-sonnet-4-6 |
| 后端架构 / API 设计（国际） | deepseek-r1 | claude-sonnet-4-6 |
| 后端架构 / API 设计（国内） | qwen3.5-plus | deepseek-v3 |
| Brainstorm / 创意探索 | claude-opus-4-6 | — |
| 深度推理 / 算法设计 | o3 | claude-opus-4-6 |
| 长上下文分析（国际） | gemini-3.1-pro | claude-opus-4-6 |
| 长上下文分析（国内文档） | kimi-k2.5 | minimax-m2.5 |
| 快速分类 / 格式转换 | claude-haiku-4-5 | — |
| 文档撰写（中文） | glm-5 | qwen3.5-plus |
| 文档撰写（英文） | claude-sonnet-4-6 | — |
| 安全审计 / 约束检查 | o3 | claude-opus-4-6 |
| 中文代码 / SQL 优化 | qwen3.5-plus | deepseek-v3 |

## 模型选择规则

1. 同一团队中**允许混用**不同模型（含国内+国际）
2. 成本敏感时，简单任务优先用 `claude-haiku-4-5`
3. **每个 Agent 必须声明降级链**（主模型 → 备用 → 兜底，至少 3 个候选）
4. 降级触发条件：模型不存在 / API Key 无效 / 访问受限（429/403）

## Context Caching 策略

### 缓存决策规则

1. `backstory` + 项目上下文 ≥ 1024 tokens，且调用 ≥ 2 次 → `cache_strategy: "prefix"`
2. 不支持缓存的 provider（见模型矩阵 ❌ 列）→ `cache_strategy: "none"`
3. Phase 1 的 3 个评审 Agent 共享设计文档 → **默认启用**
4. Anthropic 用 `cache_control` 标记 / OpenAI+DeepSeek 自动前缀匹配 / Gemini 用 CachedContent API

### Provider 缓存能力

```python
CACHE_CAPABLE_PROVIDERS = {
    "anthropic": {"method": "prompt_caching", "min_tokens": 1024},
    "openai":    {"method": "auto_prefix",    "min_tokens": 1024},
    "google":    {"method": "cached_content",  "min_tokens": 32768},
    "deepseek":  {"method": "auto_prefix",    "min_tokens": 64},
}
```
