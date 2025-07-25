# 请求头定义 (Headers)

本目录包含可复用的 HTTP 请求头定义，用于在多个 API 端点间共享通用的头部信息。

## 目录结构

```
headers/
├── ExpiresAfter.yaml   # 过期时间头定义
└── README.md           # 本文件
```

## 头部类型

### 过期时间头 (ExpiresAfter.yaml)
定义令牌或资源的过期时间信息。

```yaml
# ExpiresAfter.yaml
description: 令牌过期时间（秒）
schema:
  type: integer
  minimum: 1
  maximum: 86400
  example: 7200
```

## 设计原则

### 1. 标准化
- 遵循 HTTP 头部命名规范
- 使用标准的头部格式
- 提供清晰的描述和约束

### 2. 可复用性
- 设计通用的头部定义
- 支持不同场景的使用
- 便于在多个端点间复用

### 3. 完整性
- 包含必要的验证规则
- 提供示例值
- 添加详细的使用说明

## 常见头部定义

### 认证相关头部
```yaml
# Authorization.yaml
description: Bearer Token 认证头
schema:
  type: string
  pattern: '^Bearer [A-Za-z0-9\-\._~\+\/]+=*$'
  example: "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### 内容类型头部
```yaml
# ContentType.yaml
description: 请求内容类型
schema:
  type: string
  enum:
    - "application/json"
    - "application/xml"
    - "text/plain"
  example: "application/json"
```

### 请求标识头部
```yaml
# RequestId.yaml
description: 请求唯一标识符
schema:
  type: string
  format: uuid
  example: "123e4567-e89b-12d3-a456-426614174000"
```

### 版本控制头部
```yaml
# ApiVersion.yaml
description: API 版本号
schema:
  type: string
  pattern: '^v[0-9]+(\.[0-9]+)*$'
  example: "v1.0"
```

### 速率限制头部
```yaml
# RateLimitRemaining.yaml
description: 剩余请求次数
schema:
  type: integer
  minimum: 0
  example: 99
```

```yaml
# RateLimitReset.yaml
description: 速率限制重置时间（Unix 时间戳）
schema:
  type: integer
  format: int64
  example: 1640995200
```

### 分页头部
```yaml
# PaginationTotal.yaml
description: 总记录数
schema:
  type: integer
  minimum: 0
  example: 1000
```

```yaml
# PaginationLimit.yaml
description: 每页限制数量
schema:
  type: integer
  minimum: 1
  maximum: 100
  example: 20
```

## 使用方式

### 在响应中定义头部
```yaml
# 在 paths 文件中使用
responses:
  '200':
    description: 登录成功
    headers:
      Expires-After:
        $ref: ../components/headers/ExpiresAfter.yaml
      X-RateLimit-Remaining:
        $ref: ../components/headers/RateLimitRemaining.yaml
      X-RateLimit-Reset:
        $ref: ../components/headers/RateLimitReset.yaml
    content:
      application/json:
        schema:
          $ref: ../components/schemas/AuthResponse.yaml
```

### 在参数中使用头部
```yaml
# 作为请求参数
parameters:
  - name: Authorization
    in: header
    description: Bearer Token 认证
    required: true
    schema:
      $ref: ../components/headers/Authorization.yaml
  - name: X-Request-ID
    in: header
    description: 请求追踪标识
    required: false
    schema:
      $ref: ../components/headers/RequestId.yaml
```

## 头部分类

### 标准 HTTP 头部
基于 HTTP 规范的标准头部：
- `Content-Type`: 内容类型
- `Authorization`: 认证信息
- `Accept`: 接受的内容类型
- `User-Agent`: 用户代理信息

### 自定义头部
项目特定的自定义头部（建议使用 `X-` 前缀）：
- `X-Request-ID`: 请求标识
- `X-API-Version`: API 版本
- `X-RateLimit-*`: 速率限制信息
- `X-Trace-ID`: 链路追踪标识

### 响应头部
服务器返回的头部信息：
- `Expires-After`: 过期时间
- `X-RateLimit-Remaining`: 剩余请求数
- `X-Total-Count`: 总数量
- `Cache-Control`: 缓存控制

## 最佳实践

### 1. 命名规范
```yaml
# 使用标准的 HTTP 头部名称
Content-Type: "application/json"
Authorization: "Bearer token"

# 自定义头部使用 X- 前缀
X-Request-ID: "uuid"
X-API-Version: "v1"
X-Custom-Header: "value"
```

### 2. 验证规则
```yaml
# 添加适当的验证规则
schema:
  type: string
  pattern: '^Bearer [A-Za-z0-9\-\._~\+\/]+=*$'  # 格式验证
  minLength: 10                                   # 最小长度
  maxLength: 500                                  # 最大长度
  example: "Bearer eyJhbG..."                     # 示例值
```

### 3. 文档说明
```yaml
description: |
  Bearer Token 认证头
  
  **格式**: `Bearer <token>`
  
  **示例**: `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`
  
  **注意**: 令牌有效期为2小时，过期后需要使用刷新令牌获取新的访问令牌
```

## 安全考虑

### 敏感头部处理
```yaml
# 认证头部不应在示例中暴露真实值
Authorization:
  description: Bearer Token 认证头
  schema:
    type: string
    pattern: '^Bearer [A-Za-z0-9\-\._~\+\/]+=*$'
    example: "Bearer YOUR_ACCESS_TOKEN"  # 使用占位符
```

### 日志记录
对于包含敏感信息的头部，应在日志中进行脱敏处理：
- `Authorization`: 只记录前缀，隐藏令牌
- `X-API-Key`: 完全隐藏或只显示部分字符

## 维护指南

### 添加新头部
1. 创建新的 YAML 文件
2. 遵循命名规范
3. 添加完整的验证规则和描述
4. 在相关路径中引用

### 修改现有头部
1. 考虑向后兼容性
2. 更新相关文档
3. 通知 API 使用者

### 删除废弃头部
1. 先标记为 `deprecated`
2. 提供迁移指南
3. 设置合理的废弃期限

## 验证和测试

```bash
# 验证头部定义
npx redocly lint openapi/components/headers/ExpiresAfter.yaml

# 验证完整规范
npx redocly lint openapi/openapi.yaml
```

## 相关文档

- **[路径定义指南](../../paths/README.md)**: 如何在路径中使用头部
- **[组件管理指南](../README.md)**: 完整的组件管理说明
- **[OpenAPI 规范指南](../../README.md)**: 项目整体架构

## 参考资源

- [HTTP 头部字段列表](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [OpenAPI Header Object](https://spec.openapis.org/oas/v3.1.0#header-object)
- [HTTP 认证规范](https://tools.ietf.org/html/rfc7235)

---

> 💡 **提示**: 合理使用 HTTP 头部可以提供丰富的元数据信息，改善 API 的可用性和监控能力。 