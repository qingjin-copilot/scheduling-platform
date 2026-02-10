# 数据库设计文档

## 1. 数据库概述

本系统使用 MySQL 8.0 作为主数据库，采用 InnoDB 存储引擎，字符集为 utf8mb4。

## 2. 数据表设计

### 2.1 用户表 (user)

| 字段名 | 类型 | 长度 | 是否为空 | 默认值 | 说明 |
|--------|------|------|----------|--------|------|
| id | BIGINT | - | NOT NULL | AUTO_INCREMENT | 主键 |
| username | VARCHAR | 50 | NOT NULL | - | 用户名，唯一 |
| email | VARCHAR | 100 | NOT NULL | - | 邮箱，唯一 |
| password | VARCHAR | 255 | NOT NULL | - | 密码（加密） |
| avatar | VARCHAR | 255 | NULL | - | 头像URL |
| nickname | VARCHAR | 50 | NULL | - | 昵称 |
| role | ENUM | - | NOT NULL | 'USER' | 角色：ADMIN, USER |
| status | ENUM | - | NOT NULL | 'ACTIVE' | 状态：ACTIVE, INACTIVE, LOCKED |
| created_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

**索引：**
- PRIMARY KEY (id)
- UNIQUE KEY uk_username (username)
- UNIQUE KEY uk_email (email)
- INDEX idx_status (status)

### 2.2 项目表 (project)

| 字段名 | 类型 | 长度 | 是否为空 | 默认值 | 说明 |
|--------|------|------|----------|--------|------|
| id | BIGINT | - | NOT NULL | AUTO_INCREMENT | 主键 |
| user_id | BIGINT | - | NOT NULL | - | 用户ID，外键 |
| name | VARCHAR | 100 | NOT NULL | - | 项目名称 |
| type | ENUM | - | NOT NULL | - | 项目类型：FRONTEND, BACKEND |
| description | TEXT | - | NULL | - | 项目描述 |
| template_repo | VARCHAR | 255 | NOT NULL | - | 模板仓库地址 |
| project_path | VARCHAR | 255 | NULL | - | 项目本地路径 |
| status | ENUM | - | NOT NULL | 'INITIALIZING' | 状态：INITIALIZING, ACTIVE, ARCHIVED, FAILED |
| created_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

**索引：**
- PRIMARY KEY (id)
- INDEX idx_user_id (user_id)
- INDEX idx_status (status)
- INDEX idx_type (type)
- INDEX idx_created_at (created_at)

**外键：**
- FOREIGN KEY (user_id) REFERENCES user(id) ON DELETE CASCADE

### 2.3 UI任务表 (ui_task)

| 字段名 | 类型 | 长度 | 是否为空 | 默认值 | 说明 |
|--------|------|------|----------|--------|------|
| id | BIGINT | - | NOT NULL | AUTO_INCREMENT | 主键 |
| project_id | BIGINT | - | NOT NULL | - | 项目ID，外键 |
| name | VARCHAR | 100 | NOT NULL | - | 任务名称 |
| description | TEXT | - | NULL | - | 任务描述 |
| platform | ENUM | - | NOT NULL | 'FIGMA' | 第三方平台：FIGMA, SKETCH |
| status | ENUM | - | NOT NULL | 'PENDING' | 状态：PENDING, FETCHING, COMPLETED, FAILED |
| total_components | INT | - | NOT NULL | 0 | 总组件数 |
| fetched_components | INT | - | NOT NULL | 0 | 已获取组件数 |
| created_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

**索引：**
- PRIMARY KEY (id)
- INDEX idx_project_id (project_id)
- INDEX idx_status (status)
- INDEX idx_created_at (created_at)

**外键：**
- FOREIGN KEY (project_id) REFERENCES project(id) ON DELETE CASCADE

### 2.4 UI组件表 (ui_component)

