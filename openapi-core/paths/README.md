# API 路径定义 (Paths)

本目录包含所有 API 端点的详细定义，每个文件对应一个 API 路径，采用模块化管理，便于维护和扩展。

## 目录结构

```
paths/
├── api_v1_auth_login.yaml       # POST /api/v1/auth/login - 用户登录
├── api_v1_auth_logout.yaml      # POST /api/v1/auth/logout - 用户登出
├── api_v1_auth_refresh.yaml     # POST /api/v1/auth/refresh - 刷新令牌
├── api_v1_auth_register.yaml    # POST /api/v1/auth/register - 用户注册
├── api_v1_me.yaml               # GET /api/v1/me - 获取当前用户信息
├── api_v1_me_change-password.yaml  # PUT /api/v1/me/change-password - 修改密码
├── api_v1_me_email.yaml         # PUT /api/v1/me/email - 更新邮箱
├── api_v1_me_username.yaml      # PUT /api/v1/me/username - 更新用户名
└── README.md                    # 本文件
```

## 文件命名规范

### 命名约定

路径文件采用以下命名规则：

1. **路径转换**: 将 URL 路径中的 `/` 替换为 `_`
2. **小写字母**: 全部使用小写字母
3. **连字符保留**: 保留原有的连字符 `-`
4. **扩展名**: 使用 `.yaml` 扩展名

### 映射示例

```
API 路径                    → 文件名
/api/v1/auth/login         → api_v1_auth_login.yaml
/api/v1/auth/logout        → api_v1_auth_logout.yaml
/api/v1/me                 → api_v1_me.yaml
/api/v1/me/change-password → api_v1_me_change-password.yaml
/api/v1/users/{id}         → api_v1_users_{id}.yaml
/api/v1/orders/{orderId}/items → api_v1_orders_{orderId}_items.yaml
```

## 文件结构模板

### 基本结构

每个路径文件应包含以下基本结构：

```yaml
# 方法定义（可以包含多个 HTTP 方法）
get:
  tags:
    - 标签名称
  summary: 端点简短描述
  description: |
    详细描述，支持 Markdown 格式
    
    可以包含：
    - 使用说明
    - 注意事项
    - 示例场景
  operationId: UniqueOperationId
  parameters:
    - name: 参数名
      in: query|path|header
      description: 参数描述
      required: true|false
      schema:
        type: string
        example: "示例值"
  requestBody:
    $ref: ../components/requestBodies/RequestBodyName.yaml
  responses:
    '200':
      $ref: ../components/responses/SuccessResponse.yaml
    '400':
      $ref: ../components/responses/Problem.yaml
  security:
    - bearerAuth: []
  x-codeSamples:
    - lang: curl
      source:
        $ref: ../code_samples/curl/path/method.sh
```

## 实际示例

### 用户登录端点

```yaml
# api_v1_auth_login.yaml
post:
  tags:
    - 认证管理
  summary: 用户登录
  description: |
    用户通过用户名/邮箱和密码进行登录认证
    
    **功能特性**:
    - 支持用户名或邮箱登录
    - 支持记住登录状态
    - 返回访问令牌和刷新令牌
    
    **注意事项**:
    - 密码错误超过5次将锁定账户30分钟
    - 令牌默认有效期为2小时
  operationId: LoginUser
  requestBody:
    description: 登录凭据
    required: true
    content:
      application/json:
        schema:
          $ref: ../components/schemas/LoginInput.yaml
        examples:
          username_login:
            summary: 用户名登录
            value:
              username: "john_doe"
              password: "password123"
              remember: false
          email_login:
            summary: 邮箱登录
            value:
              username: "john@example.com"
              password: "password123"
              remember: true
  responses:
    '200':
      description: 登录成功
      content:
        application/json:
          schema:
            $ref: ../components/schemas/AuthApiResponse.yaml
          examples:
            success:
              summary: 登录成功示例
              value:
                code: 0
                message: "登录成功"
                data:
                  access_token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
                  refresh_token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
                  expires_in: 7200
                  user_info:
                    id: "user_123456"
                    username: "john_doe"
                    email: "john@example.com"
                timestamp: 1640995200000
    default:
      $ref: ../components/responses/Problem.yaml
  x-codeSamples:
    - lang: curl
      label: cURL
      source:
        $ref: ../code_samples/curl/auth/login.sh
    - lang: javascript
      label: JavaScript
      source:
        $ref: ../code_samples/javascript/auth/login.js
    - lang: python
      label: Python
      source:
        $ref: ../code_samples/python/auth/login.py
```

### 获取用户信息端点

```yaml
# api_v1_me.yaml
get:
  tags:
    - 当前用户
  summary: 获取当前用户信息
  description: |
    获取当前登录用户的详细信息
    
    **返回信息**:
    - 基本用户信息（用户名、邮箱等）
    - 账户状态
    - 创建和更新时间
  operationId: GetCurrentUser
  security:
    - bearerAuth: []
  responses:
    '200':
      description: 获取用户信息成功
      content:
        application/json:
          schema:
            $ref: ../components/schemas/UserApiResponse.yaml
    default:
      $ref: ../components/responses/Problem.yaml
  x-codeSamples:
    - lang: curl
      source: |
        curl -X GET "http://localhost:8000/api/v1/me" \
          -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
    - lang: javascript
      source: |
        fetch('http://localhost:8000/api/v1/me', {
          headers: {
            'Authorization': 'Bearer ' + localStorage.getItem('access_token')
          }
        })
        .then(response => response.json())
        .then(data => console.log(data));
```

