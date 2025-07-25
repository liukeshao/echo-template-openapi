# API响应模式使用指南

本项目实现了基于 OpenAPI 3.1 的统一响应格式，HTTP 状态码始终为 200，通过响应体中的 `code` 字段判断业务状态。

## 核心概念

### 响应格式设计原则

1. **HTTP 状态码固定为 200**：所有 API 响应的 HTTP 状态码都是 200
2. **通过 code 字段区分状态**：
   - `code = 0`：请求成功
   - `code != 0`：请求失败，具体错误码表示不同的错误类型
3. **data 字段结构根据成功/失败而不同**：
   - 成功时：`data` 包含实际的业务数据
   - 失败时：`data` 固定为 `null`

## Schema 结构层次

### 基础组件

```
BaseResponse.yaml              # 所有响应的公共字段
├── SuccessResponse.yaml       # 成功响应基础 (code=0)
└── ErrorResponse.yaml         # 错误响应 (code!=0)
```

### 具体业务场景

```
AuthApiResponse.yaml           # 认证相关API的统一响应
├── AuthSuccessResponse.yaml   # 认证成功响应 (包含用户信息+令牌)
└── ErrorResponse.yaml         # 认证失败响应

UserApiResponse.yaml           # 用户信息相关API的统一响应
├── UserSuccessResponse.yaml   # 用户信息成功响应 (包含用户详情)
└── ErrorResponse.yaml         # 用户信息获取失败响应

SimpleApiResponse.yaml         # 简单操作的统一响应
├── SimpleSuccessResponse.yaml # 简单操作成功响应 (data为null)
└── ErrorResponse.yaml         # 简单操作失败响应
```

## 使用示例

### 1. 认证相关API（如登录、注册）

使用 `AuthApiResponse.yaml`：

```yaml
responses:
  '200':
    description: 登录结果
    content:
      application/json:
        schema:
          $ref: ../components/schemas/AuthApiResponse.yaml
```

**成功响应示例**：
```json
{
  "code": 0,
  "message": "登录成功",
  "data": {
    "user": {
      "id": "01ARZ3NDEKTSV4RRFFQ69G5FAV",
      "username": "john_doe",
      "email": "john@example.com"
    },
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_at": 1641024000
  },
  "timestamp": 1641024000,
  "request_id": "req_123456789"
}
```

**失败响应示例**：
```json
{
  "code": 40101,
  "message": "邮箱或密码错误",
  "data": null,
  "errors": ["邮箱或密码不正确"],
  "timestamp": 1641024000,
  "request_id": "req_123456789"
}
```

### 2. 用户信息相关API

使用 `UserApiResponse.yaml`：

```yaml
responses:
  '200':
    content:
      application/json:
        schema:
          $ref: ../components/schemas/UserApiResponse.yaml
```

### 3. 简单操作API（如登出、修改密码）

使用 `SimpleApiResponse.yaml`：

```yaml
responses:
  '200':
    content:
      application/json:
        schema:
          $ref: ../components/schemas/SimpleApiResponse.yaml
```

## discriminator 工作原理

OpenAPI 3.1 使用 `discriminator` 基于 `code` 字段的值来确定具体的响应schema：

```yaml
oneOf:
  - $ref: ./AuthSuccessResponse.yaml    # 当 code = 0 时
  - $ref: ./ErrorResponse.yaml          # 当 code != 0 时
discriminator:
  propertyName: code
  mapping:
    0: '#/components/schemas/AuthSuccessResponse'
```

## 错误码约定

- `0`：成功
- `40001-49999`：客户端错误（4xxxx）
- `50001-59999`：服务端错误（5xxxx）
- `40101`：认证失败
- `40001`：参数验证失败
- `40301`：权限不足

## 最佳实践

1. **新API开发**：始终使用ApiResponse模式的schema
2. **错误处理**：在 `errors` 数组中提供详细的错误信息
3. **请求追踪**：确保每个响应都包含 `request_id` 用于问题追踪
4. **文档示例**：为每个API提供成功和失败的响应示例 