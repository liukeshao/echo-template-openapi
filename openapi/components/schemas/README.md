# 数据模型 (Schemas)

本目录包含所有 API 数据模型的定义，这些模型用于描述请求体、响应体和数据传输对象的结构。

## 目录结构

```
schemas/
├── AuthOutput.yaml           # 认证输出模型
├── AuthResponse.yaml         # 认证响应模型  
├── ChangePasswordInput.yaml  # 修改密码输入模型
├── LoginInput.yaml           # 登录输入模型
├── PageInput.yaml            # 分页输入模型
├── PageOutput.yaml           # 分页输出模型
├── RefreshTokenInput.yaml    # 刷新令牌输入模型
├── RegisterInput.yaml        # 注册输入模型
├── Response.yaml             # 通用响应模型
├── SuccessResponse.yaml      # 成功响应模型
├── UpdateEmailInput.yaml     # 更新邮箱输入模型
├── UpdateUsernameInput.yaml  # 更新用户名输入模型
├── UserInfo.yaml             # 用户信息模型
├── UserResponse.yaml         # 用户响应模型
└── README.md                 # 本文件
```

## 模型分类

### 输入模型 (Input)
用于定义 API 请求体的数据结构：
- `LoginInput.yaml` - 用户登录数据
- `RegisterInput.yaml` - 用户注册数据
- `ChangePasswordInput.yaml` - 修改密码数据
- `UpdateEmailInput.yaml` - 更新邮箱数据
- `UpdateUsernameInput.yaml` - 更新用户名数据
- `RefreshTokenInput.yaml` - 刷新令牌数据
- `PageInput.yaml` - 分页查询参数

### 输出模型 (Output/Response)
用于定义 API 响应体的数据结构：
- `AuthOutput.yaml` - 认证成功返回数据
- `AuthResponse.yaml` - 认证响应包装
- `UserInfo.yaml` - 用户基本信息
- `UserResponse.yaml` - 用户响应包装
- `PageOutput.yaml` - 分页响应数据

### 通用模型
可在多个场景中复用的基础模型：
- `Response.yaml` - 统一响应格式
- `SuccessResponse.yaml` - 成功响应模板

## 设计原则

### 1. 命名规范
- **PascalCase**: 所有文件名使用大写开头的驼峰命名
- **语义明确**: 名称能清晰表达模型的用途
- **后缀约定**: 
  - 输入模型使用 `Input` 后缀
  - 输出模型使用 `Output` 或 `Response` 后缀

### 2. 结构设计
- **单一职责**: 每个模型只定义一个明确的数据结构
- **组合优先**: 使用 `allOf`、`oneOf` 等组合现有模型
- **扩展性**: 为未来扩展预留空间

### 3. 验证规则
- **类型明确**: 为所有字段指定明确的数据类型
- **约束完整**: 添加长度、格式、枚举等约束
- **示例丰富**: 提供真实有效的示例数据

## 模型示例

### 基础响应模型
```yaml
# Response.yaml
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

### 用户信息模型
```yaml
# UserInfo.yaml
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

## 使用方式

### 在路径中引用
```yaml
# 在 paths 文件中使用
requestBody:
  content:
    application/json:
      schema:
        $ref: ../components/schemas/LoginInput.yaml

responses:
  '200':
    content:
      application/json:
        schema:
          allOf:
            - $ref: ../components/schemas/Response.yaml
            - type: object
              properties:
                data:
                  $ref: ../components/schemas/UserInfo.yaml
```

### 模型组合
```yaml
# 组合现有模型创建新的响应
allOf:
  - $ref: ./Response.yaml
  - type: object
    properties:
      data:
        type: array
        items:
          $ref: ./UserInfo.yaml
```

## 最佳实践

### 1. 字段定义
```yaml
properties:
  email:
    type: string
    format: email          # 使用标准格式
    description: 用户邮箱   # 提供清晰描述
    example: "user@example.com"  # 提供示例
  
  password:
    type: string
    minLength: 8           # 最小长度限制
    maxLength: 128         # 最大长度限制
    pattern: '^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).*$'  # 正则验证
    description: 密码，至少8位，包含大小写字母和数字
    writeOnly: true        # 只写字段，不在响应中返回
```

### 2. 枚举值定义
```yaml
status:
  type: string
  enum: ["active", "inactive", "pending"]
  description: 用户状态
  example: "active"
```

### 3. 数组和对象
```yaml
tags:
  type: array
  description: 用户标签列表
  items:
    type: string
    example: "developer"
  example: ["developer", "backend"]

address:
  type: object
  properties:
    street:
      type: string
      example: "123 Main St"
    city:
      type: string
      example: "New York"
```

## 维护指南

### 添加新模型
1. 创建新的 YAML 文件
2. 遵循命名规范
3. 添加完整的类型定义和验证规则
4. 提供示例数据
5. 更新相关文档

### 修改现有模型
1. 优先考虑向后兼容
2. 新增字段设为可选
3. 废弃字段标记 `deprecated: true`
4. 更新相关的路径定义

### 验证检查
```bash
# 验证特定模型
npx redocly lint openapi/components/schemas/UserInfo.yaml

# 验证所有模型
npx redocly lint openapi/openapi.yaml
```

## 相关文档

- **[组件管理指南](../README.md)**: 完整的组件管理说明
- **[路径定义指南](../../paths/README.md)**: 如何在路径中使用模型
- **[OpenAPI 规范指南](../../README.md)**: 项目整体架构

---

> 💡 **提示**: 良好的数据模型设计是 API 文档质量的基础。投入时间设计清晰、一致的数据结构将极大提升 API 的可用性。 