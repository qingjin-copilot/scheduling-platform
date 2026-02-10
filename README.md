# AI 开发工作流调度平台

一个基于 AI 的自动化开发平台，通过集成 UI 设计、AI 编码和项目部署，实现从设计到部署的完整开发流程自动化。

## 项目简介

AI 开发工作流调度平台提供了一套完整的解决方案，帮助开发者快速将 UI 设计转化为可运行的代码，并自动部署到生产环境。

### 核心功能

- **项目管理**：支持创建前端（Vue3）和后端（Spring Boot）项目，从模板仓库快速初始化
- **UI 任务管理**：批量获取第三方平台的 UI 组件源码和截图
- **编码任务管理**：基于 UI 组件自动调用 AI Agent 进行编码，实时显示进度
- **项目预览**：一键启动项目预览服务
- **项目部署**：自动化构建、Docker 镜像打包和部署

### 技术栈

**前端：**
- Vue 3 + TypeScript
- Vite
- Element Plus
- Pinia
- Vue Router

**后端：**
- Spring Boot 3.x
- MySQL 8.0
- MyBatis Plus
- Redis
- RabbitMQ
- MinIO

**部署：**
- Docker + Docker Compose
- GitHub Actions

## 文档

- [架构设计文档](docs/architecture-design.md) - 系统架构、技术栈和模块设计
- [功能设计文档](docs/functional-design.md) - 详细的功能说明和用户界面设计
- [数据库设计文档](docs/database-design.md) - 数据库表结构和 ER 图
- [API 接口文档](docs/api-specification.md) - RESTful API 和 WebSocket 接口规范

## 快速开始

### 环境要求

- Node.js 18+
- Java 17+
- MySQL 8.0+
- Redis 6.0+
- Docker 20+

### 安装步骤

1. 克隆仓库
```bash
git clone https://github.com/qingjin-copilot/scheduling-platform.git
cd scheduling-platform
```

2. 配置数据库
```bash
# 创建数据库
mysql -u root -p < docs/database-init.sql
```

3. 启动后端服务
```bash
cd backend
./mvnw spring-boot:run
```

4. 启动前端服务
```bash
cd frontend
npm install
npm run dev
```

5. 访问应用
- 前端：http://localhost:3000
- 后端 API：http://localhost:8080
- API 文档：http://localhost:8080/swagger-ui.html

## 项目结构

```
scheduling-platform/
├── docs/                       # 文档目录
│   ├── architecture-design.md  # 架构设计文档
│   ├── functional-design.md    # 功能设计文档
│   ├── database-design.md      # 数据库设计文档
│   └── api-specification.md    # API 接口文档
├── backend/                    # 后端代码（待实现）
├── frontend/                   # 前端代码（待实现）
└── README.md                   # 本文件
```

## 开发计划

- [x] 完成架构设计
- [x] 完成功能设计
- [x] 完成数据库设计
- [x] 完成 API 接口设计
- [ ] 实现后端服务
- [ ] 实现前端界面
- [ ] 集成 AI Agent CLI
- [ ] 实现部署功能
- [ ] 编写测试用例
- [ ] 完善文档

## 贡献指南

欢迎贡献代码和提出建议！请遵循以下步骤：

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 提交 Pull Request

## 许可证

本项目采用 MIT 许可证。详见 [LICENSE](LICENSE) 文件。

## 联系方式

如有问题或建议，请提交 [Issue](https://github.com/qingjin-copilot/scheduling-platform/issues)。