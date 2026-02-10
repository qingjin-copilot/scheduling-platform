# API 接口文档

## 1. 接口概述

### 1.1 基础信息
- **Base URL**: `http://localhost:8080/api/v1`
- **协议**: HTTP/HTTPS
- **数据格式**: JSON
- **字符编码**: UTF-8

### 1.2 认证方式
- 使用 JWT Token 进行身份认证
- Token 通过 HTTP Header 传递：`Authorization: Bearer <token>`
- Token 有效期：24小时

### 1.3 统一响应格式

**成功响应：**
```json
{
  "code": 200,
  "message": "success",
  "data": { ... }
}
```

**失败响应：**
```json
{
  "code": 400,
  "message": "错误信息",
  "errors": [
    {
      "field": "字段名",
      "message": "具体错误"
    }
  ]
}
```

### 1.4 HTTP 状态码

| 状态码 | 说明 |
|--------|------|
| 200 | 成功 |
| 201 | 创建成功 |
| 400 | 请求参数错误 |
| 401 | 未认证 |
| 403 | 无权限 |
| 404 | 资源不存在 |
| 500 | 服务器错误 |

## 2. 用户认证接口

### 2.1 用户注册
**接口**: `POST /auth/register`

**请求参数**:
```json
{
  "username": "string (3-50字符，必填)",
  "email": "string (邮箱格式，必填)",
  "password": "string (8-20字符，必填)"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "注册成功",
  "data": {
    "id": 1,
    "username": "testuser",
    "email": "test@example.com",
    "token": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

### 2.2 用户登录
**接口**: `POST /auth/login`

**请求参数**:
```json
{
  "username": "string (必填)",
  "password": "string (必填)"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "登录成功",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": 1,
      "username": "testuser",
      "email": "test@example.com",
      "role": "USER"
    }
  }
}
```

### 2.3 获取当前用户信息
**接口**: `GET /auth/me`

**请求头**: `Authorization: Bearer <token>`

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "username": "testuser",
    "email": "test@example.com",
    "nickname": "测试用户",
    "avatar": "https://example.com/avatar.jpg",
    "role": "USER"
  }
}
```

## 3. 项目管理接口

### 3.1 创建项目
**接口**: `POST /projects`

**请求参数**:
```json
{
  "name": "string (3-100字符，必填)",
  "type": "FRONTEND | BACKEND (必填)",
  "description": "string (可选)",
  "templateRepo": "string (可选，不填使用默认模板)"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "项目创建成功",
  "data": {
    "id": 1,
    "name": "我的项目",
    "type": "FRONTEND",
    "status": "INITIALIZING",
    "createdAt": "2024-01-20T10:30:00Z"
  }
}
```

### 3.2 获取项目列表
**接口**: `GET /projects`

**查询参数**:
- `page`: 页码，默认1
- `size`: 每页数量，默认20
- `type`: 项目类型筛选（FRONTEND, BACKEND）
- `status`: 状态筛选（ACTIVE, ARCHIVED等）
- `keyword`: 搜索关键词（搜索名称和描述）

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "total": 10,
    "page": 1,
    "size": 20,
    "items": [
      {
        "id": 1,
        "name": "我的项目",
        "type": "FRONTEND",
        "status": "ACTIVE",
        "taskCount": {
          "uiTasks": 3,
          "codingTasks": 5
        },
        "createdAt": "2024-01-20T10:30:00Z",
        "updatedAt": "2024-01-20T14:25:00Z"
      }
    ]
  }
}
```

### 3.3 获取项目详情
**接口**: `GET /projects/{id}`

**路径参数**:
- `id`: 项目ID

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "name": "我的项目",
    "type": "FRONTEND",
    "description": "项目描述",
    "templateRepo": "https://github.com/example/vue3-template.git",
    "status": "ACTIVE",
    "statistics": {
      "uiTasks": {
        "total": 3,
        "completed": 2,
        "inProgress": 1
      },
      "codingTasks": {
        "total": 5,
        "completed": 4,
        "inProgress": 1
      },
      "deployments": 2
    },
    "recentActivities": [
      {
        "action": "CODING_TASK_COMPLETED",
        "description": "编码任务\"登录页面\"已完成",
        "createdAt": "2024-01-20T14:25:00Z"
      }
    ],
    "createdAt": "2024-01-20T10:30:00Z",
    "updatedAt": "2024-01-20T14:25:00Z"
  }
}
```