| 字段名 | 类型 | 长度 | 是否为空 | 默认值 | 说明 |
|--------|------|------|----------|--------|------|
| id | BIGINT | - | NOT NULL | AUTO_INCREMENT | 主键 |
| task_id | BIGINT | - | NOT NULL | - | 任务ID，外键 |
| name | VARCHAR | 100 | NOT NULL | - | 组件名称 |
| url | VARCHAR | 500 | NOT NULL | - | 组件链接 |
| source_code | LONGTEXT | - | NULL | - | 组件源码 |
| screenshot_url | VARCHAR | 500 | NULL | - | 截图URL |
| properties | JSON | - | NULL | - | 组件属性（宽高、颜色等） |
| status | ENUM | - | NOT NULL | 'PENDING' | 状态：PENDING, FETCHED, FAILED |
| error_message | TEXT | - | NULL | - | 错误信息 |
| created_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

**索引：**
- PRIMARY KEY (id)
- INDEX idx_task_id (task_id)
- INDEX idx_status (status)

**外键：**
- FOREIGN KEY (task_id) REFERENCES ui_task(id) ON DELETE CASCADE

### 2.5 编码任务表 (coding_task)

| 字段名 | 类型 | 长度 | 是否为空 | 默认值 | 说明 |
|--------|------|------|----------|--------|------|
| id | BIGINT | - | NOT NULL | AUTO_INCREMENT | 主键 |
| project_id | BIGINT | - | NOT NULL | - | 项目ID，外键 |
| name | VARCHAR | 100 | NOT NULL | - | 任务名称 |
| description | TEXT | - | NULL | - | 任务描述 |
| ui_component_ids | JSON | - | NOT NULL | - | UI组件ID列表 |
| ai_model | VARCHAR | 50 | NOT NULL | 'GPT-4' | AI模型 |
| ai_agent_command | TEXT | - | NULL | - | AI Agent CLI命令 |
| custom_instructions | TEXT | - | NULL | - | 自定义指令 |
| status | ENUM | - | NOT NULL | 'PENDING' | 状态：PENDING, RUNNING, PAUSED, COMPLETED, FAILED, CANCELLED |
| progress | INT | - | NOT NULL | 0 | 进度（0-100） |
| process_id | VARCHAR | 50 | NULL | - | 进程ID |
| logs | LONGTEXT | - | NULL | - | 任务日志 |
| error_message | TEXT | - | NULL | - | 错误信息 |
| started_at | TIMESTAMP | - | NULL | - | 开始时间 |
| completed_at | TIMESTAMP | - | NULL | - | 完成时间 |
| created_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

**索引：**
- PRIMARY KEY (id)
- INDEX idx_project_id (project_id)
- INDEX idx_status (status)
- INDEX idx_created_at (created_at)

**外键：**
- FOREIGN KEY (project_id) REFERENCES project(id) ON DELETE CASCADE

### 2.6 部署表 (deployment)

| 字段名 | 类型 | 长度 | 是否为空 | 默认值 | 说明 |
|--------|------|------|----------|--------|------|
| id | BIGINT | - | NOT NULL | AUTO_INCREMENT | 主键 |
| project_id | BIGINT | - | NOT NULL | - | 项目ID，外键 |
| version | VARCHAR | 50 | NOT NULL | - | 版本号 |
| environment | ENUM | - | NOT NULL | 'DEV' | 环境：DEV, STAGING, PRODUCTION |
| branch | VARCHAR | 100 | NOT NULL | 'main' | 分支名 |
| config | JSON | - | NULL | - | 部署配置（端口、内存、CPU等） |
| env_variables | JSON | - | NULL | - | 环境变量 |
| status | ENUM | - | NOT NULL | 'PENDING' | 状态：PENDING, BUILDING, DEPLOYING, SUCCESS, FAILED, ROLLED_BACK |
| deploy_url | VARCHAR | 500 | NULL | - | 部署URL |
| docker_image | VARCHAR | 255 | NULL | - | Docker镜像名 |
| container_id | VARCHAR | 100 | NULL | - | 容器ID |
| logs | LONGTEXT | - | NULL | - | 部署日志 |
| error_message | TEXT | - | NULL | - | 错误信息 |
| started_at | TIMESTAMP | - | NULL | - | 开始时间 |
| completed_at | TIMESTAMP | - | NULL | - | 完成时间 |
| duration_seconds | INT | - | NULL | - | 耗时（秒） |
| created_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

