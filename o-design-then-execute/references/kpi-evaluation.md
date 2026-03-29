# KPI 评估与质量门控

## 评估框架选择

- **DeepEval**：纯 Agent 工作流
- **RAGAS**：含 RAG 检索场景
- **混用**：复杂场景同时使用

## 每步 KPI 指标

| 步骤 | Agent | 核心指标 | 阈值 |
|---|---|---|---|
| Step 1 | Designer | `GEval("completeness")` 6模块完整 / `GEval("specificity")` 角色精确 / `AnswerRelevancy` / `Hallucination≤0.10` | ≥0.75~0.80 |
| Step 2a | Skeptic | `Faithfulness≥0.90` / `GEval("review_depth")≥0.70` / `GEval("scope_compliance")≥0.90` | — |
| Step 2b | Guardian | `Faithfulness≥0.90` / `GEval("constraint_coverage")≥0.80` / `GEval("scope_compliance")≥0.90` | — |
| Step 2c | Advocate | `Faithfulness≥0.90` / `GEval("user_perspective")≥0.75` / `GEval("scope_compliance")≥0.90` | — |
| Step 3 | Designer修订 | `GEval("feedback_addressing")≥0.85` / `Faithfulness≥0.85` / `GEval("decision_logging")≥0.90` | — |
| Step 4 | Arbiter | `Faithfulness≥0.95` / `GEval("decision_quality")≥0.80` / `Hallucination≤0.05` | — |
| Phase 2 | YAML/Code | `GEval("spec_fidelity")≥0.90` / `GEval("yaml_validity")≥0.95` / `GEval("dependency_correctness")≥0.90` | — |

## 质量门控规则

```
每步完成 → 运行 KPI 评估
  ├── score ≥ threshold        → ✅ 通过，继续
  ├── threshold×0.7 ≤ score    → ⚠️ 软门控：警告+记录决策日志，继续
  └── score < threshold×0.7    → ⛔ 硬门控：必须重做
```

## 评估配置

- `eval_model`：至少 `gpt-4o-mini`，复杂场景用 Opus/o3
- `eval_gate`：生产环境用 `hard`，原型阶段用 `soft`
- 阈值范围：0.5~0.9 合理区间（不要设 1.0 或低于 0.5）

## 常见误用

- 阈值全设 1.0（不现实） / 低于 0.5（无意义）
- 用 Haiku 做 Judge → eval_model 至少 gpt-4o-mini
- 只在最终步骤评估 → 每步都要评估
- 连续 2+ 次 soft gate 警告 → 应升级为 hard gate