### 3.4 更新项目
**接口**: `PUT /projects/{id}`

**请求参数**:
```json
{
  "name": "string (可选)",
  "description": "string (可选)",
  "status": "ACTIVE | ARCHIVED (可选)"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "项目更新成功",
  "data": {
    "id": 1,
    "name": "更新后的项目名",
    "description": "更新后的描述"
  }
}
```

### 3.5 删除项目
**接口**: `DELETE /projects/{id}`

**响应示例**:
```json
{
  "code": 200,
  "message": "项目删除成功"
}
```

## 4. UI 任务管理接口

### 4.1 创建 UI 任务
**接口**: `POST /projects/{projectId}/ui-tasks`

**请求参数**:
```json
{
  "name": "string (必填)",
  "description": "string (可选)",
  "platform": "FIGMA | SKETCH (必填)",
  "componentUrls": ["string (必填，至少1个)"]
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "UI任务创建成功",
  "data": {
    "id": 1,
    "projectId": 1,
    "name": "首页组件",
    "status": "FETCHING",
    "totalComponents": 5,
    "fetchedComponents": 0
  }
}
```

### 4.2 获取 UI 任务列表
**接口**: `GET /projects/{projectId}/ui-tasks`

**查询参数**:
- `page`: 页码
- `size`: 每页数量
- `status`: 状态筛选

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "total": 5,
    "items": [
      {
        "id": 1,
        "name": "首页组件",
        "status": "COMPLETED",
        "totalComponents": 5,
        "fetchedComponents": 5,
        "progress": 100,
        "createdAt": "2024-01-20T10:00:00Z"
      }
    ]
  }
}
```

### 4.3 获取 UI 任务详情
**接口**: `GET /ui-tasks/{taskId}`

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "projectId": 1,
    "name": "首页组件",
    "description": "首页相关UI组件",
    "platform": "FIGMA",
    "status": "COMPLETED",
    "totalComponents": 5,
    "fetchedComponents": 5,
    "components": [
      {
        "id": 1,
        "name": "Header组件",
        "url": "https://figma.com/file/xxx",
        "status": "FETCHED",
        "screenshotUrl": "https://storage.example.com/screenshot1.png",
        "hasSourceCode": true
      }
    ]
  }
}
```

### 4.4 获取 UI 组件详情
**接口**: `GET /ui-components/{componentId}`

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "taskId": 1,
    "name": "Header组件",
    "url": "https://figma.com/file/xxx",
    "sourceCode": "<template>...</template>",
    "screenshotUrl": "https://storage.example.com/screenshot1.png",
    "properties": {
      "width": 1920,
      "height": 80,
      "backgroundColor": "#ffffff"
    },
    "status": "FETCHED"
  }
}
```

## 5. 编码任务管理接口

### 5.1 创建编码任务
**接口**: `POST /projects/{projectId}/coding-tasks`

**请求参数**:
```json
{
  "name": "string (必填)",
  "description": "string (可选)",
  "uiComponentIds": [1, 2, 3],
  "aiModel": "GPT-4 (可选，默认GPT-4)",
  "customInstructions": "string (可选)"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "编码任务创建成功",
  "data": {
    "id": 1,
    "projectId": 1,
    "name": "登录页面开发",
    "status": "PENDING",
    "progress": 0
  }
}
```

### 5.2 启动编码任务
**接口**: `POST /coding-tasks/{taskId}/start`

**响应示例**:
```json
{
  "code": 200,
  "message": "任务已启动",
  "data": {
    "id": 1,
    "status": "RUNNING",
    "startedAt": "2024-01-20T10:30:00Z"
  }
}
```

### 5.3 获取编码任务列表
**接口**: `GET /projects/{projectId}/coding-tasks`

**查询参数**:
- `page`: 页码
- `size`: 每页数量
- `status`: 状态筛选

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "total": 10,
    "items": [
      {
        "id": 1,
        "name": "登录页面开发",
        "status": "RUNNING",
        "progress": 50,
        "startedAt": "2024-01-20T10:30:00Z",
        "estimatedCompletion": "2024-01-20T11:00:00Z"
      }
    ]
  }
}
```

