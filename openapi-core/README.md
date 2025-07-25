# OpenAPI 规范文件目录

本目录包含项目的完整 OpenAPI 3.1.0 规范定义，采用模块化结构组织，遵循 Redocly 2 最佳实践。

## 目录结构

```
openapi-core/
├── openapi.yaml            # 主规范文件（入口点）
├── components/             # 可复用组件
│   ├── schemas/           # 数据模型定义
│   ├── responses/         # 响应定义
│   └── headers/           # 请求头定义
├── paths/                 # API 路径定义
├── code_samples/          # 多语言代码示例
└── examples/              # 示例文件（如有需要）
```

## 主规范文件 (openapi.yaml)

这是 API 文档的入口点，包含以下重要部分：

### 基本信息 (info)
- **标题和描述**: API 的基本信息和使用说明
- **版本控制**: 遵循语义化版本控制
- **联系信息**: 支持邮箱和许可证信息
- **服务器配置**: 开发和生产环境地址

### 标签和标签组 (tags & x-tagGroups)
使用 `x-tagGroups` 扩展对 API 端点进行逻辑分组：

```yaml
x-tagGroups:
  - name: 核心功能
    tags:
      - 认证管理
      - 当前用户
```

### 路径引用 (paths)
所有 API 端点都通过 `$ref` 引用外部文件：

```yaml
paths:
  /api/v1/auth/login:
    $ref: paths/api_v1_auth_login.yaml
```

## 关键特性

### 1. 统一响应格式

本项目采用统一的 API 响应结构：

```json
{
  "code": 0,              // 0=成功，非0=错误
  "message": "操作成功",   // 响应消息
  "data": {},             // 数据载荷（可选）
  "timestamp": 1234567890 // 时间戳
}
```

**响应类型架构**:

本项目采用分层响应类型设计：

1. **基础响应组件**:
   - `BaseResponse`: 响应基础结构
   - `SuccessResponse`: 成功响应基类
   - `ErrorResponse`: 错误响应基类

2. **具体业务响应**:
   - `AuthApiResponse`: 认证相关API响应（登录、注册、刷新令牌）
   - `UserApiResponse`: 用户信息相关API响应（获取、更新用户信息）
   - `SimpleApiResponse`: 简单操作API响应（登出、修改密码）

**设计优势**:
- HTTP 状态码固定为 200，简化客户端处理
- 通过 `code` 字段进行业务逻辑判断
- 统一的错误处理机制
- 具体响应类型提供类型安全和更好的文档体验

### 2. Bearer Token 认证

使用标准的 Bearer Token 认证机制：

```yaml
components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: 'JWT访问令牌，格式: Bearer `<token>`'
```

### 3. Redocly 扩展支持

充分利用 Redocly 的扩展功能：

- **x-tagGroups**: 侧边栏标签分组
- **x-codeSamples**: 多语言代码示例
- **x-logo**: 自定义 API Logo（如需要）

## 文件组织原则

### 命名约定

1. **路径文件**: 使用下划线替换路径中的斜杠
   - `/api/v1/auth/login` → `api_v1_auth_login.yaml`

2. **Schema 文件**: 使用 PascalCase 命名
   - `UserInfo.yaml`, `AuthApiResponse.yaml`

3. **响应文件**: 使用描述性名称
   - `Problem.yaml`, `SuccessResponse.yaml`

### 模块化设计

- **单一职责**: 每个文件只定义一个概念
- **引用复用**: 大量使用 `$ref` 避免重复
- **层次清晰**: 目录结构反映 API 的逻辑结构

## 最新更新

### 组件架构优化 (2024)

最近进行了组件架构优化，确保所有定义的组件都被正确使用：

- **移除未使用组件**: 删除了通用的 `ApiResponse` 组件
- **具体类型优先**: 统一使用具体的响应类型 (`AuthApiResponse`, `UserApiResponse`, `SimpleApiResponse`)
- **Lint 合规**: 通过所有 Redocly lint 检查，达到质量标准
- **文档更新**: 完善相关文档说明