## 设计最佳实践

### 1. 标签管理

使用标签对 API 进行逻辑分组：

```yaml
tags:
  - 认证管理    # 登录、注册、令牌相关
  - 当前用户    # 用户信息管理
  - 用户管理    # 管理员功能
  - 订单管理    # 订单相关操作
```

### 2. 操作 ID 规范

使用清晰、一致的 `operationId`：

```yaml
# 命名规范: <动作><资源>
operationId: GetCurrentUser     # 获取当前用户
operationId: UpdateUserProfile  # 更新用户资料
operationId: CreateOrder        # 创建订单
operationId: DeleteOrder        # 删除订单
operationId: ListOrders         # 列出订单
```

### 3. 参数定义

#### 查询参数

```yaml
parameters:
  - name: page
    in: query
    description: 页码，从1开始
    required: false
    schema:
      type: integer
      minimum: 1
      default: 1
      example: 1
  - name: size
    in: query
    description: 每页大小
    required: false
    schema:
      type: integer
      minimum: 1
      maximum: 100
      default: 20
      example: 20
```

#### 路径参数

```yaml
parameters:
  - name: id
    in: path
    description: 用户唯一标识符
    required: true
    schema:
      type: string
      pattern: '^[a-zA-Z0-9_-]+$'
      example: "user_123456"
```

#### 请求头参数

```yaml
parameters:
  - name: X-Request-ID
    in: header
    description: 请求追踪标识符
    required: false
    schema:
      type: string
      format: uuid
      example: "123e4567-e89b-12d3-a456-426614174000"
```

### 4. 响应设计

#### 成功响应

```yaml
responses:
  '200':
    description: 操作成功
    headers:
      X-RateLimit-Remaining:
        description: 剩余请求次数
        schema:
          type: integer
          example: 99
    content:
      application/json:
        schema:
          $ref: ../components/schemas/UserApiResponse.yaml
```

#### 错误响应

统一使用错误响应组件：

```yaml
responses:
  '400':
    $ref: ../components/responses/Problem.yaml
  '401':
    description: 未认证
    content:
      application/json:
        schema:
          $ref: ../components/schemas/ErrorResponse.yaml
        example:
          code: 401
          message: "访问令牌无效或已过期"
          timestamp: 1640995200000
```

### 5. 安全配置

#### 认证要求

```yaml
security:
  - bearerAuth: []  # 需要 Bearer Token
```

#### 可选认证

```yaml
security:
  - bearerAuth: []  # 需要认证
  - {}              # 或者无需认证
```

### 6. 示例数据

提供丰富的示例数据：

```yaml
requestBody:
  content:
    application/json:
      schema:
        $ref: ../components/schemas/CreateUserInput.yaml
      examples:
        basic_user:
          summary: 基本用户
          description: 创建基本用户的示例
          value:
            username: "john_doe"
            email: "john@example.com"
            password: "password123"
        admin_user:
          summary: 管理员用户
          description: 创建管理员用户的示例
          value:
            username: "admin"
            email: "admin@example.com"
            password: "admin123"
            role: "admin"
```

## 验证和测试

### Lint 检查

```bash
# 验证特定路径文件
npx redocly lint openapi/paths/api_v1_auth_login.yaml

# 验证所有路径文件
npx redocly lint openapi/openapi.yaml
```

### 常见错误检查

1. **操作 ID 重复**: 确保每个操作有唯一的 `operationId`
2. **引用错误**: 检查所有 `$ref` 引用是否正确
3. **参数冲突**: 避免同名参数在同一位置重复定义
4. **响应缺失**: 至少定义一个成功响应

## 版本管理

### 向后兼容性

- 新增字段时设为可选
- 废弃字段使用 `deprecated: true`
- 重大变更时增加版本号

### 废弃标记

```yaml
parameters:
  - name: old_param
    deprecated: true
    description: |
      ⚠️ **已废弃**: 请使用 `new_param` 替代
      
      此参数将在 v2.0 中移除。
```

## 相关文档

- **[组件管理](../components/README.md)**: 可复用组件的使用
- **[代码示例](../code_samples/README.md)**: 多语言示例管理
- **[OpenAPI 根目录](../README.md)**: 整体架构说明

## 参考资源

- [OpenAPI 3.1.0 Paths 规范](https://spec.openapis.org/oas/v3.1.0#paths-object)
- [OpenAPI Operation 对象](https://spec.openapis.org/oas/v3.1.0#operation-object)
- [HTTP 状态码参考](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

---

> 💡 **提示**: 良好的路径定义是 API 文档质量的关键。投入时间设计清晰、一致的端点定义将极大提升开发者体验。 