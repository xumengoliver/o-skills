# Phase 详细规格

## Phase 0：共享分析

### 0.1 识别导出目录

验证以下核心文件或目录存在：

- `records.json`
- `url-map.json`
- `dto/`
- `responses/`
- `assets/`

可选但需要记录的文件：

- `routes.js`
- `server.js`
- `package.json`
- `api-summary.md`
- `file-structure.md`

如果核心输入缺失，直接在 `ReverseBuilder/docs/shared-context.md` 标注缺口，并暂停对应阶段，不要臆造数据源。

### 0.2 技术栈确认

默认目标技术栈：

- 前端：Vue 3 + Vite + TypeScript + Pinia
- 后端：.NET Core 8.0 + C#
- 数据库：SQL Server + EF Core

若用户未指定，以默认栈生成。若源站明显为 Vue2，则允许“源站 Vue2，目标 Vue3”的升级式重建，但必须在文档中明确写出升级决策。

### 0.3 解析核心数据

按以下顺序构建共享上下文：

1. 从 `records.json` 提取所有 `resourceType === "xhr"` 或可判定为接口调用的记录。
2. 排除 404、重定向、空响应及无效采样。
3. 以 `method + normalized pathname` 去重，并保留最完整样本。
4. 从 `url-map.json` 提取静态资源、JS、CSS、HTML、图片、字体的本地映射关系。
5. 从 `dto/index.ts` 提取接口名、字段名、字段类型、可选性。
6. 从 `responses/` 中回填 DTO 的 `unknown` 字段、数组元素结构、嵌套对象结构。
7. 从 `records.json` 的请求头中识别认证机制、租户标识、区域标识、自定义 trace 头。
8. 从响应结构中识别统一包裹格式，例如 `code`、`errcode`、`message`、`data`、`success`。

### 0.4 输出：共享上下文文档

写入 `ReverseBuilder/docs/shared-context.md`，至少包含：

- 核心输入清单
- 推断出的源站技术栈
- API 总数与模块划分
- 静态资源结构摘要
- DTO 与 response 的可信度说明
- 已知缺失项和风险项

---

## Phase F0：前端产物侦测

### F0.1 技术栈识别

至少从以下维度判定：

- Vue2 / Vue3
- Webpack / Vite / 其他打包器
- 是否启用代码压缩
- 是否启用变量混淆
- 是否存在 SSR / 纯 SPA 特征

建议证据：

- `new Vue(`、`Vue.component`、`VueRouter`、`new Vuex.Store`
- `createApp(`、`defineComponent(`、`createRouter(`、`defineStore(`
- `webpackJsonp`、`__webpack_require__`
- `import.meta`、ESM `import`

### F0.2 构建工具判定

按文件结构和引导代码双重确认：

1. 看 HTML 入口脚本写法。
2. 看 JS 顶部 bootstrap 结构。
3. 看 chunk 命名风格。
4. 看 CSS 输出路径与命名风格。

输出结论必须明确到“高可信度 / 中可信度 / 低可信度”。

### F0.3 Source Map 探测

按优先级执行：

1. 直接搜索 `assets/` 中的 `*.map`
2. 搜索 JS 文件尾部的 `sourceMappingURL`
3. 检查 `records.json` 响应头中的 `SourceMap` 或 `X-SourceMap`
4. 通过 `url-map.json` 判断是否存在隐式 map 路径

若命中 Source Map：

- 记录命中文件
- 记录映射来源
- 记录可恢复范围（单文件、局部、全站）

### F0.4 压缩与混淆级别判断

区分三类：

- 未压缩：保留函数名、换行和注释较多
- 仅压缩：单行或短变量，但结构仍可读
- 压缩 + 混淆：大量 `a,b,c,n,t` 变量，控制流可读性极差

### F0.5 路径决策

输出 `ReverseBuilder/docs/reverse-strategy.md`，结论必须是以下之一：

- 黄金路径：Source Map 可直接恢复源码结构
- 硬核逆向路径：无 Source Map，但可通过 JS/CSS 产物恢复
- 新旧结合路径：仅保留 API、资源和视觉参考，用 Vue3 重新实现 UI

文档需包含：

- 选择该路径的理由
- 放弃其他路径的理由
- 风险清单
- 预计最脆弱的阶段

---

## Phase F1：资源去 Hash 化

### F1.1 文件归一化

将带 hash 的构建产物改为稳定语义名，例如：

- `index-a1b2c3.js` → `index.js`
- `chunk-vendors-xxxx.js` → `chunk-vendors.js`
- `logo_a1b2c3d4.png` → `logo.png`

### F1.2 引用修复

修复以下引用来源：

- HTML 中的 `script`、`link`
- JS 中的动态 `import()`
- CSS 中的 `url(...)`
- 运行时拼接的资源路径

### F1.3 输出映射

生成 `ReverseBuilder/docs/dehash-mapping.yaml`，用于后续追溯。映射至少包含：

- 原文件名
- 新文件名
- 类型
- 所属目录
- 是否人工确认

### F1.4 退出条件

满足以下条件才进入 F2：

