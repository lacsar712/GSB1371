# 学生选课系统

## 🛠 技术栈

- **Frontend**: 纯静态 HTML + 原生 CSS + 原生 JavaScript
- **Backend**: Node.js + Express + Sequelize
- **Database**: SQLite（UTF-8 编码，完整支持中文）

## 🚀 启动指南 (How to Run)

1. 确保 Docker Desktop 已启动。
2. 在根目录执行：`docker compose up --build`
3. 等待容器启动完成（后端会自动创建 SQLite 数据库并执行建表与 Seed）。
4. 前端容器会在后端健康检查通过后自动启动，无需手动干预。

> **启动顺序说明**：`frontend` 配置了 `depends_on.backend.condition: service_healthy`，只有当后端 `/api/auth/ping` 健康检查连续通过后，前端才会启动。两个服务均设置了 `restart: unless-stopped`，异常退出后会自动重启。

## 🔗 服务地址 (Services)

- **Frontend**: http://localhost:31371
- **Backend API**: http://localhost:8137
- **Database**: SQLite 文件挂载于后端容器 `/app/data/course.sqlite`（通过 Volume `backend_data` 持久化）

## 🧪 测试账号

| 角色   | 用户名/学号 | 密码   |
|--------|-------------|--------|
| 管理员 | admin       | 123456 |
| 学生   | S2024001    | 123456 |
| 学生   | S2024002    | 123456 |
| 学生   | S2024003    | 123456 |

## 数据库与字符集

- 使用 **SQLite 3**，默认 **UTF-8** 编码，完整支持中文。
- 数据文件路径可通过环境变量 `SQLITE_PATH` 配置（默认 `backend/data/course.sqlite`）。
- API 响应头统一为：`Content-Type: application/json; charset=utf-8`。

## 🔧 环境变量

| 变量名 | 说明 | 默认值 | 所在服务 |
|--------|------|--------|----------|
| `SQLITE_PATH` | SQLite 数据库文件路径 | `/app/data/course.sqlite` | backend |
| `PORT` | 后端监听端口 | `8137` | backend |

> 所有环境变量均使用 `UPPER_SNAKE_CASE` 命名，在 `docker-compose.yml` 的 `environment` 中配置。

## 🏥 健康检查

- 后端内置健康检查端点：`GET /api/auth/ping`，返回 `{ "ok": true, "message": "pong" }`。
- Docker Compose 中后端服务配置了 `healthcheck`，每 5 秒探测一次，超时 3 秒，最多重试 5 次，启动等待 20 秒。
- 本地开发时可通过 `npm run healthcheck`（在 `backend/` 目录下）手动执行健康检查。

## 🐛 排障指南 (Troubleshooting)

### 后端容器无法启动

1. 查看后端日志：`docker compose logs backend`
2. 常见原因：
   - 端口 8137 被占用 → 修改 `docker-compose.yml` 中 `PORT` 环境变量和端口映射。
   - SQLite 数据损坏 → 删除 Volume 重建：`docker compose down -v && docker compose up --build`
   - Node 版本不兼容 → 确认 `engines.node >= 18.0.0`，Dockerfile 使用 `node:20-alpine`。

### 前端容器未启动

1. 前端依赖后端健康检查通过后才启动，先确认后端状态：`docker compose ps`
2. 若后端显示 `unhealthy`，参照上一条排查后端问题。
3. 查看前端日志：`docker compose logs frontend`

### 前端页面无法访问后端 API

1. 确认后端健康：`curl http://localhost:8137/api/auth/ping`
2. 确认 Nginx 代理配置：前端容器内 `nginx.conf` 将 `/api/` 代理到 `http://backend:8137`。
3. 进入前端容器检查：`docker compose exec frontend sh`，然后 `curl http://backend:8137/api/auth/ping`。

### 容器频繁重启

1. 两个服务均设置了 `restart: unless-stopped`，异常退出会自动重启。
2. 查看重启原因：`docker compose logs --tail 50 <service_name>`
3. 若需停止自动重启：`docker compose stop <service_name>`

### 重置所有数据

```bash
docker compose down -v        # 停止容器并删除 Volume
docker compose up --build     # 重新构建并启动
```

## 项目结构

```
├── backend/           # Node + Express + Sequelize
│   ├── src/
│   │   ├── index.js   # 入口、CORS、路由
│   │   ├── logger.js  # Winston 日志
│   │   ├── db.js      # 密码哈希
│   │   ├── seed.js    # 建表 + Seed
│   │   ├── models/    # Sequelize 模型
│   │   ├── routes/    # auth / courses / enrollment / admin
│   │   └── middleware/
│   ├── Dockerfile
│   └── package.json
├── frontend/           # 纯静态 HTML + CSS + JS
│   ├── index.html      # 登录页（表单直接调用后端 API）
│   ├── student.html    # 学生端课程选课页面
│   ├── admin.html      # 管理端课程管理页面
│   ├── css/style.css   # 所有页面共用的样式
│   ├── js/auth.js      # 登录逻辑（原生 JS，fetch 调用后端）
│   ├── js/student.js   # 学生端操作逻辑
│   ├── js/admin.js     # 管理端操作逻辑
│   ├── nginx.conf      # 代理 /api 到 backend
│   └── Dockerfile
├── docker-compose.yml  # backend（SQLite）+ frontend
└── README.md
```

## Docker 构建说明

- 前端为纯静态资源（HTML/CSS/JS），无需 npm 构建，直接由 Nginx 托管。
- 后端使用 `npm ci` 安装依赖，要求 Node.js >= 18.0.0（`engines` 字段锁定）。
- SQLite 数据持久化：Volume `backend_data`（挂载到后端 `/app/data`）。
- 前端通过 Nginx 将 `/api` 代理到 `http://backend:8137`，浏览器访问同一域名无需 CORS。
- 两个服务均配置 `restart: unless-stopped`，异常退出后自动重启。
- 前端启动依赖后端健康检查通过（`condition: service_healthy`）。
