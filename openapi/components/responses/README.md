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
      allOf:
        - $ref: ../schemas/Response.yaml
        - type: object
          properties:
            code:
              type: integer
              description: 错误码
              example: 1001
            message:
              type: string
              description: 错误消息
              example: "参数验证失败"
            data:
              type: object
              description: 错误详情
              properties:
                errors:
                  type: array
                  description: 具体错误列表
                  items:
                    type: object
                    properties:
                      field:
                        type: string
                        description: 错误字段
                        example: "username"
                      message:
                        type: string
                        description: 字段错误信息
                        example: "用户名不能为空"
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
          allOf:
            - $ref: ../components/schemas/Response.yaml
            - type: object
              properties:
                data:
                  $ref: ../components/schemas/UserInfo.yaml
  default:
    $ref: ../components/responses/Problem.yaml
```

### 特定错误响应
```yaml
responses:
  '200':
    # 成功响应
  '400':
    description: 请求参数错误
    content:
      application/json:
        schema:
          $ref: ../components/schemas/Response.yaml
        example:
          code: 1001
          message: "参数验证失败"
          data:
            errors:
              - field: "username"
                message: "用户名不能为空"
              - field: "email"
                message: "邮箱格式不正确"
          timestamp: 1640995200000
  '401':
    description: 未认证
    content:
      application/json:
        schema:
          $ref: ../components/schemas/Response.yaml
        example:
          code: 401
          message: "访问令牌无效或已过期"
          timestamp: 1640995200000
```

## 常见响应模板

### 成功响应
```yaml
# SuccessResponse.yaml
description: 操作成功
content:
  application/json:
    schema:
      $ref: ../schemas/Response.yaml
    example:
      code: 0
      message: "操作成功"
      timestamp: 1640995200000
```

### 认证错误
```yaml
# UnauthorizedResponse.yaml
description: 认证失败
content:
  application/json:
    schema:
      $ref: ../schemas/Response.yaml
    example:
      code: 401
      message: "访问令牌无效或已过期"
      timestamp: 1640995200000
```

### 权限错误
```yaml
# ForbiddenResponse.yaml
description: 权限不足
content:
  application/json:
    schema:
      $ref: ../schemas/Response.yaml
    example:
      code: 403
      message: "权限不足，无法访问此资源"
      timestamp: 1640995200000
```

### 资源不存在
```yaml
# NotFoundResponse.yaml
description: 资源不存在
content:
  application/json:
    schema:
      $ref: ../schemas/Response.yaml
    example:
      code: 404
      message: "请求的资源不存在"
      timestamp: 1640995200000
```

## 错误码规范

### 错误码分类
- **0**: 成功
- **1xxx**: 客户端错误（参数、认证、权限等）
- **2xxx**: 服务端错误（系统、数据库、第三方服务等）

### 具体错误码示例
```yaml
# 认证相关错误
401: "访问令牌无效或已过期"
402: "刷新令牌无效或已过期"
403: "权限不足"

# 参数相关错误
1001: "参数验证失败"
1002: "缺少必需参数"
1003: "参数格式错误"

# 业务逻辑错误
1101: "用户名已存在"
1102: "邮箱已被注册"
1103: "密码不正确"

# 系统错误
2001: "服务器内部错误"
2002: "数据库连接失败"
2003: "第三方服务不可用"
```

## 扩展响应

### 添加新的响应类型
1. 创建新的 YAML 文件
2. 遵循统一的响应格式
3. 提供清晰的描述和示例
4. 在相关路径中引用

### 示例：分页响应
```yaml
# PaginatedResponse.yaml
description: 分页响应
content:
  application/json:
    schema:
      allOf:
        - $ref: ../schemas/Response.yaml
        - type: object
          properties:
            data:
              type: object
              properties:
                items:
                  type: array
                  description: 数据列表
                  items:
                    type: object
                pagination:
                  type: object
                  description: 分页信息
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

## 最佳实践

### 1. 一致性
- 保持所有响应格式的一致性
- 使用统一的错误码体系
- 提供清晰的错误消息

### 2. 详细性
- 包含足够的错误详情
- 支持国际化的错误消息
- 提供问题修复建议

### 3. 可维护性
- 集中管理通用响应
- 避免重复定义
- 定期审查和更新

## 验证和测试

```bash
# 验证响应定义
npx redocly lint openapi/components/responses/Problem.yaml

# 验证完整规范
npx redocly lint openapi/openapi.yaml
```

## 相关文档

- **[数据模型指南](../schemas/README.md)**: 响应中使用的数据模型
- **[组件管理指南](../README.md)**: 完整的组件管理说明
- **[路径定义指南](../../paths/README.md)**: 如何在路径中使用响应

---

> 💡 **提示**: 良好的响应定义能提供一致的错误处理体验，帮助开发者快速理解和处理 API 错误。 