- 所有入口文件可被稳定引用
- 主要 JS/CSS 文件名已去 Hash
- 关键资源路径不再断裂

---

## Phase F2：代码预处理

### F2.1 解压缩与格式化

对 JS/CSS 进行规范化处理：

- JS：格式化、保留注释、保留模块边界
- CSS：拆回可读选择器和声明块
- HTML：恢复可读结构

### F2.2 黄金路径：Source Map 还原

若存在可用 Source Map：

1. 恢复原始源码目录结构
2. 恢复原始模块边界
3. 恢复原始文件名与相对引用
4. 记录无法恢复的孤立模块

此路径下，F3 可以降级为“校准与补洞”，而不是从零逆向。

### F2.3 硬核逆向入口

若无 Source Map，则输出给 F3 的中间产物：

- bootstrap 分析笔记
- chunk 依赖图
- 组件候选导出清单
- 路由候选清单
- store 候选清单

### F2.4 风险标注

对下列情况加 TODO 标记并记录到 `docs/reverse-risk-log.md`：

- 动态路由拼接
- 条件编译片段
- 环境变量分支
- 混淆导致的语义丢失

---

## Phase F3：Vue组件提取

### F3.1 引导代码分析

先判断入口属于哪类：

- Webpack：`webpackJsonp`、`__webpack_require__`、模块编号分发表
- Vite：ESM import 链、`import.meta`、动态 chunk

目标是拿到：

- 根应用入口
- 路由注册位置
- store 注册位置
- 主要页面 chunk

### F3.2 组件导出定位

优先关注以下结构：

- `defineComponent(...)`
- `export default`
- `setup()`
- `render()`
- `components:`
- `name:`

必须把“页面组件”和“可复用子组件”区分开，不允许所有东西都塞回单文件页面。

### F3.3 Template 还原

按以下顺序还原：

1. 从 `render()`、`h()`、`createElement`、`_createVNode` 提取 DOM 树
2. 从 `_renderList` 识别 `v-for`
3. 从条件分支识别 `v-if`、`v-else-if`、`v-show`
4. 从事件绑定识别 `@click`、`@change`、`@input`
5. 从文本节点识别插值表达式
6. 从指令包装识别 `v-model`、自定义指令、slot 边界

### F3.4 变量语义化

将仅有位置含义的变量重命名为语义变量，例如：

- `a` → `userList`
- `b` → `selectedRow`
- `n` → `fetchTableData`

重命名依据必须来自：

- API 调用上下文
- 模板使用位置
- 字段结构
- 路由或组件名

不要无依据创造业务术语。

### F3.5 输出映射

生成 `ReverseBuilder/docs/chunk-mapping.yaml`，至少包含：

- chunk 文件
- 推断组件名
- 推断路由路径
- 依赖的 API 模块
- 可信度等级

### F3.6 退出条件

进入 F4 前必须具备：

- 根组件边界清晰
- 主要页面组件已识别
- 至少一版路由树已可生成
- 组件与 chunk 的主映射已建立

---

## Phase F4：项目重构

### F4.1 重构顺序

严格按以下顺序落地：

1. 初始化 Vue3 + Vite + TypeScript 工程
2. 建立 `App.vue` 与全局布局
3. 建立路由树
4. 落地主页面组件
5. 抽取复用子组件
6. 接入 API 层和类型层
7. 迁移状态管理与样式体系

### F4.2 CSS 拆分策略

将样式拆为四类：

- Scoped CSS：回归对应组件
- Global CSS：放入全局样式入口
- 第三方样式：独立保留并标注来源
- 设计变量：抽取为 tokens 或 CSS variables

若发现 `data-v-xxxx`：

1. 从组件 render/template 中反向定位作用域节点
2. 比对类名和结构层级
3. 把样式归到最可能的 SFC
4. 不确定时保守放入局部模块样式并标注 TODO

### F4.3 API 层与类型层

生成：

- `src/api/index.ts`：Axios 实例与拦截器
- `src/api/{module}.ts`：按业务模块拆分
- `src/types/api.ts`：由 DTO 转换的 TS 类型

URL、方法、请求体、响应体必须与后端规划一致。

### F4.4 Vuex → Pinia 迁移

迁移规则：

| Vuex | Pinia |
| --- | --- |
| `state` | `state` |
| `getters` | `getters` |
| `mutations` | 直接写 action 或直接改 state |
| `actions` | `actions` |
| `modules` | 多 store 文件 |
| `mapState/mapGetters` | `storeToRefs` / 直接访问 |
| `mapActions/mapMutations` | 直接调用 store action |

### F4.5 路由守卫迁移

将原有登录态、权限、菜单、租户判断逻辑迁移到 Vue Router 4：

- 全局前置守卫
- 路由 meta
- 权限白名单
- 登录跳转保活逻辑

### F4.6 新旧结合路径特殊要求

若走“新旧结合”路径：

- 不强求完全还原原组件树
- 以 API 一致、视觉接近、交互闭环为优先
- 必须在文档中明确哪些页面是“参考旧 UI 重写”

---

## Phase F5：前端审查

前端审查员至少检查以下 7 项：

