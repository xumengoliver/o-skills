# Phase 详细规格

## Phase 1: 结构化评审

### 步骤 1 — Primary Designer 出方案

Designer 必须同时完成**业务设计**和**执行团队设计**，包含以下 6 个必须模块：

```
## 1. 目标定义
- 最终交付物、成功标准、约束条件（时间、技术栈、预算）

## 2. Agent 团队设计
为每个 Agent 提供：
- role: 精确角色名称（如 "高级市场研究分析师"，不要泛称 "研究员"）
- goal: 具体目标
- backstory: 角色背景，影响行为风格和专业深度
- tools: 需要的工具
- model: 推荐模型标识符（见模型路由章节）
- model_reason: 2句话说明选型理由
- cache_strategy: "prefix" | "none"
- cache_reason: 1句话说明理由

## 3. Task 分解
为每个 Task 提供：
- description: 详细描述含具体步骤
- agent: 分配给哪个 Agent
- expected_output: 精确的输出格式和内容要求
- context: 依赖哪些前置任务的输出

## 4. 流程类型决策
- Sequential / Hierarchical / Parallel / 混合？理由？
- 哪些任务有依赖必须顺序？哪些可并行？

## 5. 风险预案
- 单个 Agent 失败时的降级策略
- 输出质量不达标时的重试策略

## 6. 评估策略声明
- eval_framework: "deepeval" | "ragas" | "both"
- eval_model: 用于 LLM-as-Judge 的模型
- eval_gate: "hard" | "soft" | "report_only"
```

### 步骤 2 — 并行评审（3 Agent 同时派发）

#### Skeptic（怀疑者）— 假设方案会失败
- 职责重叠/空白？依赖链瓶颈/循环？`expected_output` 够清晰？backstory 偏离风险？
- **禁止**：提新功能、重新架构

#### Guardian（约束守护者）— 检查现实约束
- Agent 数量 3-5？Token 合理？工具可行？流程匹配依赖？
- **模型**：能力匹配？昂贵模型分给简单任务？降级链 ≥ 3？
- **缓存**：≥1024t 且多次调用→启用 prefix？
- **KPI**：每步有指标？阈值 0.5-0.9？
- **禁止**：讨论产品目标、加功能

#### Advocate（用户代言人）— 交付物满足用户预期
- 质量标准符合预期？输出风格一致？错误场景覆盖？
- 是否满足可访问性、响应式与可用性要求？
- **禁止**：重新架构、加功能

### 步骤 3 — Designer 汇总修订

每条反馈：接受→修订 | 拒绝→附理由 | 需讨论→交 Arbiter。冲突反馈须在决策日志记录取舍理由。

### 步骤 4 — Arbiter 仲裁

审查最终设计+日志+异议 → **APPROVED**(→P2) / **REVISE**(→步骤2) / **REJECT**(→步骤1)

## Phase 2: 执行规格生成

**仅在 Phase 1 APPROVED 后执行。**

### agents.yaml 模板

```yaml
agent_name:
  role: "精确的角色名称"
  goal: "该 Agent 的具体目标"
  backstory: |
    角色背景描述
  llm: "首选模型标识符"
  llm_fallback:              # 按能力排序，至少 3 个
    - "备用模型1"
    - "备用模型2"
    - "兜底模型"
  cache_config:
    strategy: "prefix"
    cached_prefix_sections: ["system_prompt", "backstory", "project_context"]
    ttl: 300
  tools:
    - ToolName1
  verbose: true
```

### tasks.yaml 模板

```yaml
task_name:
  description: |
    任务详细描述
  agent: agent_name
  expected_output: |
    输出格式描述
  context:
    - dependent_task_name

# 并行任务
parallel_task_name:
  description: 可并行执行的任务描述
  agent: agent_name
  expected_output: 输出格式
  async_execution: true

# 并行组定义
parallel_groups:
  - group_id: "G1"
    label: "独立实现层"
    agents: [frontend_agent, backend_agent]
    depends_on: []
  - group_id: "G2"
    label: "集成验证层"
    agents: [qa_agent]
    depends_on: ["G1"]
```

### 流程类型选择

| 流程类型 | 适用场景 | 风险 |
|---|---|---|
| **Sequential** | 任务之间存在严格顺序依赖 | 总时间 = 所有任务之和 |
| **Hierarchical** | 需要 Manager Agent 动态分配 | 管理者成为瓶颈 |
| **Parallel** | 存在 2+ 个无依赖的独立子任务 | Token 消耗增加 |
| **混合** | 部分可并行，部分有依赖 | 任务图复杂度增加 |

**并行判定清单**：
- [ ] 不依赖其他任务输出
- [ ] 不写入共享资源
- [ ] 单个失败不影响其他并行任务
- [ ] 存在明确的汇总任务整合结果

## Phase 3: 自动续航执行

### 自动续航协议

```
1. 读取/创建 progress.yaml
2. 确定当前模块（first pending module）
3. 执行当前模块的所有任务
4. 模块验证（build/test/lint）
5. 更新 progress.yaml（标记完成 + 检查点）
6. 判断：是否还有未完成模块？
   ├── YES → 回到步骤 2（自动继续）
   └── NO  → 最终验证 → 输出完成报告
```

### progress.yaml 结构

```yaml
project:
  name: "项目名称"
  execution_mode: "serial"  # serial | parallel_layered
  auto_continue: true

design:
  status: "approved"  # drafting | reviewing | approved | rejected

modules:
  - id: shared
    status: completed
    verification:
      build: passed
      tests: passed
      lint: passed
    checkpoint: "commit:abc123"

  - id: fi
    status: in_progress
    current_task: "fi_repository_implementation"

summary:
  completion_percentage: 5
  next_module: "fi"
```

### 自动继续 vs 必须暂停

**✅ 自动继续**：模块/task 完成 | warning（非 error）| soft gate 未达标 | 独立模块

**⛔ 必须暂停**：需要外部凭证/无法推断的业务决策 | hard gate 失败 ≥ 3 次 | build error ≥ 3 轮 | 颠覆已批准设计 | 安全/数据丢失风险
