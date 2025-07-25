# 响应定义 (Responses)

本目录包含可复用的 HTTP 响应定义，用于在多个 API 端点间共享通用的响应结构。

## 目录结构

```
responses/
├── Problem.yaml    # 错误响应定义
└── README.md       # 本文件
```

## 响应类型

### 错误响应 (Problem.yaml)
定义标准的错误响应格式，用于所有可能出现错误的 API 端点。

```yaml
# Problem.yaml
description: 错误响应
content:
  application/json:
    schema:
      $ref: ../schemas/ErrorResponse.yaml
```

## 设计原则

### 1. 统一格式
所有响应都遵循项目的统一响应格式：
- HTTP 状态码固定为 200
- 通过 `code` 字段区分成功和错误
- 提供清晰的错误消息和详情

### 2. 可复用性
- 设计通用的响应结构
- 支持不同类型的错误场景
- 便于在多个端点间复用

### 3. 详细信息
- 提供具体的错误代码
- 包含人性化的错误消息
- 支持字段级别的错误详情

## 使用方式

### 在路径定义中引用
```yaml
# 在 paths 文件中使用
responses:
  '200':
    description: 操作成功
    content:
      application/json:
        schema:
          $ref: ../components/schemas/AuthApiResponse.yaml
  default:
    $ref: ../components/responses/Problem.yaml
```

### 特定错误响应
```yaml
responses:
  '200':
    # 成功响应使用具体的ApiResponse schema
    description: 操作成功
    content:
      application/json:
        schema:
          $ref: ../components/schemas/SimpleApiResponse.yaml
        example:
          code: 0
          message: "操作成功"
          data: null
          timestamp: 1640995200000
          request_id: "req_123456789"
  default:
    description: 错误响应
    content:
      application/json:
        schema:
          $ref: ../schemas/ErrorResponse.yaml
        example:
          code: 1001
          message: "参数验证失败"
          data: null
          errors:
            - "用户名不能为空"
            - "邮箱格式不正确"
          timestamp: 1640995200000
          request_id: "req_123456789"
```

## 常见响应模板

### 认证类API响应
```yaml
# AuthApiResponse 示例
description: 认证响应
content:
  application/json:
    schema:
      $ref: ../schemas/AuthApiResponse.yaml
    example:
      code: 0
      message: "登录成功"
      data:
        user:
          id: "01ARZ3NDEKTSV4RRFFQ69G5FAV"
          username: "john_doe"
          email: "john@example.com"
        access_token: "eyJhbGciOiJIUzI1NiIs..."
        refresh_token: "eyJhbGciOiJIUzI1NiIs..."
        expires_at: 1641024000
      timestamp: 1640995200000
      request_id: "req_123456789"
```

### 用户信息类API响应
```yaml
# UserApiResponse 示例
description: 用户信息响应
content:
  application/json:
    schema:
      $ref: ../schemas/UserApiResponse.yaml
    example:
      code: 0
      message: "获取用户信息成功"
      data:
        id: "01ARZ3NDEKTSV4RRFFQ69G5FAV"
        username: "john_doe"
        email: "john@example.com"
        status: "active"
        last_login_at: "2024-01-01T10:00:00Z"
        created_at: "2024-01-01T10:00:00Z"
      timestamp: 1640995200000
      request_id: "req_123456789"
```

### 简单操作API响应
```yaml
# SimpleApiResponse 示例
description: 简单操作响应
content:
  application/json:
    schema:
      $ref: ../schemas/SimpleApiResponse.yaml
    example:
      code: 0
      message: "操作成功"
      data: null
      timestamp: 1640995200000
      request_id: "req_123456789"
```

### 错误响应
```yaml
# ErrorResponse 示例
description: 错误响应
content:
  application/json:
    schema:
      $ref: ../schemas/ErrorResponse.yaml
    example:
      code: 40101
      message: "认证失败"
      data: null
      errors: ["访问令牌无效或已过期"]
      timestamp: 1640995200000
      request_id: "req_123456789"
```

## 错误码规范

### 错误码分类
- **0**: 成功
- **40001-49999**: 客户端错误（参数、认证、权限等）
- **50001-59999**: 服务端错误（系统、数据库、第三方服务等）

### 具体错误码示例
```yaml
# 认证相关错误
40101: "邮箱或密码错误"
40102: "访问令牌无效或已过期"
40103: "刷新令牌无效或已过期"
40301: "权限不足"

# 参数相关错误
40001: "参数验证失败"
40002: "缺少必需参数"
40003: "参数格式错误"

# 业务逻辑错误
40901: "用户名已存在"
40902: "邮箱已被注册"
40903: "密码不正确"

# 系统错误
50001: "服务器内部错误"
50002: "数据库连接失败"
50003: "第三方服务不可用"
```

## 扩展响应

### 添加新的响应类型
1. 创建新的 YAML 文件
2. 遵循统一的响应格式
3. 提供清晰的描述和示例
4. 在相关路径中引用

### 示例：分页响应
```yaml
# PaginatedApiResponse.yaml
allOf:
  - $ref: ./SuccessResponse.yaml
  - type: object
    required:
      - data
    properties:
      data:
        type: object
        required:
          - items
          - pagination
        properties:
          items:
            type: array
            description: 分页数据列表
          pagination:
            type: object
            required:
              - page
              - size
              - total
              - total_pages
            properties:
              page:
                type: integer
                description: 当前页码
                minimum: 1
              size:
                type: integer
                description: 每页大小
                minimum: 1
              total:
                type: integer
                description: 总记录数
                minimum: 0
              total_pages:
                type: integer
                description: 总页数
                minimum: 0
```

## 最佳实践

1. **响应统一性**: 所有API都使用相同的响应结构
2. **错误处理**: 在 `errors` 数组中提供详细的错误信息
3. **请求追踪**: 确保每个响应都包含 `request_id` 用于问题追踪
4. **语义化错误码**: 使用有意义的错误码，便于客户端处理
5. **文档示例**: 为每个响应提供完整的示例数据 