1. 页面组件和子组件拆分是否合理
2. 路由树是否覆盖主要页面
3. API 地址、方法、参数是否匹配 records.json
4. Pinia store 是否保留原有业务边界
5. Scoped CSS 是否错误串页
6. 设计变量和全局样式是否污染局部组件
7. 无法确定的片段是否被明确标记 TODO

审查结论只允许：

- `accept`
- `reject_with_reason`
- `defer`

---

## Phase B：后端生成

### B1：API 路由图谱

从 `records.json` 生成后端路由图谱：

- 按 URL 前缀聚类为业务模块
- 标记 HTTP 方法
- 标记认证需求
- 标记分页、导出、文件上传、文件下载等特征

### B2：Entity / DTO 模型

#### 实体关系推导规则

1. Response 中稳定数组结构可抽取为独立实体。
2. 字段名含 `Id`、`Code`、`No` 时优先判断为外键或业务标识。
3. `createTime`、`updateTime`、`createdBy` 等字段归为审计字段。
4. `parentId`、`pid`、`children` 暗示树形或自引用结构。
5. `unknown`、空对象、空数组字段保留 nullable 并写 TODO。
6. 同一对象中明显不同前缀字段组允许拆为子对象或导航属性。
7. 多对多结构以中间表实体实现，不直接拍扁。

#### DTO 分类命名

| 类别 | 命名 | 来源 |
| --- | --- | --- |
| Response DTO | `XxxResponse` | `dto/index.ts` 或 response 结构 |
| Request DTO | `XxxRequest` | `requestBody` |
| Query DTO | `XxxQuery` | URL 查询参数 |
| Entity | `Xxx` | 从响应结构反推 |

### B3：.NET Core 项目结构

建议结构：

```text
ReverseBuilder/backend/src/MyApp/
├── Program.cs
├── appsettings.json
├── Controllers/
├── Services/
├── Repositories/
├── Entities/
├── DTOs/
│   ├── Requests/
│   ├── Responses/
│   └── Queries/
├── Data/
│   ├── AppDbContext.cs
│   ├── Configurations/
│   └── Seeds/
├── Middleware/
├── Common/
└── Mapping/
    └── MappingProfile.cs
```

### B4：代码生成

至少落地：

- Controller / Service / Repository 三层
- EF Core 实体与配置
- AutoMapper 或等价映射层
- 统一响应包装（若项目大多数接口共享同一协议）
- 错误码常量或枚举
- 种子数据初始化

### B5：后端审查

后端审查员至少检查：

- 路由设计是否与原始接口一致
- DTO 与 Entity 是否过度耦合
- EF 关系映射是否合理
- 认证与鉴权是否遗漏
- 分页、排序、导出接口是否被错误简化
- 空值与异常路径是否有兜底

---

## Phase I：集成验证

### I0：Response 文件符合性验证

逐条比对 `responseFile` 与后端 DTO：

- 字段是否存在
- 类型是否兼容
- 数组和嵌套对象是否匹配
- 包裹层结构是否一致

退出门槛：

- 通过：100% 字段可解释
- 条件通过：仅剩少量无法自动判定的展示字段
- 不通过：存在关键字段缺失或类型不兼容

### I1：接口一致性检查

至少检查以下 12 项：

1. 前端 API URL 与后端路由一致
2. HTTP 方法一致
3. 查询参数命名一致
4. 请求体结构一致
5. 响应包裹结构一致
6. 认证头处理一致
7. 文件上传字段一致
8. 文件下载 content type 一致
9. 分页字段一致
10. 错误码映射一致
11. CORS 与代理配置一致
12. 前端类型声明与后端 DTO 一致

### I2：交叉修正

发现问题后，必须回退到对应责任侧修复：

- URL / DTO / 鉴权错误 → 后端侧
- 页面逻辑 / 参数传递 / 样式错误 → 前端侧
- 共享协议理解错误 → 更新 shared-context 与 strategy 文档

### I3：最终验证

至少完成：

- 前端构建通过
- 后端构建通过
- 数据库迁移可执行
- Swagger 或接口文档可生成
- 主流程页面可跑通

---

## Phase DTE：design-then-execute 迭代验证

最多允许 3 轮 DTE。每轮必须说明：

- 本轮质疑点
- 被质疑的证据
- 接受 / 拒绝原因
- 落地修正点

如果第 3 轮仍无法收敛，直接在交付文档中列为“人工确认项”，不要无限循环。

---

## Phase D：交付文件生成

### D1：docs/CONFIGURATION.md

至少包含 8 个配置区块：

1. 项目名与命名空间替换
2. 数据库连接字符串
3. 前端环境变量
4. JWT / 认证配置
5. CORS 配置
6. 前后端端口
7. 日志级别
8. 种子数据开关

### D2：环境变量模板

至少生成：

- `frontend/.env.example`
- `.env.example`

### D3：.gitignore 补充

补充前端构建产物、后端构建产物、日志、环境变量、IDE 文件。

### D4：README.md 追加

README 至少应说明：

- 项目结构
- 启动步骤
- 数据库初始化方式
- 前后端联调方式
- 已知未完全自动恢复的部分
