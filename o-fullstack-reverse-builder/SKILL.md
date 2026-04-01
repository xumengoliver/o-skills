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
| **前端** | Vue 3 + Vite + TypeScript + Pinia |
| **后端** | .NET Core 8.0 (C#) |
| **数据库** | SQL Server + EF Core |

用户可覆盖以上默认值。源站为 Vue2 时，默认升级到 Vue3 + Pinia（Vuex → Pinia）。

---

## 快速参考

```
流程: Phase 0 → F0 → F1 → F2 → F3 → F4 → F5 → B1~B5 → I0~I3 → DTE → D

Phase 0 : 共享分析（解析数据源 → 生成共享上下文）
Phase F0: 前端产物侦测（技术栈识别 → 构建工具判定 → Source Map 探测 → 策略决策）
Phase F1: 资源去 Hash 化（重命名文件 → 调整引用 → 获取干净静态版本）
Phase F2: 代码预处理（解压缩/格式化 → Source Map 还原 OR 硬核逆向入口）
Phase F3: Vue 组件提取（引导代码分析 → 组件导出定位 → Template 还原 → 变量语义化）
Phase F4: 项目重构（App.vue/路由 → 页面组件 → 小组件 → CSS 拆分 → Store 迁移）
Phase F5: 前端审查
Phase B : 后端生成（B1 路由图谱 → B2 Entity/DTO → B3 项目结构 → B4 代码生成 → B5 审查）
Phase I : 集成验证（I0 Response符合性 → I1 一致性检查 → I2 交叉修正 → I3 最终验证）
Phase DTE: 调用 design-then-execute 迭代验证
Phase D : 交付文件生成

输入: records.json + url-map.json + dto/ + responses/ + assets/
输出: ReverseBuilder/frontend/ + ReverseBuilder/backend/ + ReverseBuilder/docs/
```

---

## 核心决策树：Source Map 优先

```
┌─────────────────────────────────────────────┐
│           Phase F0: 产物侦测                │
│  识别技术栈 → 判定构建工具 → 探测 Source Map │
└────────────────┬────────────────────────────┘
                 │
         Source Map 存在？
        ┌────────┴────────┐
        YES               NO
        │                 │
   ┌────▼────┐     ┌─────▼──────┐
   │ 黄金路径 │     │  评估复杂度  │
   │ 直接还原 │     └──────┬─────┘
   │ 源码结构 │      ┌─────┴──────┐
   └─────────┘      低/中         高
                     │            │
              ┌──────▼──────┐ ┌──▼───────────┐
              │  硬核逆向路径 │ │ 新旧结合路径  │
              │ F1→F2→F3→F4 │ │ 抓API+资源    │
              └─────────────┘ │ 看UI重写Vue3  │
                              └───────────────┘
```

**优先级建议**：
1. **先找 Source Map** — 这是决定成败的关键
2. **有 Source Map** → 直接还原源码目录结构，几乎零损耗
3. **无 Source Map + 复杂度可控** → 硬核逆向路径（F1→F2→F3→F4）
4. **无 Source Map + 高度混淆/复杂** → 新旧结合：抓取 API 接口数据 + 图片样式资源 → 看着旧 UI 用 Vue3 重写，往往更快且代码质量更高

---

## 输出目录约定

```
{WebHack导出目录}/
├── records.json / url-map.json / dto/ / responses/ / assets/  # 原始数据 — 不动
└── ReverseBuilder/       # 生成的全栈项目根目录
    ├── frontend/         # Vue 3 + Vite + TypeScript + Pinia
    ├── backend/          # .NET Core 8.0 + EF Core
    └── docs/             # 跨项目共享文档
```

---

## 多角色协作架构

| 角色 | 职责 |
|---|---|
| **逆向分析师** | 分析构建产物、识别框架版本与构建工具、探测 Source Map、解析 Webpack/Vite 引导代码、策略决策 |
| **前端架构师** | 组件重建、路由还原、状态管理迁移（Vuex→Pinia）、CSS 拆分、API 层生成 |
| **前端审查员** | 质疑组件拆分合理性、路由完整性、API 层一致性、Scoped CSS 归属 |
| **后端架构师** | 分析 records.json，生成 Controller/Service/Repository |
| **后端审查员** | 质疑 API 设计、实体关系、安全/性能 |
| **集成验证员** | 验证前后端接口一致性、Response 结构匹配 |

### 协作流程

```
Phase 0 共享分析
    ↓
Phase F0 逆向分析师主导: 产物侦测 → 策略决策
    ↓
[黄金路径] 或 [硬核逆向路径] 或 [新旧结合路径]
    ↓
Phase F1~F4 前端架构师主导 → F5 前端审查员审查
    ↓  (与前端并行)
Phase B1~B4 后端架构师主导 → B5 后端审查员审查
    ↓
Phase I 集成验证员主导 → 通过/回退修正
```

### 挑战-应答协议

- **最多 3 轮**质疑
- 策略：`accept` | `reject_with_reason` | `defer`

---

## Phase 0：共享分析

验证核心文件存在，解析 records.json、url-map.json、dto/、responses/、assets/，输出 `ReverseBuilder/docs/shared-context.md`。

详见 [phase-specs.md](references/phase-specs.md)

---

## Phase F0：前端产物侦测（逆向分析师主导）

**目标**：在动手之前，先搞清楚"面对的是什么"，做出正确的策略决策。

### F0.1 技术栈识别

| 检测项 | Vue2 特征 | Vue3 特征 |
|---|---|---|
| 全局变量 | `new Vue(` / `Vue.component` | `createApp(` / `defineComponent` |
| 响应式 | `Object.defineProperty` | `Proxy`、`ref(`、`reactive(` |
| 路由 | `VueRouter` 3.x | `createRouter`、`createWebHistory` |
| 状态管理 | `Vuex`、`new Vuex.Store` | `Pinia`、`defineStore` 或 `Vuex` 4.x |
| 组合式 API | 无 | `setup()`、`<script setup>` |

### F0.2 构建工具判定

| 检测项 | Webpack 特征 | Vite 特征 |
|---|---|---|
| 引导代码 | `webpackJsonp`、`__webpack_require__` | `import.meta`、ES Module 原生导入 |
| chunk 命名 | `app.xxxxx.js`、`chunk-vendors.xxx.js` | `index-xxxxx.js`、动态 `import()` |
| HTML 入口 | `<script src="/js/app.xxx.js">` | `<script type="module" src="/assets/index-xxx.js">` |
| CSS 产物 | `css/app.xxx.css`、`chunk` CSS | `assets/index-xxx.css` |

### F0.3 Source Map 探测

按优先级依次检查：
1. assets/ 中 `*.map` 文件
2. JS 文件末尾 `//# sourceMappingURL=` 注释
3. HTTP 响应头 `SourceMap:` 或 `X-SourceMap:`（检查 records.json）
4. 尝试访问 `{js文件}.map` 路径（检查 url-map.json）

### F0.4 策略决策

输出 `docs/reverse-strategy.md`：
- 源站框架：Vue2 / Vue3
- 构建工具：Webpack / Vite / 其他
- Source Map：发现 / 未发现
- 代码压缩级别：未压缩 / 仅压缩 / 压缩+混淆
- **选定路径**：黄金路径 / 硬核逆向 / 新旧结合
- 路径选择理由

详见 [phase-specs.md](references/phase-specs.md)

---

## Phase F1：资源去 Hash 化

**目标**：将所有带 content-hash 的文件名还原为语义化名称，并修正所有内部引用，获得一个**无哈希戳的干净静态版本**。

### F1.1 文件重命名

- 构建产物 Hash：`index-a1b2c3.js` → `index.js`
- WebHack 后缀 Hash：`logo_a1b2c3d4.png` → `logo.png`
- CSS 文件同理

### F1.2 引用调整

- JS 文件内的 chunk 引用路径
- CSS 中 `url()` 引用
- HTML 中 `<script src>` 和 `<link href>`
- 动态 `import()` 路径

### F1.3 输出 `docs/dehash-mapping.yaml`

记录所有 `原始名 → 新名` 映射，用于后续回溯。

详见 [data-formats.md](references/data-formats.md)

---

## Phase F2：代码预处理

**目标**：让代码变成人可读的状态。

### F2.1 压缩检测与格式化

检查 JS 文件是否被压缩（单行超长、无换行缩进）：
- **已压缩** → 使用 Prettier/js-beautify 格式化
- **未压缩** → 直接进入下一步

### F2.2 Source Map 还原（黄金路径）

如果 F0 发现了 Source Map：
1. 使用 `source-map` 库解析 `.map` 文件
2. 直接还原出原始源码目录结构
3. 跳过 F3（组件提取），直接进入 F4（项目重构）

### F2.3 硬核逆向入口（无 Source Map）

如果没有 Source Map，进入 F3 硬核逆向流程。

详见 [phase-specs.md](references/phase-specs.md)

---

## Phase F3：Vue 组件提取（硬核逆向路径）

**目标**：从编译后的 JS 代码中提取 Vue 组件定义，还原 Template 和业务逻辑。

### F3.1 引导代码分析

- **Webpack**：定位 `webpackJsonp` / `__webpack_require__`，找到模块注册表
- **Vite**：定位 ES Module 导入图，找到组件动态 `import()` 链

### F3.2 组件导出定位

在模块注册表中查找：
- `defineComponent({` / `__defineComponent({`
- `render:` 函数 / `setup:` 函数
- `components:` 子组件注册
- `name:` 字段（最佳组件命名线索）

### F3.3 Template 还原

**从 render 函数反写 Template HTML**：
1. 识别 `createElement` / `h()` / `_createVNode` 调用
2. 分析嵌套结构 → 还原 DOM 树
3. `_withDirectives` → `v-if` / `v-show` / `v-model`
4. `_renderList` / `_Fragment` → `v-for`
5. `_createTextVNode` → 插值 `{{ }}`
6. 事件绑定 `onClick:` → `@click`

### F3.4 变量名语义化

| 压缩模式 | 还原策略 |
|---|---|
| 单字母变量 `a, b, c` | 根据 API 调用上下文推断 `userList, pageSize` |
| 短函数名 `fn1, fn2` | 根据函数体逻辑命名 `fetchUsers, handleSubmit` |
| 属性名保留 | API 响应字段名通常未压缩，可作锚点 |
| 路由路径保留 | `/user/list` → 推断 `UserList` 组件 |

### F3.5 输出 `docs/chunk-mapping.yaml`

记录 chunk → 组件 → 路由的完整映射。

详见 [phase-specs.md](references/phase-specs.md)

---

## Phase F4：项目重构

**目标**：将提取的组件按正确顺序组装为可运行的 Vue3 项目。

### F4.1 重构顺序（严格按序）

```
1. App.vue + 路由入口 (router/index.ts)
2. 布局组件 (Layout.vue / DefaultLayout.vue)
3. 页面级组件 (views/**/*.vue)
4. 小组件 / 共享组件 (components/**/*.vue)
5. CSS 拆分与归位
6. Store 迁移 (Vuex → Pinia)
7. API 层生成 (api/*.ts + types/api.ts)
```

### F4.2 CSS 拆分策略

| 样式类型 | 判定依据 | 处理方式 |
|---|---|---|
| Scoped CSS | `data-v-` 属性哈希 | 根据 `data-v-xxxxxxxx` 反向查找对应 Vue 文件，塞回 `<style scoped>` |
| 全局样式 | 无 `data-v-` 前缀 | → `src/assets/styles/global.css` |
| 第三方库样式 | Element/Ant/Vuetify 类名 | → `package.json` 依赖 + 全局引入 |
| CSS Variable | `:root { --xxx }` | → `src/assets/styles/variables.css` |

**Scoped CSS 还原流程**：
1. 扫描所有 CSS 规则中的 `[data-v-xxxxxxxx]` 选择器
2. 在 HTML/render 函数中查找相同 `data-v-` 哈希对应的组件
3. 将匹配的 CSS 规则移入对应 `.vue` 文件的 `<style scoped>`
4. 删除 `[data-v-xxxxxxxx]` 选择器后缀（Vue SFC 编译时自动添加）

### F4.3 Store 迁移（Vuex → Pinia）

| Vuex 概念 | Pinia 等价 | 迁移注意 |
|---|---|---|
| `state` | `state: () => ({})` | 直接迁移 |
| `getters` | `getters: {}` | 参数从 `(state)` 改为 `(this)` 可选 |
| `mutations` | ❌ 不再需要 | 直接在 action 中修改 state |
| `actions` | `actions: {}` | 去掉 `commit()`，直接 `this.xxx = yyy` |
| `modules` | 独立 `defineStore()` | 每个模块 → 独立 Store 文件 |
| `mapState/mapGetters` | `storeToRefs()` | `<script setup>` 中解构 |
| `mapActions` | 直接调用 `store.xxx()` | 无需映射 |

### F4.4 路由与守卫迁移

从旧代码中寻找 `router.beforeEach` / `beforeResolve` / `afterEach`，迁移到 Vue3 路由格式。

详见 [phase-specs.md](references/phase-specs.md)

---

## Phase F5：前端审查

检查：组件拆分合理性、路由完整性、API 层匹配、Pinia Store 正确性、Scoped CSS 归属、Source Map 还原准确性、Hash 映射完整性。

---

## Phase B：后端生成

### B1：API 路由图谱

按 URL 前缀分组成模块，标注认证需求。

### B2：Entity / DTO 模型

实体关系推导、DTO 分类命名。

详见 [phase-specs.md](references/phase-specs.md)

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

详见 [phase-specs.md](references/phase-specs.md)

---

## Phase DTE：design-then-execute 迭代验证

最多 3 轮 DTE 迭代。审查焦点：架构合理性、API 安全性、前后端一致性。

---

## Phase D：交付文件生成

生成 CONFIGURATION.md、Docker 文件、环境变量模板、.gitignore、README.md。

详见 [phase-specs.md](references/phase-specs.md)

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
- ❌ 跳过 Phase 0 / F0 / 审查 / I0
- ❌ 未做 Source Map 探测就直接硬核逆向
- ❌ 用 chunk ID 或 Hash 命名组件（应还原业务名）
- ❌ 复制 responses/（应用种子数据）
- ❌ Controller 返回原始 JSON 串（应用强类型 DTO）
- ❌ 保留 Vuex 不迁移 Pinia（源站为 Vue2 时）
- ❌ Scoped CSS 不拆分直接全局堆砌
- ❌ 压缩代码不格式化就开始分析

---

## 依赖 Skills

`design-then-execute` | `dotnet-backend` | `vue` | `typescript-pro` | `docker-expert`
