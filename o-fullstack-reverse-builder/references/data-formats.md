# 数据源格式与处理

## 导出目录标准结构

```
{项目名}-export/
├── package.json / server.js / routes.js   # Mock Server
├── records.json                           # API 请求/响应元数据（核心）
├── url-map.json                           # 原始URL→本地路径映射
├── api-summary.md / file-structure.md     # 文档
├── assets/{domain}/                       # 静态文件
├── dto/index.ts                           # TS DTO 接口定义
└── responses/*.json|*.bin                 # API 响应体
```

## records.json 关键字段

| 字段 | 含义 | 逆向用途 |
|---|---|---|
| `method` | HTTP 方法 | Controller 方法类型 |
| `url` | 完整请求 URL | 路径 + 参数定义 |
| `requestBody` | 请求体 JSON | 推断 Request DTO |
| `statusCode` | 响应状态码 | 判断有效性 |
| `responseFile` | 响应体文件路径 | 读取真实响应推断 DTO |
| `resourceType` | xhr/document/script/stylesheet | 区分 API 和静态资源 |
| `contentType` | 响应内容类型 | 返回值序列化方式 |
| `requestHeaders` | 请求头 | 提取认证方式 |
| `responseHeaders` | 响应头 | CORS、缓存策略 |

## url-map.json resourceType 枚举

| 值 | 含义 | 前端处理 | 后端处理 |
|---|---|---|---|
| `script` | JS | 分析 Vue 组件/路由/状态 | 分析 API 调用端点 |
| `document` | HTML | 提取 SPA 入口 | 配置页面路由 |
| `stylesheet` | CSS | 提取样式变量/主题 | 静态资源服务 |
| `image`/`font`/`media` | 静态资源 | 迁移到 `src/assets/` | 静态资源服务 |

## dto/index.ts 命名规则

`Api` + 路径各段 PascalCase + `Response`

示例：`ApiPublicSystemConfigResponse`

## responses/ .bin 文件处理

| Content-Type | 处理方式 |
|---|---|
| `text/html` | 跳过，不纳入路由图 |
| `application/octet-stream` | Controller 返回 `FileContentResult` |
| `image/*`/`font/*` | 迁移到 `assets/` |
| `application/pdf`/`zip` | 文件下载端点 |

## Vue Hash 文件名还原

### 问题

Vue 构建产物使用 content-hash 文件名，需还原为源码结构。

### 五步还原

1. **识别入口**：从 index.html 提取入口 chunk
2. **分析 JS 内部结构**：提取 Router 路由定义、chunk 注释、defineComponent
3. **构建映射表**：输出 `chunk-mapping.yaml`
4. **生成 Vue 项目结构**
5. **精度处理**：无法确定组件名时以路由路径命名

### 精度处理规则

| 情况 | 处理方式 |
|---|---|
| 无法确定组件名 | 以路由路径命名：`/user/list` → `UserList.vue` |
| chunk 含多组件 | 按 `defineComponent`/`setup()` 边界拆分 |
| 高度压缩 | 保留骨架 + TODO 标记 |
| CSS 无法拆分 | `src/assets/styles/` 全局样式 |
| 第三方库 | `package.json` 添加依赖 |

## WebHack 后缀还原

还原正则：`/^(.+?)(_[a-z0-9]{4,8})(\.[^.]+)$/`

- 匹配：`logo_a1b2c3d4.png` → `logo.png`
- 不匹配：`main.feba65ae.js`（构建 hash 用 `.` 分隔）
