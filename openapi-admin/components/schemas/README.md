# Admin API响应模式使用指南

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
用户管理相关：
UserApiResponse.yaml           # 单个用户信息API的统一响应
├── UserSuccessResponse.yaml   # 用户信息成功响应
└── ErrorResponse.yaml         # 用户信息获取失败响应

UserListApiResponse.yaml       # 用户列表API的统一响应
├── UserListSuccessResponse.yaml # 用户列表成功响应
└── ErrorResponse.yaml         # 用户列表获取失败响应

部门管理相关：
DepartmentApiResponse.yaml     # 单个部门信息API的统一响应
DepartmentListApiResponse.yaml # 部门列表API的统一响应
DepartmentTreeApiResponse.yaml # 部门树形结构API的统一响应
DepartmentStatsApiResponse.yaml # 部门统计信息API的统一响应

菜单管理相关：
MenuApiResponse.yaml           # 单个菜单信息API的统一响应
MenuListApiResponse.yaml       # 菜单列表API的统一响应
MenuTreeApiResponse.yaml       # 菜单树形结构API的统一响应

角色管理相关：
RoleApiResponse.yaml           # 单个角色信息API的统一响应
RoleListApiResponse.yaml       # 角色列表API的统一响应

职位管理相关：
PositionApiResponse.yaml       # 职位信息API的统一响应

通用操作：
SimpleApiResponse.yaml         # 简单操作的统一响应
├── SimpleSuccessResponse.yaml # 简单操作成功响应 (data为null)
└── ErrorResponse.yaml         # 简单操作失败响应

CheckDeletableApiResponse.yaml # 检查删除权限API的统一响应
├── CheckDeletableSuccessResponse.yaml # 检查删除权限成功响应
└── ErrorResponse.yaml         # 检查删除权限失败响应
```

## 使用示例

### 1. 用户信息相关API

使用 `UserApiResponse.yaml`：

```yaml
responses:
  '200':
    description: 用户信息
    content:
      application/json:
        schema:
          $ref: ../components/schemas/UserApiResponse.yaml
```

### 2. 用户列表API

使用 `UserListApiResponse.yaml`：

```yaml
responses:
  '200':
    description: 用户列表
    content:
      application/json:
        schema:
          $ref: ../components/schemas/UserListApiResponse.yaml
```

### 3. 简单操作API（如删除、更新）

使用 `SimpleApiResponse.yaml`：

```yaml
responses:
  '200':
    description: 操作结果
    content:
      application/json:
        schema:
          $ref: ../components/schemas/SimpleApiResponse.yaml
```

### 4. 检查删除权限API

使用 `CheckDeletableApiResponse.yaml`：

```yaml
responses:
  '200':
    description: 删除权限检查结果
    content:
      application/json:
        schema:
          $ref: ../components/schemas/CheckDeletableApiResponse.yaml
```

## discriminator 工作原理

OpenAPI 3.1 使用 `discriminator` 基于 `code` 字段的值来确定具体的响应schema：

```yaml
oneOf:
  - $ref: ./UserSuccessResponse.yaml    # 当 code = 0 时
  - $ref: ./ErrorResponse.yaml          # 当 code != 0 时
discriminator:
  propertyName: code
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
5. **选择合适的响应模式**：
   - 返回单个实体：使用对应的ApiResponse（如UserApiResponse）
   - 返回列表：使用对应的ListApiResponse（如UserListApiResponse）
   - 简单操作：使用SimpleApiResponse
   - 检查权限：使用CheckDeletableApiResponse