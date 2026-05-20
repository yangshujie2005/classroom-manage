# 智慧教室管理系统

基于 Spring Boot + Vue 3 / uni-app 的全栈智慧教室管理平台，支持教室预约、课程管理、设备报修、用户管理、数据分析等功能。

## 项目结构

```
├── admin/                    # Web 管理后台
│   ├── backend/              # Spring Boot 2.5 + Java 8
│   ├── frontend/             # Vue 3 + Vite + Element Plus
│   └── docker-compose.yml    # 一键部署（Nginx + 后端）
├── app/                      # 移动端 / 小程序
│   ├── backend/              # Spring Boot 2.5 + Java 8
│   ├── frontend/             # uni-app (Vue 3) 跨平台
│   └── docker-compose.yml    # 后端容器化部署
├── .env.development          # 前端环境变量
└── CLAUDE.md                 # Claude Code 指引
```

## 技术栈

| 层级 | 管理后台 (admin) | 移动端 (app) |
|------|-----------------|-------------|
| 后端框架 | Spring Boot 2.5 | Spring Boot 2.5 |
| ORM | MyBatis-Plus 3.5 | MyBatis-Plus 3.5 |
| 数据库 | SQLite（默认）/ MySQL | SQLite（默认）/ MySQL |
| 缓存 | Redis / Local | Redis / Local |
| 认证 | Spring Security + JWT | Spring Security + JWT |
| 前端 | Vue 3 + Element Plus | uni-app (Vue 3) + Tailwind |
| API 文档 | Knife4j (Swagger) | Knife4j (Swagger) |

## 快速开始

### 环境要求

- JDK 1.8+
- Maven 3.9+
- Node.js 18+
- Docker & Docker Compose（可选，用于容器化部署）

### 本地开发

**管理后台后端**
```bash
cd admin/backend
./mvnw spring-boot:run -P sqlite
# 启动后访问 http://localhost:8080
# API 文档: http://localhost:8080/doc.html
```

**管理后台前端**
```bash
cd admin/frontend
npm install
npm run dev
# 访问 http://localhost:5173
```

**移动端后端**
```bash
cd app/backend
./mvnw spring-boot:run -P sqlite
# 启动后访问 http://localhost:6789
```

**移动端前端（微信小程序）**
```bash
cd app/frontend
npm install
npm run dev:mp-weixin
# 在微信开发者工具中导入 dist/dev/mp-weixin
```

### Docker 部署

**管理后台一键部署**
```bash
cd admin
docker-compose up -d --build
# 访问 http://localhost/classroom/
```

**移动端后端部署**
```bash
cd app
docker-compose up -d --build
```

两个服务共享同一个 SQLite 数据库文件，确保数据一致性。

## 主要功能

- **教室管理** — 教室信息维护、教室状态查看
- **课程预约** — 课程创建、时段预约、审核流程
- **设备报修** — 在线报修申请、维修状态跟踪
- **用户管理** — 多角色权限、微信登录、手机号注册
- **数据统计** — ECharts 可视化报表
- **文件管理** — 图片/文件上传，支持本地存储和 S3（华为云 OBS）
- **AI 助手** — 基于 LLM 的智能问答（移动端）

## 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `DB_TYPE` | 数据库类型 | `sqlite` |
| `CACHE_TYPE` | 缓存类型 | `local` |
| `LOGIN_ENABLE` | 启用登录认证 | `true` |
| `UPLOAD_PATH` | 文件上传路径 | `/app/uploadPath` |
| `STORAGE_TYPE` | 文件存储方式 | `local` |
| `PORT` | 服务端口 | admin: `8080`, app: `6789` |

## 数据库切换

默认使用 SQLite，换用 MySQL：

```bash
# Maven 构建时指定 mysql profile
./mvnw clean package -P mysql

# 运行时设置环境变量
export DB_TYPE=mysql
```

MySQL 连接配置在 `src/main/resources/application-mysql.yml` 中。