**索引：**
- PRIMARY KEY (id)
- INDEX idx_project_id (project_id)
- INDEX idx_environment (environment)
- INDEX idx_status (status)
- INDEX idx_created_at (created_at)

**外键：**
- FOREIGN KEY (project_id) REFERENCES project(id) ON DELETE CASCADE

### 2.7 项目活动日志表 (project_activity)

| 字段名 | 类型 | 长度 | 是否为空 | 默认值 | 说明 |
|--------|------|------|----------|--------|------|
| id | BIGINT | - | NOT NULL | AUTO_INCREMENT | 主键 |
| project_id | BIGINT | - | NOT NULL | - | 项目ID，外键 |
| user_id | BIGINT | - | NOT NULL | - | 用户ID，外键 |
| action | VARCHAR | 50 | NOT NULL | - | 操作类型 |
| entity_type | VARCHAR | 50 | NULL | - | 实体类型（UI_TASK, CODING_TASK等） |
| entity_id | BIGINT | - | NULL | - | 实体ID |
| description | TEXT | - | NOT NULL | - | 活动描述 |
| metadata | JSON | - | NULL | - | 元数据 |
| created_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP | 创建时间 |

**索引：**
- PRIMARY KEY (id)
- INDEX idx_project_id (project_id)
- INDEX idx_user_id (user_id)
- INDEX idx_created_at (created_at)

**外键：**
- FOREIGN KEY (project_id) REFERENCES project(id) ON DELETE CASCADE
- FOREIGN KEY (user_id) REFERENCES user(id) ON DELETE CASCADE

### 2.8 系统配置表 (system_config)

| 字段名 | 类型 | 长度 | 是否为空 | 默认值 | 说明 |
|--------|------|------|----------|--------|------|
| id | BIGINT | - | NOT NULL | AUTO_INCREMENT | 主键 |
| config_key | VARCHAR | 100 | NOT NULL | - | 配置键，唯一 |
| config_value | TEXT | - | NOT NULL | - | 配置值 |
| description | VARCHAR | 255 | NULL | - | 配置说明 |
| is_encrypted | TINYINT | 1 | NOT NULL | 0 | 是否加密（0-否，1-是） |
| created_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

**索引：**
- PRIMARY KEY (id)
- UNIQUE KEY uk_config_key (config_key)

### 2.9 API密钥表 (api_key)

| 字段名 | 类型 | 长度 | 是否为空 | 默认值 | 说明 |
|--------|------|------|----------|--------|------|
| id | BIGINT | - | NOT NULL | AUTO_INCREMENT | 主键 |
| user_id | BIGINT | - | NOT NULL | - | 用户ID，外键 |
| platform | VARCHAR | 50 | NOT NULL | - | 平台名称（FIGMA, AI_AGENT等） |
| api_key | VARCHAR | 255 | NOT NULL | - | API密钥（加密） |
| is_active | TINYINT | 1 | NOT NULL | 1 | 是否启用（0-否，1-是） |
| created_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | - | NOT NULL | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

**索引：**
- PRIMARY KEY (id)
- INDEX idx_user_id (user_id)
- INDEX idx_platform (platform)
- UNIQUE KEY uk_user_platform (user_id, platform)

**外键：**
- FOREIGN KEY (user_id) REFERENCES user(id) ON DELETE CASCADE

## 3. 数据库初始化脚本

