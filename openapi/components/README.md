# 可复用组件 (Components)

本目录包含 OpenAPI 规范中的所有可复用组件，遵循 DRY（Don't Repeat Yourself）原则，提高文档的可维护性和一致性。

## 目录结构

```
components/
├── schemas/           # 数据模型定义
│   ├── AuthOutput.yaml      # 认证输出模型
│   ├── AuthApiResponse.yaml # 认证响应模型
│   ├── LoginInput.yaml      # 登录输入模型
│   ├── RegisterInput.yaml   # 注册输入模型
│   ├── UserInfo.yaml        # 用户信息模型
│   ├── UserApiResponse.yaml # 用户响应模型
│   ├── SimpleApiResponse.yaml # 简单操作响应模型
│   └── ...
├── responses/         # 响应定义
│   └── Problem.yaml         # 错误响应模型
├── headers/           # 请求头定义
│   └── ExpiresAfter.yaml    # 过期时间头
└── README.md          # 本文件
```

## 组件类型说明

### 1. Schemas (数据模型)

存储所有的数据结构定义，包括请求体、响应体和数据传输对象。

#### 命名约定

- 使用 **PascalCase** 命名
- 名称应清晰表达模型用途
- 输入模型使用 `Input` 后缀
- 输出模型使用 `Output` 或 `Response` 后缀

#### 示例结构

```yaml
# schemas/UserInfo.yaml
type: object
description: 用户基本信息
required:
  - id
  - username
  - email
properties:
  id:
    type: string
    description: 用户唯一标识符
    example: "user_123456"
  username:
    type: string
    description: 用户名
    minLength: 3
    maxLength: 20
    pattern: '^[a-zA-Z0-9_]+$'
    example: "john_doe"
  email:
    type: string
    format: email
    description: 用户邮箱地址
    example: "john@example.com"
  createdAt:
    type: string
    format: date-time
    description: 创建时间
    readOnly: true
    example: "2023-01-01T00:00:00Z"
```

### 2. Responses (响应定义)

定义标准化的 HTTP 响应结构，可在多个端点间复用。

#### 统一响应格式

本项目使用统一的响应包装器：

```yaml
# schemas/BaseResponse.yaml
type: object
description: 统一响应格式
required:
  - code
  - message
  - timestamp
properties:
  code:
    type: integer
    description: 响应码，0表示成功，非0表示错误
    example: 0
  message:
    type: string
    description: 响应消息
    example: "操作成功"
  data:
    description: 响应数据，根据具体接口而定
  timestamp:
    type: integer
    format: int64
    description: 响应时间戳
    example: 1640995200000
```

#### 错误响应示例

```yaml
# responses/Problem.yaml
description: 错误响应
content:
  application/json:
    schema:
      allOf:
        - $ref: ../schemas/BaseResponse.yaml
        - type: object
          properties:
            code:
              example: 1001
            message:
              example: "参数验证失败"
            data:
              type: object
              properties:
                errors:
                  type: array
                  items:
                    type: object
                    properties:
                      field:
                        type: string
                        example: "username"
                      message:
                        type: string
                        example: "用户名不能为空"
```

### 3. Headers (请求头定义)

定义可复用的 HTTP 头部。

```yaml
# headers/ExpiresAfter.yaml
description: 令牌过期时间（秒）
schema:
  type: integer
  minimum: 1
  maximum: 86400
  example: 7200
```

### 4. Parameters (参数定义)

定义可复用的查询参数、路径参数等。

```yaml
# parameters/PageInput.yaml
name: page
in: query
description: 页码，从1开始
required: false
schema:
  type: integer
  minimum: 1
  default: 1
  example: 1
```

## 使用指南

### 引用组件

在其他文件中使用 `$ref` 引用组件：

```yaml
# 在 paths 文件中引用
responses:
  '200':
    description: 登录成功
    content:
      application/json:
        schema:
          $ref: ../components/schemas/AuthApiResponse.yaml
```

### 组合模式

#### allOf 组合

用于扩展基础模型：

```yaml
# 扩展基础响应模型
allOf:
  - $ref: ../schemas/BaseResponse.yaml
  - type: object
    properties:
      data:
        $ref: ../schemas/UserInfo.yaml
```