### 5.4 获取编码任务详情
**接口**: `GET /coding-tasks/{taskId}`

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "projectId": 1,
    "name": "登录页面开发",
    "description": "开发登录页面",
    "uiComponents": [
      {
        "id": 1,
        "name": "LoginForm组件"
      }
    ],
    "aiModel": "GPT-4",
    "status": "RUNNING",
    "progress": 50,
    "startedAt": "2024-01-20T10:30:00Z",
    "logs": "[10:30:15] 任务开始执行...\n[10:30:20] 分析 UI 组件结构..."
  }
}
```

### 5.5 获取编码任务日志
**接口**: `GET /coding-tasks/{taskId}/logs`

**查询参数**:
- `tail`: 获取最后N行日志，默认100

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "taskId": 1,
    "logs": "[10:30:15] 任务开始执行...\n[10:30:20] 分析 UI 组件结构...\n[10:30:45] 生成页面框架代码..."
  }
}
```

### 5.6 取消编码任务
**接口**: `POST /coding-tasks/{taskId}/cancel`

**响应示例**:
```json
{
  "code": 200,
  "message": "任务已取消",
  "data": {
    "id": 1,
    "status": "CANCELLED"
  }
}
```

## 6. 部署管理接口

### 6.1 创建部署
**接口**: `POST /projects/{projectId}/deployments`

**请求参数**:
```json
{
  "version": "string (必填)",
  "environment": "DEV | STAGING | PRODUCTION (必填)",
  "branch": "string (可选，默认main)",
  "config": {
    "port": 8080,
    "memory": "512M",
    "cpu": "1",
    "replicas": 1
  },
  "envVariables": {
    "NODE_ENV": "production"
  },
  "runTests": true,
  "sendNotification": true
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "部署任务创建成功",
  "data": {
    "id": 1,
    "projectId": 1,
    "version": "v1.0.0",
    "environment": "PRODUCTION",
    "status": "PENDING"
  }
}
```

### 6.2 获取部署列表
**接口**: `GET /projects/{projectId}/deployments`

**查询参数**:
- `page`: 页码
- `size`: 每页数量
- `environment`: 环境筛选
- `status`: 状态筛选

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "total": 5,
    "items": [
      {
        "id": 1,
        "version": "v1.0.0",
        "environment": "PRODUCTION",
        "status": "SUCCESS",
        "deployUrl": "https://app.example.com",
        "startedAt": "2024-01-20T15:00:00Z",
        "completedAt": "2024-01-20T15:03:25Z",
        "durationSeconds": 205
      }
    ]
  }
}
```

### 6.3 获取部署详情
**接口**: `GET /deployments/{deploymentId}`

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "projectId": 1,
    "version": "v1.0.0",
    "environment": "PRODUCTION",
    "branch": "main",
    "status": "SUCCESS",
    "deployUrl": "https://app.example.com",
    "dockerImage": "app:v1.0.0",
    "containerId": "abc123",
    "logs": "[15:00:00] 开始部署...\n[15:00:05] ✓ 代码检出完成...",
    "startedAt": "2024-01-20T15:00:00Z",
    "completedAt": "2024-01-20T15:03:25Z",
    "durationSeconds": 205
  }
}
```

### 6.4 回滚部署
**接口**: `POST /deployments/{deploymentId}/rollback`

**响应示例**:
```json
{
  "code": 200,
  "message": "回滚成功",
  "data": {
    "id": 2,
    "version": "v1.0.0",
    "status": "SUCCESS"
  }
}
```

## 7. WebSocket 接口

### 7.1 编码任务进度推送
**连接**: `ws://localhost:8080/ws/coding-tasks/{taskId}`

**认证**: 连接时需要在URL中携带token参数  
`ws://localhost:8080/ws/coding-tasks/{taskId}?token=xxx`

**消息格式**:
```json
{
  "type": "PROGRESS_UPDATE | LOG_MESSAGE | STATUS_CHANGE | TASK_COMPLETED | TASK_FAILED",
  "taskId": 1,
  "progress": 50,
  "message": "正在生成页面框架代码...",
  "timestamp": "2024-01-20T10:32:00Z"
}
```

### 7.2 部署进度推送
**连接**: `ws://localhost:8080/ws/deployments/{deploymentId}`

