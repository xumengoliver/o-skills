# 数据源格式与处理

## 导出目录标准结构

```text
{项目名}-export/
├── package.json / server.js / routes.js
├── records.json
├── url-map.json
├── api-summary.md / file-structure.md
├── assets/{domain}/
├── dto/index.ts
└── responses/*.json|*.bin
```

说明：

- `records.json` 是接口逆向的主证据
- `url-map.json` 是静态资源和本地文件映射的主证据
- `dto/index.ts` 是类型命名和字段约束的主证据
- `responses/` 是字段补全与实体推断的补强证据

## records.json 关键字段

| 字段 | 含义 | 逆向用途 |
| --- | --- | --- |
| `method` | HTTP 方法 | Controller 方法类型 |
| `url` | 完整请求 URL | 路径、模块和查询参数定义 |
| `requestBody` | 请求体 JSON | 推断 Request DTO |
| `statusCode` | 响应状态码 | 判断是否采用该样本 |
| `responseFile` | 响应体文件路径 | 读取真实响应推断 DTO |
| `resourceType` | xhr/document/script/stylesheet 等 | 区分 API 和静态资源 |
| `contentType` | 响应内容类型 | 决定序列化方式或文件返回方式 |
| `requestHeaders` | 请求头 | 提取认证方式、租户信息、区域信息 |
| `responseHeaders` | 响应头 | CORS、缓存、Source Map、下载头 |

## url-map.json resourceType 枚举

| 值 | 含义 | 前端处理 | 后端处理 |
| --- | --- | --- | --- |
| `script` | JS | 分析组件、路由、状态管理 | 识别 API 调用路径 |
| `document` | HTML | 提取 SPA 或多页入口 | 识别页面落点 |
| `stylesheet` | CSS | 提取样式、主题、变量 | 静态资源服务 |
| `image` / `font` / `media` | 静态资源 | 迁移到资源目录 | 静态资源服务 |

## dto/index.ts 命名规则

典型模式：`Api` + 路径各段 PascalCase + `Response`

示例：`ApiPublicSystemConfigResponse`

使用方式：

- 优先保留已有 DTO 名称，不随意重命名
- 若 response 结构与 DTO 冲突，以真实 response 为准，同时记录 DTO 偏差

## responses/ .bin 文件处理

| Content-Type | 处理方式 |
| --- | --- |
| `text/html` | 跳过，不纳入 API 路由图 |
| `application/octet-stream` | 生成文件下载端点 |
| `image/*` / `font/*` | 迁移到静态资源目录 |
| `application/pdf` / `application/zip` | 标记为下载接口 |

## 去Hash化流程

### 目标

将构建产物中的 hash 文件名还原为稳定语义名，并修正所有引用路径，得到后续可分析、可构建、可追踪的中间版本。

### 两类 hash 场景

1. 构建产物 hash：如 `index-a1b2c3.js`、`main.feba65ae.js`
2. WebHack 资源后缀 hash：如 `logo_a1b2c3d4.png`

### 推荐处理顺序

1. 从 HTML 入口识别首屏 JS/CSS
2. 对入口文件去 Hash
3. 递归修复其内部依赖引用
4. 对图片、字体等静态资源去 Hash
5. 生成最终映射表

### 输出格式

输出 `dehash-mapping.yaml` 时建议至少包含：

```yaml
- original: assets/index-a1b2c3.js
  normalized: assets/index.js
  type: script
  referencedBy:
    - index.html
  confidence: high
```

## Source Map 线索格式

Source Map 线索主要来自以下位置：

1. `*.map` 物理文件
2. JS 尾部的 `//# sourceMappingURL=...`
3. `responseHeaders` 中的 `SourceMap` / `X-SourceMap`
4. 与 JS 文件同路径同名的推断 URL

发现 Source Map 时应记录：

- 来源文件
- 映射目标
- 是否可读取
- 是否完整覆盖当前 chunk

## 渲染函数还原模式

常见逆向线索：

| 模式 | 语义 |
| --- | --- |
| `render()` / `h()` / `_createVNode` | DOM 结构主入口 |
| `_renderList(...)` | `v-for` |
| 条件表达式 / 三元表达式 | `v-if` / `v-else` |
| `_withDirectives(...)` | 指令包装 |
| `_toDisplayString(...)` | 文本插值 |
| 事件对象绑定 | `@click` / `@input` 等事件 |

处理原则：

- 先还原结构，再还原语义
- 先页面主骨架，再子组件细节
- 无法确定时保留可读 render 对应 TODO，不要假装完整还原

## Scoped CSS 与 data-v- 规则

Vue SFC 构建后常见 `data-v-xxxx` 作用域标记。处理方式：

1. 找到带该属性的 DOM 结构片段
2. 比对对应 chunk 内部的组件 render 函数
3. 将匹配选择器优先归属到最接近的组件
4. 若同一规则影响多个组件，保守放入局部模块样式

### 归属优先级

1. 组件根节点命中
2. 组件内部唯一类名命中
3. 页面级局部样式命中
4. 无法精确归属时降级为模块级样式

## Vue Hash 文件名还原

### 五步还原

1. 识别入口 chunk
2. 分析 JS 内部结构，提取路由、组件和 store 线索
3. 构建 `chunk-mapping.yaml`
4. 生成 Vue 项目结构
5. 对不确定项执行精度处理

### 精度处理规则

| 情况 | 处理方式 |
| --- | --- |
| 无法确定组件名 | 以路由路径命名 |
| chunk 含多组件 | 以 `defineComponent` / `setup()` 边界拆分 |
| 高度压缩 | 保留骨架并打 TODO |
| CSS 无法拆分 | 暂放模块样式或全局样式目录 |
| 第三方库 | 作为依赖补回，不手写重实现 |

## WebHack 后缀还原

资源后缀 hash 还原正则：`^(.+?)(_[a-z0-9]{4,8})(\.[^.]+)$`

示例：

- `logo_a1b2c3d4.png` → `logo.png`
- `icon_f1e2.svg` → `icon.svg`

说明：

- `main.feba65ae.js` 不属于这一类，它属于构建 hash，需按构建产物规则处理
