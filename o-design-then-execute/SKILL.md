---
name: o-design-then-execute
description: "三阶段工作流：多Agent评审→执行规格生成→自动续航执行。Use when: 新项目、复杂需求、架构设计、断点续航、dte、odte、dte触发词、omdte触发词。"
---

# Design-Then-Execute: 评审 → 规格 → 自动续航

三阶段开发工作流，自动续航直到完全完成：
1. **Phase 1** — 多 Agent 结构化评审，验证设计可行性
2. **Phase 2** — 生成 agents.yaml / tasks.yaml / crew.py
3. **Phase 3** — 按模块自动执行，无需逐步确认，直到完成

防止：未验证就实现、职责不清、设计缺陷延迟暴露、分阶段停滞。

---

## 三阶段架构总览

```
Phase 1: Designer → 并行评审(Skeptic|Guardian|Advocate) → 汇总修订 → Arbiter(APPROVED?)
Phase 2: → agents.yaml + tasks.yaml + crew.py
Phase 3: → progress.yaml 驱动自动续航
         ModuleA(✅) → ModuleB(🔄) → ModuleC(⏳) → ... → ALL_COMPLETE
```

---

## Phase 1: 结构化评审

### 步骤 1 — Designer 出方案

输出 6 个必须模块：目标定义、Agent 团队设计、Task 分解、流程类型决策、风险预案、评估策略声明。

详见 [phase-specs.md](references/phase-specs.md#phase-1-结构化评审)

### 步骤 2 — 并行评审（3 Agent 同时派发）

| 角色 | 职责 | 禁止 |
|---|---|---|
| **Skeptic** | 职责重叠/空白？依赖链瓶颈？expected_output 清晰？ | 提新功能、重新架构 |
| **Guardian** | Agent 数量 3-5？模型匹配？缓存策略？KPI 阈值？ | 讨论产品目标、加功能 |
| **Advocate** | 质量标准？输出风格？错误场景覆盖？可访问性？ | 重新架构、加功能 |

### 步骤 3 — Designer 汇总修订

每条反馈：接受→修订 | 拒绝→附理由 | 需讨论→交 Arbiter。

### 步骤 4 — Arbiter 仲裁

**APPROVED**(→P2) / **REVISE**(→步骤2) / **REJECT**(→步骤1)

---

## Phase 2: 执行规格生成

**仅在 Phase 1 APPROVED 后执行。**

### agents.yaml 示例

```yaml
agent_name:
  role: "精确的角色名称"
  goal: "具体目标"
  backstory: "角色背景"
  llm: "claude-sonnet-4-6"
  llm_fallback: ["qwen3.5-plus", "deepseek-v3", "claude-haiku-4-5"]
  cache_config:
    strategy: "prefix"
    ttl: 300
  tools: [ToolName]
```

### tasks.yaml 示例

```yaml
task_name:
  description: "任务描述"
  agent: agent_name
  expected_output: "输出格式"
  context: [dependent_task]

# 并行组
parallel_groups:
  - group_id: "G1"
    agents: [frontend_agent, backend_agent]
    depends_on: []
```

详细模板见 [phase-specs.md](references/phase-specs.md#phase-2-执行规格生成)

---

## Phase 3: 自动续航执行

### 自动续航协议

```
1. 读取 progress.yaml
2. 执行当前模块任务
3. 模块验证（build/test/lint）
4. 更新 progress.yaml
5. 自动推进下一模块（不询问用户）
```

### 自动继续 vs 必须暂停

**✅ 自动继续**：模块完成 | warning | soft gate 未达标

**⛔ 必须暂停**：外部凭证需求 | hard gate ≥ 3 次 | build error ≥ 3 轮 | 安全风险

详细规格见 [phase-specs.md](references/phase-specs.md#phase-3-自动续航执行)

---

## 模型路由与缓存

详见 [model-routing.md](references/model-routing.md)

核心原则：每个 Agent 根据任务性质分配最擅长的 LLM，必须声明降级链（至少 3 个候选）。

---

## KPI 评估与质量门控

详见 [kpi-evaluation.md](references/kpi-evaluation.md)

每步必须有可量化指标，三级门控：通过 / 软门控（警告） / 硬门控（重做）。

---

## 决策日志模板

```markdown
# 决策日志 — [项目名称]
## 决策 N: [标题]
- 选择/备选/评审反馈/结论理由
## 未解决项
## 最终裁决: APPROVED / REVISE / REJECT
```

---

## 退出条件

### Phase 1 & 2 退出条件

- [ ] 所有评审 Agent 已返回反馈
- [ ] 异议已解决或明确拒绝
- [ ] Arbiter 已裁决 APPROVED
- [ ] agents.yaml、tasks.yaml、crew.py 已生成

### Phase 3 退出条件

- [ ] 所有模块 status == "completed"
- [ ] 每个模块 verification 全部 passed
- [ ] 完成报告已生成

---

## Anti-Patterns

- ❌ 跳过评审直接写 YAML
- ❌ Agent 角色太泛 / expected_output 不具体
- ❌ 所有 Agent 用同一模型
- ❌ 无降级链
- ❌ 模块间等待用户确认（必须自动继续）
- ❌ 进度追踪缺失

---

## 依赖 Skills

`brainstorming` | `multi-agent-patterns` | `crewai` | `executing-plans` | `verification-before-completion`
