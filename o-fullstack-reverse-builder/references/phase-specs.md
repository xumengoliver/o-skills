# Phase 详细规格

## Phase 0：共享分析

### 0.1 识别导出目录

验证核心文件存在：`records.json`、`url-map.json`、`routes.js`、`server.js`、`assets/`、`responses/`。

### 0.2 技术栈确认

默认：Vue3/Vite/.NET 8/SQL Server/EF Core。用户可覆盖。

### 0.3 解析核心数据（6 步）

1. **records.json**：筛选 `resourceType==="xhr"`，排除 404，按 pathname 去重
2. **url-map.json**：按 resourceType 分组
3. **dto/index.ts**：提取 interface 名称与字段
4. **responses/**：补充 DTO 中 `unknown` 类型字段
5. **assets/**：框架检测，分析入口
6. **错误码清单**：识别 `errcode`/`code`/`error` 字段

### 0.4 输出：共享上下文文档

写入 `ReverseBuilder/docs/shared-context.md`。

---

## Phase F：前端生成

### F1：Hash 分析与 chunk 映射

**五步还原**：
1. 识别入口（index.html 的 script/link）
2. 分析 JS 内部结构（Router、chunk 注释、defineComponent）
3. 构建 chunk→组件映射表 → `docs/chunk-mapping.yaml`
4. 生成 Vue 项目结构
5. 精度处理（无法确定组件名时以路由路径命名）

### F2：Vue 组件结构重建

#### F2.1 项目脚手架

Vue3+Vite+TS，`vite.config.ts` 必须包含 `/api` 代理到后端。

#### F2.2 路由还原

从入口 JS 提取路由定义 → `src/router/index.ts`

#### F2.3 API 调用层

- `src/api/index.ts`：Axios 实例 + 拦截器
- `src/api/{module}.ts`：按模块分文件
- `src/types/api.ts`：从 dto/index.ts 转换

#### F2.4 组件生成

| 类别 | 目录 |
|---|---|
| Layout | `views/Layout.vue` |
| 路由视图 | `views/{模块}/{Name}.vue` |
| 共享组件 | `components/Common/{Name}.vue` |
| 单页子组件 | `components/{模块}/{Name}.vue` |

### F3：前端审查

检查：组件拆分、路由完整性、API 层匹配、状态管理、样式还原、Hash 映射准确性。

---

## Phase B：后端生成

### B1：API 路由图谱

按 URL 前缀分组成模块，标注认证需求。

### B2：Entity / DTO 模型

#### 实体关系推导规则

1. Response 中 data 数组 → 提取为独立实体
2. 字段名含 Id/Code 后缀 → 外键引用
3. createTime/updateTime → 审计字段
4. parentId/pid → 树形自引用
5. unknown → nullable + TODO
6. 不同 prefix 的字段组 → 拆分实体 + 导航属性
7. `List<Tag>` in `Article` → 中间表实体

#### DTO 分类命名

| 类别 | 命名 | 来源 |
|---|---|---|
| Response DTO | `XxxResponse` | dto/index.ts |
| Request DTO | `XxxRequest` | records.json requestBody |
| Query DTO | `XxxQuery` | URL 查询参数 |
| Entity | `Xxx`（无后缀） | 从 Response 反推 |

### B3：.NET Core 项目结构

```
ReverseBuilder/backend/src/MyApp/
├── Program.cs / appsettings.json
├── Controllers/
├── Services/
├── Repositories/
├── Entities/
├── DTOs/{Requests,Responses,Queries}/
├── Data/AppDbContext.cs + Configurations/ + Seeds/
├── Middleware/
├── Common/
└── Mapping/MappingProfile.cs
```

### B4：代码生成

- Controller/Service/Repository 三层
- ApiResponse\<T\> 统一响应格式（≥80% API 共享结构时使用）
- 种子数据从 responses/*.json 提取
- ErrorCodes.cs 从错误码清单生成

### B5：后端审查

检查：API 设计、实体关系、安全、性能、错误处理、DTO 转换。

---

## Phase I：集成验证

### I0：Response 文件符合性验证

比对 responseFile JSON 与后端 DTO。

**退出门槛**：
- 通过：100% 字段匹配
- 条件通过：≤3 个无法自动判定
- 不通过：字段缺失/类型不兼容

### I1：接口一致性检查（12 项）

1. 前端 API URL 与后端路由匹配
2. HTTP 方法一致
3. 请求/响应 DTO 字段对应
4. Token 处理同步
5. CORS 配置正确
...（共 12 项）

### I2：交叉修正

定位问题侧，由对应架构师修正。

### I3：最终验证

- 前端 `npm run build` 通过
- 后端 `dotnet build` 通过
- 数据库迁移脚本执行无错
- Swagger 文档生成正确

---

## Phase DTE：design-then-execute 迭代验证

最多 3 轮 DTE 迭代。审查焦点：架构合理性、API 安全性、前后端一致性。

---

## Phase D：交付文件生成

### D1：docs/CONFIGURATION.md

8 个配置区块：项目名替换、连接字符串、前端环境变量、JWT、CORS、端口、日志级别、种子数据开关。

### D2：Docker 交付文件

- `Dockerfile.backend`
- `Dockerfile.frontend`
- `docker-compose.yml`
- `docker/nginx.conf`

### D3：环境变量模板

- `frontend/.env.example`
- `.env.example`

### D4：.gitignore 补充

### D5：README.md 追加
