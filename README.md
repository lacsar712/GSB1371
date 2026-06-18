# 学生选课系统

## 🛠 技术栈

- **Frontend**: 纯静态 HTML + 原生 CSS + 原生 JavaScript
- **Backend**: Node.js + Express + Sequelize
- **Database**: SQLite（UTF-8 编码，完整支持中文）

## 🚀 启动指南 (How to Run)

1. 确保 Docker Desktop 已启动。
2. 在根目录执行：`docker compose up --build`
3. 等待容器启动完成：
   - 后端会自动创建 SQLite 数据库并执行建表与 Seed。
   - 前端会在后端健康检查通过后才启动，确保服务可用。
4. 查看服务健康状态：`docker compose ps`

> **启动可靠性说明**：
> - 两服务均配置了 `restart: unless-stopped`，异常退出后会自动重启。
> - 前端通过 `depends_on` + `condition: service_healthy` 等待后端健康检查通过后再启动，避免启动时序问题。

## 🔗 服务地址 (Services)

- **Frontend**: http://localhost:31371
- **Backend API**: http://localhost:8137
- **Health Check**: http://localhost:8137/api/auth/ping
- **Database**: SQLite 文件挂载于后端容器 `/app/data/course.sqlite`（通过 Volume `backend_data` 持久化）

## 🔧 环境变量 (Environment Variables)

所有环境变量均使用 `UPPER_SNAKE_CASE` 命名：

| 变量名         | 说明                 | 默认值                  |
|----------------|----------------------|-------------------------|
| `PORT`         | 后端服务端口         | `8137`                  |
| `SQLITE_PATH`  | SQLite 数据库文件路径 | `/app/data/course.sqlite` |
| `LOG_LEVEL`    | 日志级别             | `info`                  |

## 🔍 排障指南 (Troubleshooting)

### 查看容器状态
```bash
docker compose ps
```

### 查看服务日志
```bash
# 查看所有服务日志
docker compose logs

# 仅查看后端日志（跟随输出）
docker compose logs -f backend

# 仅查看前端日志
docker compose logs frontend
```

### 健康检查失败
- 确认后端容器是否正常运行：`docker compose ps backend`
- 查看后端启动日志：`docker compose logs backend`
- 手动测试健康检查端点：`curl http://localhost:8137/api/auth/ping`
- 重启后端服务：`docker compose restart backend`

### 前端无法访问后端
- 确认后端健康检查状态：`docker compose ps backend`（`STATUS` 列应显示 `healthy`）
- 确认前端已启动：`docker compose ps frontend`
- 检查 Nginx 代理配置：前端通过 `/api` 路径代理到 `backend:8137`

### 重置数据
如需完全重置数据库，删除 Volume 后重建：
```bash
docker compose down -v
docker compose up --build
```

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
- 后端使用 `npm ci` 安装依赖。
- SQLite 数据持久化：Volume `backend_data`（挂载到后端 `/app/data`）。
- 前端通过 Nginx 将 `/api` 代理到 `http://backend:8137`，浏览器访问同一域名无需 CORS。
