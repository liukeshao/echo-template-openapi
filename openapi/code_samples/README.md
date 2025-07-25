# 代码示例 (Code Samples)

本目录管理 API 文档中的多语言代码示例，通过 `x-codeSamples` 扩展为每个 API 端点提供丰富的示例代码，帮助开发者快速理解和使用 API。

## 目录结构

```
code_samples/
├── curl/                  # cURL 示例
│   ├── auth/
│   │   ├── login.sh      # 登录示例
│   │   ├── register.sh   # 注册示例
│   │   └── refresh.sh    # 刷新令牌示例
│   └── users/
│       ├── get-profile.sh # 获取用户信息示例
│       └── update-profile.sh # 更新用户信息示例
├── javascript/            # JavaScript 示例
│   ├── auth/
│   │   ├── login.js
│   │   ├── register.js
│   │   └── refresh.js
│   └── users/
│       ├── get-profile.js
│       └── update-profile.js
├── python/               # Python 示例
│   ├── auth/
│   │   ├── login.py
│   │   ├── register.py
│   │   └── refresh.py
│   └── users/
│       ├── get-profile.py
│       └── update-profile.py
└── README.md             # 本文件
```

## 组织原则

### 目录命名约定

遵循以下命名规则：

1. **语言目录**: 使用小写的语言名称
   - `curl/`, `javascript/`, `python/`, `java/`, `go/` 等

2. **功能分组**: 按 API 标签或功能模块分组
   - `auth/` - 认证相关
   - `users/` - 用户管理
   - `orders/` - 订单管理

3. **文件命名**: `<HTTP方法>-<端点描述>.<扩展名>`
   - `login.js` - 登录
   - `get-profile.py` - 获取用户信息
   - `create-order.go` - 创建订单

### 路径映射规则

API 路径到文件路径的映射：

```
API 端点: POST /api/v1/auth/login
文件路径: <语言>/auth/login.<扩展名>

API 端点: GET /api/v1/me
文件路径: <语言>/users/get-profile.<扩展名>

API 端点: PUT /api/v1/me/username  
文件路径: <语言>/users/update-username.<扩展名>
```

## 支持的语言

### 当前支持的语言

- **cURL**: 命令行工具，适合快速测试
- **JavaScript**: 浏览器和 Node.js 环境
- **Python**: 使用 requests 库
- **Java**: 使用 HttpClient 或 OkHttp
- **Go**: 使用标准库 net/http

### 添加新语言支持

1. 创建语言目录：
   ```bash
   mkdir openapi/code_samples/php
   ```

2. 添加到 Redocly 配置：
   ```yaml
   # redocly.yaml
   generateCodeSamples:
     languages:
       - lang: curl
       - lang: javascript
       - lang: python
       - lang: php  # 新增
   ```

## 代码示例规范

### 通用要求

1. **完整性**: 包含完整的请求代码，可直接运行
2. **清晰性**: 代码结构清晰，添加必要注释
3. **一致性**: 同一语言的所有示例保持一致的风格
4. **实用性**: 使用真实的参数值和响应处理

### 示例模板

#### cURL 示例

```bash
#!/bin/bash
# 用户登录示例

curl -X POST "http://localhost:8000/api/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "password": "password123",
    "remember": true
  }' \
  | jq '.'

# 响应示例:
# {
#   "code": 0,
#   "message": "登录成功",
#   "data": {
#     "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
#     "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
#     "expires_in": 7200,
#     "user_info": {
#       "id": "user_123456",
#       "username": "john_doe",
#       "email": "john@example.com"
#     }
#   },
#   "timestamp": 1640995200000
# }
```

#### JavaScript 示例

```javascript
// 用户登录示例
async function login(username, password, remember = false) {
  try {
    const response = await fetch('http://localhost:8000/api/v1/auth/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        username,
        password,
        remember
      })
    });

    const result = await response.json();
    
    if (result.code === 0) {
      // 登录成功，保存令牌
      localStorage.setItem('access_token', result.data.access_token);
      localStorage.setItem('refresh_token', result.data.refresh_token);
      
      console.log('登录成功:', result.data.user_info);
      return result.data;
    } else {
      console.error('登录失败:', result.message);
      throw new Error(result.message);
    }
  } catch (error) {
    console.error('请求失败:', error);
    throw error;
  }
}

// 使用示例
login('john_doe', 'password123', true)
  .then(data => {
    console.log('用户信息:', data.user_info);
  })
  .catch(error => {
    console.error('登录失败:', error.message);
  });
```