#### oneOf 选择

用于表示多种可能的数据类型：

```yaml
# 多种输入类型
oneOf:
  - $ref: ../schemas/LoginInput.yaml
  - $ref: ../schemas/RegisterInput.yaml
```

#### anyOf 组合

用于表示可以是多种类型的组合：

```yaml
anyOf:
  - $ref: ../schemas/UserInfo.yaml
  - $ref: ../schemas/AdminInfo.yaml
```

## 最佳实践

### 1. 设计原则

- **单一职责**: 每个组件只定义一个概念
- **可复用性**: 设计通用的、可在多处使用的组件
- **一致性**: 保持命名和结构的一致性
- **可扩展性**: 为未来的扩展留有空间

### 2. 验证规则

为所有字段添加适当的验证：

```yaml
properties:
  email:
    type: string
    format: email           # 邮箱格式验证
    description: 用户邮箱
  password:
    type: string
    minLength: 8            # 最小长度
    maxLength: 128          # 最大长度
    pattern: '^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).*$'  # 正则验证
    description: 密码，至少8位，包含大小写字母和数字
  age:
    type: integer
    minimum: 0             # 最小值
    maximum: 120           # 最大值
    description: 用户年龄
```

### 3. 示例数据

为每个字段提供有意义的示例：

```yaml
properties:
  username:
    type: string
    example: "john_doe"     # 单个示例
  status:
    type: string
    enum: ["active", "inactive", "pending"]
    examples:               # 多个示例
      active:
        value: "active"
        summary: "活跃用户"
      inactive:
        value: "inactive"
        summary: "非活跃用户"
```

### 4. 文档描述

- 使用清晰、简洁的中文描述
- 支持 Markdown 格式
- 包含使用说明和注意事项

```yaml
description: |
  用户登录输入模型
  
  **注意事项**:
  - 用户名或邮箱都可以用于登录
  - 密码长度至少8位
  - 支持记住登录状态
```

### 5. 版本管理

当需要更新组件时：

1. **向后兼容**: 优先考虑向后兼容的修改
2. **新增版本**: 如需破坏性变更，创建新版本
3. **废弃标记**: 使用 `deprecated: true` 标记废弃的组件

```yaml
# 废弃示例
UserInfoV1:
  deprecated: true
  description: |
    ⚠️ **已废弃**: 请使用 UserInfo 替代
    
    此模型将在下个版本中移除。
```

## 常见模式

### 分页响应

```yaml
# schemas/PageOutput.yaml
type: object
description: 分页响应数据
properties:
  items:
    type: array
    description: 数据列表
    items:
      type: object
  pagination:
    type: object
    properties:
      page:
        type: integer
        description: 当前页码
        example: 1
      size:
        type: integer
        description: 每页大小
        example: 20
      total:
        type: integer
        description: 总记录数
        example: 100
      pages:
        type: integer
        description: 总页数
        example: 5
```

### 时间戳字段

```yaml
# schemas/TimestampMixin.yaml
type: object
properties:
  createdAt:
    type: string
    format: date-time
    description: 创建时间
    readOnly: true
    example: "2023-01-01T00:00:00Z"
  updatedAt:
    type: string
    format: date-time
    description: 更新时间
    readOnly: true
    example: "2023-01-01T00:00:00Z"
```

## 工具支持

### 验证命令

```bash
# 验证组件定义
npx redocly lint openapi/openapi.yaml

# 只验证特定组件
npx redocly lint openapi/components/schemas/UserInfo.yaml
```

### 生成工具

可以使用以下工具生成组件：

- **JSON to Schema**: [Redocly JSON to JSON Schema](https://redocly.com/tools/json-to-json-schema/)
- **Schema Generator**: 各种语言的 Schema 生成工具

## 参考资源

- [OpenAPI 3.1.0 Components 规范](https://spec.openapis.org/oas/v3.1.0#components-object)
- [JSON Schema 规范](https://json-schema.org/)
- [Redocly 组件最佳实践](https://redocly.com/docs/api-reference-docs/spec-components/)

---

> 💡 **提示**: 设计组件时，优先考虑复用性和可维护性。一个好的组件设计可以大大减少重复代码，提高文档质量。