**消息格式**:
```json
{
  "type": "DEPLOYMENT_UPDATE | STATUS_CHANGE | DEPLOYMENT_COMPLETED | DEPLOYMENT_FAILED",
  "deploymentId": 1,
  "status": "BUILDING",
  "message": "正在构建Docker镜像...",
  "timestamp": "2024-01-20T15:01:00Z"
}
```

## 8. 系统配置接口

### 8.1 获取系统配置
**接口**: `GET /config/{key}`

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "key": "default_frontend_template",
    "value": "https://github.com/example/vue3-template.git",
    "description": "默认Vue3前端模板仓库"
  }
}
```

### 8.2 更新系统配置（仅管理员）
**接口**: `PUT /config/{key}`

**请求参数**:
```json
{
  "value": "string (必填)"
}
```

## 9. API 密钥管理接口

### 9.1 添加 API 密钥
**接口**: `POST /api-keys`

**请求参数**:
```json
{
  "platform": "FIGMA | AI_AGENT (必填)",
  "apiKey": "string (必填)"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "API密钥添加成功",
  "data": {
    "id": 1,
    "platform": "FIGMA",
    "isActive": true
  }
}
```

### 9.2 获取 API 密钥列表
**接口**: `GET /api-keys`

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "id": 1,
      "platform": "FIGMA",
      "isActive": true,
      "createdAt": "2024-01-20T10:00:00Z"
    }
  ]
}
```

### 9.3 删除 API 密钥
**接口**: `DELETE /api-keys/{id}`

## 10. 项目预览接口

### 10.1 启动预览
**接口**: `POST /projects/{projectId}/preview/start`

**响应示例**:
```json
{
  "code": 200,
  "message": "预览服务启动成功",
  "data": {
    "previewUrl": "http://localhost:3000",
    "processId": "12345"
  }
}
```

### 10.2 停止预览
**接口**: `POST /projects/{projectId}/preview/stop`

**响应示例**:
```json
{
  "code": 200,
  "message": "预览服务已停止"
}
```

### 10.3 获取预览状态
**接口**: `GET /projects/{projectId}/preview/status`

**响应示例**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "isRunning": true,
    "previewUrl": "http://localhost:3000",
    "startedAt": "2024-01-20T14:25:00Z"
  }
}
```

## 11. 错误码说明

| 错误码 | 说明 |
|--------|------|
| 1001 | 用户名已存在 |
| 1002 | 邮箱已存在 |
| 1003 | 用户名或密码错误 |
| 1004 | Token无效或已过期 |
| 2001 | 项目不存在 |
| 2002 | 项目名称已存在 |
| 2003 | 无权限访问该项目 |
| 3001 | UI任务不存在 |
| 3002 | UI组件获取失败 |
| 4001 | 编码任务不存在 |
| 4002 | 任务执行失败 |
| 5001 | 部署失败 |
| 5002 | 回滚失败 |

## 12. 接口调用示例

### 12.1 使用 cURL

```bash
# 用户登录
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"testuser","password":"password123"}'

# 创建项目（需要token）
curl -X POST http://localhost:8080/api/v1/projects \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  -d '{"name":"我的项目","type":"FRONTEND"}'

# 获取项目列表
curl -X GET http://localhost:8080/api/v1/projects \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

### 12.2 使用 JavaScript (Axios)

```javascript
// 配置axios
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:8080/api/v1',
  headers: {
    'Content-Type': 'application/json'
  }
});

// 添加请求拦截器（自动添加token）
api.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// 用户登录
const login = async (username, password) => {
  const response = await api.post('/auth/login', { username, password });
  return response.data;
};

// 创建项目
const createProject = async (projectData) => {
  const response = await api.post('/projects', projectData);
  return response.data;
};

// 获取项目列表
const getProjects = async (params) => {
  const response = await api.get('/projects', { params });
  return response.data;
};
```

## 13. 接口测试

建议使用以下工具进行接口测试：
- Postman
- Swagger UI (访问 http://localhost:8080/swagger-ui.html)
- curl
- httpie

## 14. 接口版本管理

- 当前版本：v1
- URL 包含版本号：`/api/v1/...`
- 新版本不向后兼容时，增加版本号：`/api/v2/...`
- 旧版本保持至少6个月的兼容期
