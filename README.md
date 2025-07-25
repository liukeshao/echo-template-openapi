# OpenAPI Template 

## 项目简介

这是一个 OpenAPI 模板项目，采用最新的 Redocly CLI 2.0.0 工具链，提供了完整的 API 文档管理解决方案。项目遵循 OpenAPI 3.1.0 规范，支持现代化的 API 开发工作流。

## 升级亮点

- ✅ **Redocly CLI 2.0.0**: 采用最新版本，性能和稳定性显著提升
- ✅ **配置兼容性**: 配置文件已调整为符合2.0版本规范
- ✅ **所有功能正常**: lint、preview-docs、build-docs、bundle 等命令都已验证可用
- ✅ **使用 Context7**: 利用最新的文档和最佳实践指导升级
- ✅ **统一响应格式**: 采用统一的 API 响应结构设计
- ✅ **模块化组织**: 清晰的目录结构和组件化设计
- ✅ **Lint 质量控制**: 通过所有质量检查，无未使用组件警告
- ✅ **分层响应架构**: 采用具体响应类型提供更好的类型安全

## 快速开始

### 环境要求

- [Node.js](https://nodejs.org/) v16.0.0 或更高版本
- npm 7.0.0 或更高版本

### 安装

```bash
# 克隆项目
git clone <repository-url>
cd echo-template-openapi

# 安装依赖
npm install
```

### 使用方法

#### `npm start`
启动文档预览服务器，支持热重载
```bash
npm start
# 访问 http://localhost:8080 查看文档
```

#### `npm run build`
构建静态文档到 dist 目录
```bash
npm run build
# 输出文件: dist/index.html
```

#### `npm test`
验证 OpenAPI 定义文件的正确性
```bash
npm test
# 运行 lint 检查和规范验证
```

#### `npm run preview`
预览构建后的文档
```bash
npm run preview
```

## 项目结构

```
echo-template-openapi/
├── docs/                    # 文档相关资源
│   ├── index.html          # HTML 模板文件
│   └── favicon.png         # 网站图标
├── openapi-core/           # OpenAPI 规范文件
│   ├── openapi.yaml        # 主规范文件（入口点）
│   ├── components/         # 可复用组件
│   │   ├── schemas/        # 数据模型定义
│   │   ├── responses/      # 响应定义
│   │   └── headers/        # 请求头定义
│   ├── paths/              # API 路径定义
│   └── code_samples/       # 代码示例
├── dist/                   # 构建输出目录
│   └── core@v1.yaml        # 打包后的OpenAPI文件
├── redocly.yaml            # Redocly CLI 配置
└── package.json            # 项目依赖和脚本
```

## 贡献指南

### 开发工作流

1. **分支管理**: 从 `main` 分支创建功能分支
2. **修改文档**: 根据下面的指南修改相应文件
3. **验证**: 运行 `npm test` 确保无错误
4. **预览**: 运行 `npm start` 预览文档效果
5. **提交**: 创建 Pull Request

### 文档结构说明

本项目采用模块化的文档组织方式，详细说明请参考各目录的 README：

- **[OpenAPI 根目录](openapi-core/README.md)**: 主规范文件和整体结构
- **[组件管理](openapi-core/components/README.md)**: 可复用组件的组织和使用
- **[路径定义](openapi-core/paths/README.md)**: API 端点的定义规范
- **[代码示例](openapi-core/code_samples/README.md)**: 多语言代码示例管理

### 添加新的 API 端点

1. 在 `openapi-core/paths/` 目录下创建新的 YAML 文件
2. 在 `openapi-core/openapi.yaml` 中添加路径引用
3. 定义相关的 schemas 和 responses
4. 添加代码示例（可选）
5. 运行 `npm test` 验证无 lint 错误
6. 运行 `npm run build` 重新构建文档

### Schema 设计原则

- 使用 `$ref` 引用复用组件
- 提供清晰的描述和示例
- 遵循命名约定
- 添加适当的验证规则

### 配置说明

`redocly.yaml` 配置文件控制各种工具的行为，包括：

- **Lint 规则**: 代码质量检查
- **文档引擎**: 渲染选项和主题
- **构建设置**: 输出格式和优化

更多配置选项请参考 [Redocly CLI 文档](https://redocly.com/docs/cli/configuration/)。

## API 设计规范

### 响应格式

所有 API 响应都使用统一格式，根据不同业务场景采用对应的响应类型：

#### 基础响应结构
```json
{
  "code": 0,           // 0=成功，非0=错误
  "message": "成功",    // 响应消息
  "data": {},          // 响应数据（可选）
  "timestamp": 1234567890,
  "request_id": "req_123"  // 请求追踪ID
}
```

#### 响应类型分类

1. **AuthApiResponse** - 认证相关接口响应
   - 用于：注册、登录、刷新令牌
   - data 包含：用户信息、访问令牌、刷新令牌

2. **UserApiResponse** - 用户信息相关接口响应  
   - 用于：获取用户信息、更新用户名、更新邮箱
   - data 包含：用户详细信息

3. **SimpleApiResponse** - 简单操作接口响应
   - 用于：登出、修改密码等
   - data 为 null（仅返回操作结果）

### 认证机制

使用 Bearer Token 认证：
```
Authorization: Bearer <access_token>
```

### 错误处理

- HTTP 状态码始终为 200
- 错误通过 `code` 字段标识
- 提供清晰的错误消息

## 最佳实践

### 文档编写

1. **描述清晰**: 为每个端点、参数和响应提供清晰描述
2. **示例丰富**: 包含请求和响应示例
3. **版本管理**: 合理使用版本控制
4. **标签组织**: 使用标签和标签组组织 API

#### 组件使用

1. **避免未使用组件**: 定义的所有组件都应该被引用，避免 lint 错误
2. **具体响应类型**: 优先使用具体的响应类型（如 AuthApiResponse）而非通用类型
3. **组件复用**: 充分利用 `$ref` 引用复用组件
4. **定期清理**: 定期检查和清理未使用的组件

### 性能优化

1. **模块化**: 将大型规范分解为小文件
2. **引用复用**: 使用 `$ref` 避免重复
3. **构建优化**: 利用 Redocly 的构建优化功能
4. **Lint 检查**: 运行 `npm test` 确保规范质量

### 团队协作

1. **代码审查**: 对文档变更进行审查
2. **自动化**: 使用 CI/CD 自动验证和部署
3. **文档同步**: 保持文档与代码同步
4. **质量控制**: 确保所有变更都通过 lint 检查

## 部署

### 静态部署

```bash
# 构建静态文件
npm run build

# 部署到静态托管服务
# 例如: Netlify, Vercel, GitHub Pages
```

### Docker 部署

```bash
# 使用 Redocly 官方镜像
docker run -p 8080:80 \
  -v $(pwd)/openapi:/usr/share/nginx/html/spec \
  -e SPEC_URL=spec/openapi.yaml \
  redocly/redoc
```

## 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件。

## 支持

- 📧 邮箱: support@example.com
- 📖 文档: [Redocly CLI 文档](https://redocly.com/docs/cli/)
- 🐛 问题反馈: [GitHub Issues](../../issues)

---

> 💡 **提示**: 如需了解更多关于 OpenAPI 和 Redocly 的最佳实践，请查看各子目录的详细 README 文档。