```sql
-- 创建数据库
CREATE DATABASE IF NOT EXISTS scheduling_platform 
DEFAULT CHARACTER SET utf8mb4 
DEFAULT COLLATE utf8mb4_unicode_ci;

USE scheduling_platform;

-- 1. 用户表
CREATE TABLE `user` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `username` VARCHAR(50) NOT NULL COMMENT '用户名',
  `email` VARCHAR(100) NOT NULL COMMENT '邮箱',
  `password` VARCHAR(255) NOT NULL COMMENT '密码（加密）',
  `avatar` VARCHAR(255) DEFAULT NULL COMMENT '头像URL',
  `nickname` VARCHAR(50) DEFAULT NULL COMMENT '昵称',
  `role` ENUM('ADMIN', 'USER') NOT NULL DEFAULT 'USER' COMMENT '角色',
  `status` ENUM('ACTIVE', 'INACTIVE', 'LOCKED') NOT NULL DEFAULT 'ACTIVE' COMMENT '状态',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_username` (`username`),
  UNIQUE KEY `uk_email` (`email`),
  KEY `idx_status` (`status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户表';

-- 2. 项目表
CREATE TABLE `project` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_id` BIGINT NOT NULL COMMENT '用户ID',
  `name` VARCHAR(100) NOT NULL COMMENT '项目名称',
  `type` ENUM('FRONTEND', 'BACKEND') NOT NULL COMMENT '项目类型',
  `description` TEXT DEFAULT NULL COMMENT '项目描述',
  `template_repo` VARCHAR(255) NOT NULL COMMENT '模板仓库地址',
  `project_path` VARCHAR(255) DEFAULT NULL COMMENT '项目本地路径',
  `status` ENUM('INITIALIZING', 'ACTIVE', 'ARCHIVED', 'FAILED') NOT NULL DEFAULT 'INITIALIZING' COMMENT '状态',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_status` (`status`),
  KEY `idx_type` (`type`),
  KEY `idx_created_at` (`created_at`),
  CONSTRAINT `fk_project_user` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='项目表';

-- 3. UI任务表
CREATE TABLE `ui_task` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `project_id` BIGINT NOT NULL COMMENT '项目ID',
  `name` VARCHAR(100) NOT NULL COMMENT '任务名称',
  `description` TEXT DEFAULT NULL COMMENT '任务描述',
  `platform` ENUM('FIGMA', 'SKETCH') NOT NULL DEFAULT 'FIGMA' COMMENT '第三方平台',
  `status` ENUM('PENDING', 'FETCHING', 'COMPLETED', 'FAILED') NOT NULL DEFAULT 'PENDING' COMMENT '状态',
  `total_components` INT NOT NULL DEFAULT 0 COMMENT '总组件数',
  `fetched_components` INT NOT NULL DEFAULT 0 COMMENT '已获取组件数',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_project_id` (`project_id`),
  KEY `idx_status` (`status`),
  KEY `idx_created_at` (`created_at`),
  CONSTRAINT `fk_uitask_project` FOREIGN KEY (`project_id`) REFERENCES `project` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='UI任务表';

-- 4. UI组件表
CREATE TABLE `ui_component` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `task_id` BIGINT NOT NULL COMMENT '任务ID',
  `name` VARCHAR(100) NOT NULL COMMENT '组件名称',
  `url` VARCHAR(500) NOT NULL COMMENT '组件链接',
  `source_code` LONGTEXT DEFAULT NULL COMMENT '组件源码',
  `screenshot_url` VARCHAR(500) DEFAULT NULL COMMENT '截图URL',
  `properties` JSON DEFAULT NULL COMMENT '组件属性',
  `status` ENUM('PENDING', 'FETCHED', 'FAILED') NOT NULL DEFAULT 'PENDING' COMMENT '状态',
  `error_message` TEXT DEFAULT NULL COMMENT '错误信息',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_task_id` (`task_id`),
  KEY `idx_status` (`status`),
  CONSTRAINT `fk_component_uitask` FOREIGN KEY (`task_id`) REFERENCES `ui_task` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='UI组件表';

-- 5. 编码任务表
CREATE TABLE `coding_task` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `project_id` BIGINT NOT NULL COMMENT '项目ID',
  `name` VARCHAR(100) NOT NULL COMMENT '任务名称',
  `description` TEXT DEFAULT NULL COMMENT '任务描述',
  `ui_component_ids` JSON NOT NULL COMMENT 'UI组件ID列表',
  `ai_model` VARCHAR(50) NOT NULL DEFAULT 'GPT-4' COMMENT 'AI模型',
  `ai_agent_command` TEXT DEFAULT NULL COMMENT 'AI Agent CLI命令',
  `custom_instructions` TEXT DEFAULT NULL COMMENT '自定义指令',
  `status` ENUM('PENDING', 'RUNNING', 'PAUSED', 'COMPLETED', 'FAILED', 'CANCELLED') NOT NULL DEFAULT 'PENDING' COMMENT '状态',
  `progress` INT NOT NULL DEFAULT 0 COMMENT '进度',
  `process_id` VARCHAR(50) DEFAULT NULL COMMENT '进程ID',
  `logs` LONGTEXT DEFAULT NULL COMMENT '任务日志',
  `error_message` TEXT DEFAULT NULL COMMENT '错误信息',
  `started_at` TIMESTAMP NULL DEFAULT NULL COMMENT '开始时间',
  `completed_at` TIMESTAMP NULL DEFAULT NULL COMMENT '完成时间',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_project_id` (`project_id`),
  KEY `idx_status` (`status`),
  KEY `idx_created_at` (`created_at`),
  CONSTRAINT `fk_codingtask_project` FOREIGN KEY (`project_id`) REFERENCES `project` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='编码任务表';

-- 6. 部署表
CREATE TABLE `deployment` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `project_id` BIGINT NOT NULL COMMENT '项目ID',
  `version` VARCHAR(50) NOT NULL COMMENT '版本号',
  `environment` ENUM('DEV', 'STAGING', 'PRODUCTION') NOT NULL DEFAULT 'DEV' COMMENT '环境',
  `branch` VARCHAR(100) NOT NULL DEFAULT 'main' COMMENT '分支名',
  `config` JSON DEFAULT NULL COMMENT '部署配置',
  `env_variables` JSON DEFAULT NULL COMMENT '环境变量',
  `status` ENUM('PENDING', 'BUILDING', 'DEPLOYING', 'SUCCESS', 'FAILED', 'ROLLED_BACK') NOT NULL DEFAULT 'PENDING' COMMENT '状态',
  `deploy_url` VARCHAR(500) DEFAULT NULL COMMENT '部署URL',
  `docker_image` VARCHAR(255) DEFAULT NULL COMMENT 'Docker镜像名',
  `container_id` VARCHAR(100) DEFAULT NULL COMMENT '容器ID',
  `logs` LONGTEXT DEFAULT NULL COMMENT '部署日志',
  `error_message` TEXT DEFAULT NULL COMMENT '错误信息',
  `started_at` TIMESTAMP NULL DEFAULT NULL COMMENT '开始时间',
  `completed_at` TIMESTAMP NULL DEFAULT NULL COMMENT '完成时间',
  `duration_seconds` INT DEFAULT NULL COMMENT '耗时（秒）',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_project_id` (`project_id`),
  KEY `idx_environment` (`environment`),
  KEY `idx_status` (`status`),
  KEY `idx_created_at` (`created_at`),
  CONSTRAINT `fk_deployment_project` FOREIGN KEY (`project_id`) REFERENCES `project` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='部署表';

-- 7. 项目活动日志表
CREATE TABLE `project_activity` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `project_id` BIGINT NOT NULL COMMENT '项目ID',
  `user_id` BIGINT NOT NULL COMMENT '用户ID',
  `action` VARCHAR(50) NOT NULL COMMENT '操作类型',
  `entity_type` VARCHAR(50) DEFAULT NULL COMMENT '实体类型',
  `entity_id` BIGINT DEFAULT NULL COMMENT '实体ID',
  `description` TEXT NOT NULL COMMENT '活动描述',
  `metadata` JSON DEFAULT NULL COMMENT '元数据',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`),
  KEY `idx_project_id` (`project_id`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_created_at` (`created_at`),
  CONSTRAINT `fk_activity_project` FOREIGN KEY (`project_id`) REFERENCES `project` (`id`) ON DELETE CASCADE,
  CONSTRAINT `fk_activity_user` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='项目活动日志表';

-- 8. 系统配置表
CREATE TABLE `system_config` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `config_key` VARCHAR(100) NOT NULL COMMENT '配置键',
  `config_value` TEXT NOT NULL COMMENT '配置值',
  `description` VARCHAR(255) DEFAULT NULL COMMENT '配置说明',
  `is_encrypted` TINYINT(1) NOT NULL DEFAULT 0 COMMENT '是否加密',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_config_key` (`config_key`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='系统配置表';

-- 9. API密钥表
CREATE TABLE `api_key` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_id` BIGINT NOT NULL COMMENT '用户ID',
  `platform` VARCHAR(50) NOT NULL COMMENT '平台名称',
  `api_key` VARCHAR(255) NOT NULL COMMENT 'API密钥（加密）',
  `is_active` TINYINT(1) NOT NULL DEFAULT 1 COMMENT '是否启用',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_platform` (`platform`),
  UNIQUE KEY `uk_user_platform` (`user_id`, `platform`),
  CONSTRAINT `fk_apikey_user` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='API密钥表';

-- 插入默认系统配置
INSERT INTO `system_config` (`config_key`, `config_value`, `description`) VALUES
('default_frontend_template', 'https://github.com/example/vue3-template.git', '默认Vue3前端模板仓库'),
('default_backend_template', 'https://github.com/example/springboot-template.git', '默认Spring Boot后端模板仓库'),
('max_concurrent_tasks', '5', '最大并发任务数'),
('task_timeout_minutes', '60', '任务超时时间（分钟）');
```

## 4. ER 图

```
┌─────────────┐
│    user     │
└──────┬──────┘
       │
       │ 1:N
       │
┌──────▼──────┐     1:N     ┌─────────────┐
│   project   ├─────────────►│  ui_task    │
└──────┬──────┘             └──────┬──────┘
       │                           │
       │ 1:N                       │ 1:N
       │                           │
       │                    ┌──────▼───────┐
       │                    │ ui_component │
       │                    └──────────────┘
       │
       │ 1:N
       │
┌──────▼──────────┐
│  coding_task    │
└─────────────────┘
       │
       │ 1:N
       │
┌──────▼──────┐
│ deployment  │
└─────────────┘
       │
       │ 1:N
       │
┌──────▼────────────┐
│ project_activity  │
└───────────────────┘
```

## 5. 数据完整性约束

### 5.1 外键约束
- 所有外键关系使用 `ON DELETE CASCADE`，删除父记录时自动删除子记录
- 保证数据一致性和引用完整性

### 5.2 唯一性约束
- 用户名和邮箱全局唯一
- 配置键全局唯一
- 用户+平台的API密钥唯一

### 5.3 默认值
- 时间戳字段自动设置当前时间
- 状态字段有合理的默认值
- 进度默认为0

## 6. 数据备份策略

### 6.1 备份类型
- **全量备份**：每天凌晨2点执行
- **增量备份**：每小时执行一次
- **binlog备份**：实时备份

### 6.2 备份保留
- 全量备份保留30天
- 增量备份保留7天
- binlog保留7天

### 6.3 恢复演练
- 每月进行一次恢复演练
- 确保备份数据可用性

## 7. 性能优化建议

### 7.1 索引优化
- 为常用查询字段添加索引
- 避免过多索引影响写入性能
- 定期分析慢查询并优化

### 7.2 分区策略
- 对日志表按时间进行分区
- 提高查询和维护效率

### 7.3 读写分离
- 主库处理写操作
- 从库处理读操作
- 使用中间件实现自动路由

## 8. 数据安全

### 8.1 敏感数据加密
- 密码使用 BCrypt 加密
- API密钥使用 AES 加密
- HTTPS 传输加密

### 8.2 访问控制
- 数据库用户权限最小化
- 应用层访问控制
- SQL 注入防护

### 8.3 审计日志
- 记录所有数据修改操作
- 保留操作人和操作时间
- 定期审计和分析