这些改进提升了API文档的质量和维护性，同时确保更好的开发体验。

## 开发指南

### 添加新的 API 端点

1. **创建路径文件**:
   ```bash
   touch openapi-core/paths/api_v1_new_endpoint.yaml
   ```

2. **定义端点**:
   ```yaml
   post:
     tags:
       - 新功能
     summary: 端点描述
     operationId: NewEndpoint
     requestBody:
       required: true
       content:
         application/json:
           schema:
             $ref: ../components/schemas/NewRequest.yaml
     responses:
       '200':
         description: 操作结果
         content:
           application/json:
             schema:
               # 根据业务类型选择合适的响应类型
               $ref: ../components/schemas/SimpleApiResponse.yaml
   ```

3. **添加到主文件**:
   ```yaml
   # 在 openapi-core/openapi.yaml 的 paths 部分添加
   /api/v1/new-endpoint:
     $ref: paths/api_v1_new_endpoint.yaml
   ```

4. **验证和构建**:
   ```bash
   # 验证规范文件
   npm test
   
   # 重新构建文档
   npm run build
   ```

### 定义数据模型

1. **创建 Schema**:
   ```bash
   touch openapi-core/components/schemas/NewModel.yaml
   ```

2. **定义结构**:
   ```yaml
   type: object
   required:
     - id
     - name
   properties:
     id:
       type: string
       description: 唯一标识符
       example: "123456"
     name:
       type: string
       description: 名称
       maxLength: 100
       example: "示例名称"
   ```

### 最佳实践

#### 1. 描述编写

- **清晰简洁**: 用简单的语言描述功能
- **包含示例**: 为所有字段提供示例值
- **支持 Markdown**: 在描述中使用 Markdown 格式

#### 2. 验证规则

- **数据类型**: 明确指定所有字段的数据类型
- **约束条件**: 添加 minLength、maxLength、pattern 等
- **枚举值**: 使用 enum 限制可选值

#### 3. 引用管理

- **本地引用**: 优先使用相对路径引用
- **组件复用**: 将通用组件放在 components 目录
- **避免循环**: 注意避免循环引用问题

#### 4. Lint 和质量控制

- **避免未使用组件**: 确保所有定义的组件都被实际使用
- **具体类型优先**: 使用具体的响应类型而非通用基类
- **定期清理**: 定期审查和删除未使用的组件
- **自动化验证**: 集成 CI/CD 自动运行 lint 检查

**常见 Lint 错误和解决方案**:
- `no-unused-components`: 删除未使用的组件或确保被正确引用
- `operation-4xx-response`: 项目使用统一200响应，此规则已关闭
- `security-defined`: 确保需要认证的端点正确配置 security

## 验证和测试

### Lint 检查

```bash
# 运行规范验证
npm test

# 详细检查
npx redocly lint openapi-core/openapi.yaml
```

### 预览文档

```bash
# 启动预览服务器
npm start

# 或直接使用 Redocly CLI
npx redocly preview-docs openapi-core/openapi.yaml
```

### 构建静态文档

```bash
# 构建生产文档
npm run build

# 输出: dist/index.html
```

## 配置参考

相关配置文件：

- **[redocly.yaml](../redocly.yaml)**: Redocly CLI 配置
- **[package.json](../package.json)**: 项目脚本和依赖

## 相关文档

- **[组件管理指南](components/README.md)**: 可复用组件的使用说明
- **[路径定义指南](paths/README.md)**: API 端点定义规范
- **[代码示例指南](code_samples/README.md)**: 多语言示例管理

## OpenAPI 规范参考

- [OpenAPI 3.1.0 规范](https://spec.openapis.org/oas/v3.1.0)
- [Redocly 扩展文档](https://redocly.com/docs/api-reference-docs/specification-extensions/)
- [JSON Schema 规范](https://json-schema.org/)
