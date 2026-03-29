---
name: o-fullstack-reverse-builder
description: "从WebHack导出的Mock Server项目逆向还原完整前后端项目。Use when: 目录含records.json/url-map.json/dto/responses/assets，frb、ofrb、frb触发词、或ofrb触发词。"
---

# Fullstack Reverse Builder

从 WebHack 导出的结构化 Mock Server 项目，逆向还原并生成**完整全栈项目**（前端 + 后端 + 数据库）。

---

## 默认技术栈

| 层 | 默认技术 |
|---|---|
| **前端** | Vue 3 + Vite + TypeScript |
| **后端** | .NET Core 8.0 (C#) |
| **数据库** | SQL Server + EF Core |

用户可覆盖以上默认值。

---

## 快速参考

```
流程: Phase 0 → F1 → F2 → F3 → B1 → B2 → B3 → B4 → B5 → I0 → I1 → I2 → I3 → DTE → D

Phase 0: 共享分析（解析数据源 → 生成共享上下文）
Phase F: 前端生成（F1 Hash分析 → F2 组件重建 → F3 审查）
Phase B: 后端生成（B1 路由图谱 → B2 Entity/DTO → B3 项目结构 → B4 代码生成 → B5 审查）
Phase I: 集成验证（I0 Response符合性 → I1 一致性检查 → I2 交叉修正 → I3 最终验证）
Phase DTE: 调用 design-then-execute 迭代验证
Phase D: 交付文件生成

输入: records.json + url-map.json + dto/ + responses/ + assets/
输出: ReverseBuilder/frontend/ + ReverseBuilder/backend/ + ReverseBuilder/docs/
```

---

## 输出目录约定

```
{WebHack导出目录}/
├── records.json / url-map.json / dto/ / responses/ / assets/  # 原始数据 — 不动
└── ReverseBuilder/       # 生成的全栈项目根目录
    ├── frontend/         # Vue 3 + Vite + TypeScript
    ├── backend/          # .NET Core 8.0 + EF Core
    └── docs/             # 跨项目共享文档
```

---

## 多角色协作架构

| 角色 | 职责 |
|---|---|
| **前端架构师** | 分析 assets/ 产物，还原 Vue 组件/路由/状态管理/API层 |
| **前端审查员** | 质疑组件拆分、路由还原、API层一致性 |
| **后端架构师** | 分析 records.json，生成 Controller/Service/Repository |
| **后端审查员** | 质疑 API 设计、实体关系、安全/性能 |
| **集成验证员** | 验证前后端接口一致性 |

### 协作流程

```
Phase 0 共享分析 → 前后端并行 → 各自审查 → 集成验证 → 通过/回退修正
```

### 挑战-应答协议

- **最多 3 轮**质疑
- 策略：`accept` | `reject_with_reason` | `defer`

---

## Phase 0：共享分析

验证核心文件存在，解析 records.json、url-map.json、dto/、responses/、assets/，输出 `ReverseBuilder/docs/shared-context.md`。

详见 [phase-specs.md](references/phase-specs.md#phase-0共享分析)

---

## Phase F：前端生成

### F1：Hash 分析与 chunk 映射

执行五步还原，输出 `docs/chunk-mapping.yaml`。

详见 [data-formats.md](references/data-formats.md#vue-hash文件名还原)

### F2：Vue 组件结构重建

生成标准 Vue3+Vite+TS 项目，包含路由、API 层、组件。

详见 [phase-specs.md](references/phase-specs.md#phase-f前端生成)

### F3：前端审查

检查组件拆分、路由完整性、API 层匹配、状态管理、样式还原。

---

## Phase B：后端生成

### B1：API 路由图谱

按 URL 前缀分组成模块，标注认证需求。

### B2：Entity / DTO 模型

实体关系推导、DTO 分类命名。

详见 [phase-specs.md](references/phase-specs.md#phase-b后端生成)

### B3：.NET Core 项目结构

标准三层架构：Controllers/Services/Repositories。

### B4：代码生成

Controller/Service/Repository、ApiResponse\<T\>、种子数据、ErrorCodes.cs。

### B5：后端审查

检查 API 设计、实体关系、安全、性能、错误处理。

---

## Phase I：集成验证

### I0：Response 文件符合性验证

比对 responseFile JSON 与后端 DTO。

### I1：接口一致性检查（12 项）

前端 API URL 与后端路由匹配、HTTP 方法一致、DTO 字段对应等。

### I2：交叉修正

定位问题侧，由对应架构师修正。

### I3：最终验证

前后端编译通过、数据库迁移无错、Swagger 文档正确。

详见 [phase-specs.md](references/phase-specs.md#phase-i集成验证)

---

## Phase DTE：design-then-execute 迭代验证

最多 3 轮 DTE 迭代。审查焦点：架构合理性、API 安全性、前后端一致性。

---

## Phase D：交付文件生成

生成 CONFIGURATION.md、Docker 文件、环境变量模板、.gitignore、README.md。

详见 [phase-specs.md](references/phase-specs.md#phase-d交付文件生成)

---

## 项目命名规则

1. **优先用户指定**
2. **域名推断**：从 records.json URL 提取
3. **子目录推断**：从 API 前缀推断
4. **兜底**：保持 `MyApp` + TODO 标记

---

## 代码注释与 TODO 约定

- **C#**：`/// <summary>方法描述 — 对应 HTTP方法 路径</summary>`
- **Vue**：`<!-- [REVERSE-BUILDER] 还原自 chunk: xxx | 路由: /path -->`
- **TODO**：统一 `TODO [REVERSE-BUILDER] {描述}` 前缀

---

## 反模式清单

- ❌ 编辑原始数据
- ❌ 硬编码 API URL
- ❌ 跳过 Phase 0/审查/I0
- ❌ 用 chunk ID 命名组件（应还原业务名）
- ❌ 复制 responses/（应用种子数据）
- ❌ Controller 返回原始 JSON 串（应用强类型 DTO）

---

## 依赖 Skills

`design-then-execute` | `dotnet-backend` | `vue` | `typescript-pro` | `docker-expert`