#### Python 示例

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
用户登录示例
"""

import requests
import json

def login(username, password, remember=False):
    """
    用户登录
    
    Args:
        username (str): 用户名或邮箱
        password (str): 密码
        remember (bool): 是否记住登录状态
        
    Returns:
        dict: 登录结果
        
    Raises:
        Exception: 登录失败时抛出异常
    """
    url = "http://localhost:8000/api/v1/auth/login"
    
    payload = {
        "username": username,
        "password": password,
        "remember": remember
    }
    
    headers = {
        "Content-Type": "application/json"
    }
    
    try:
        response = requests.post(url, json=payload, headers=headers)
        response.raise_for_status()  # 检查 HTTP 错误
        
        result = response.json()
        
        if result.get("code") == 0:
            # 登录成功
            print(f"登录成功: {result['data']['user_info']['username']}")
            return result["data"]
        else:
            # 业务逻辑错误
            raise Exception(f"登录失败: {result.get('message', '未知错误')}")
            
    except requests.exceptions.RequestException as e:
        raise Exception(f"网络请求失败: {str(e)}")
    except json.JSONDecodeError as e:
        raise Exception(f"响应解析失败: {str(e)}")

# 使用示例
if __name__ == "__main__":
    try:
        login_data = login("john_doe", "password123", True)
        print(f"访问令牌: {login_data['access_token'][:20]}...")
        print(f"用户信息: {login_data['user_info']}")
    except Exception as e:
        print(f"错误: {e}")
```

## 在 OpenAPI 中引用

### 使用 x-codeSamples 扩展

在 API 端点定义中添加代码示例引用：

```yaml
# paths/api_v1_auth_login.yaml
post:
  tags:
    - 认证管理
  summary: 用户登录
  operationId: LoginUser
  requestBody:
    $ref: ../components/requestBodies/LoginInput.yaml
  responses:
    '200':
      $ref: ../components/schemas/AuthApiResponse.yaml
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

### 内联代码示例

对于简单的示例，也可以直接内联：

```yaml
x-codeSamples:
  - lang: curl
    label: 基本用法
    source: |
      curl -X GET "http://localhost:8000/api/v1/me" \
        -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
  - lang: javascript
    label: Fetch API
    source: |
      fetch('http://localhost:8000/api/v1/me', {
        headers: {
          'Authorization': 'Bearer ' + localStorage.getItem('access_token')
        }
      })
      .then(response => response.json())
      .then(data => console.log(data));
```

## 自动生成配置

### Redocly CLI 配置

在 `redocly.yaml` 中配置自动生成：

```yaml
theme:
  openapi:
    generateCodeSamples:
      languages:
        - lang: curl
          label: cURL
        - lang: javascript
          label: JavaScript (fetch)
        - lang: python
          label: Python (requests)
        - lang: java
          label: Java (HttpClient)
        - lang: go
          label: Go (net/http)
```

### 生成规则

自动生成的代码示例将：

1. 根据请求体 schema 生成示例数据
2. 包含正确的认证头
3. 处理响应数据
4. 添加错误处理逻辑

## 最佳实践

### 1. 代码质量

- **语法正确**: 确保代码可以直接运行
- **错误处理**: 包含适当的错误处理逻辑
- **注释说明**: 添加必要的注释和说明

### 2. 安全考虑

- **敏感信息**: 不要在示例中包含真实的密钥或密码
- **占位符**: 使用明显的占位符，如 `YOUR_API_KEY`
- **HTTPS**: 生产环境示例使用 HTTPS

### 3. 维护性

- **版本同步**: 保持代码示例与 API 版本同步
- **定期测试**: 定期测试示例代码的可用性
- **文档更新**: API 变更时及时更新相关示例

### 4. 用户体验

- **循序渐进**: 从简单到复杂的示例顺序
- **实际场景**: 提供真实使用场景的示例
- **多种方式**: 为同一功能提供多种实现方式

## 测试和验证

### 手动测试

定期运行代码示例确保其正确性：

```bash
# 测试 cURL 示例
chmod +x openapi/code_samples/curl/auth/login.sh
./openapi/code_samples/curl/auth/login.sh

# 测试 Python 示例
python3 openapi/code_samples/python/auth/login.py

# 测试 JavaScript 示例
node openapi/code_samples/javascript/auth/login.js
```

### 自动化测试

可以编写测试脚本来验证代码示例：

```bash
#!/bin/bash
# test_samples.sh - 代码示例测试脚本

echo "测试代码示例..."

# 测试 cURL 示例
for file in openapi/code_samples/curl/**/*.sh; do
  echo "测试: $file"
  chmod +x "$file"
  if ! bash "$file" > /dev/null 2>&1; then
    echo "❌ 失败: $file"
  else
    echo "✅ 通过: $file"
  fi
done

echo "测试完成"
```

## 工具和资源

### 推荐工具

- **Postman**: 生成各种语言的代码示例
- **Insomnia**: API 测试和代码生成
- **HTTPie**: 人性化的 HTTP 命令行工具
- **curl-to-code**: 将 cURL 命令转换为其他语言

### 参考资源

- [Redocly x-codeSamples 文档](https://redocly.com/docs/api-reference-docs/specification-extensions/x-code-samples/)
- [OpenAPI Code Generation](https://openapi-generator.tech/)
- [语言特定的 HTTP 客户端库](https://github.com/topics/http-client)

---

> 💡 **提示**: 高质量的代码示例是优秀 API 文档的重要组成部分，投入时间完善代码示例将大大提升开发者体验。
