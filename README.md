# 学生选课系统

## 🛠 技术栈

- **Frontend**: 纯静态 HTML + 原生 CSS + 原生 JavaScript
- **Backend**: Node.js + Express + Sequelize
- **Database**: SQLite（UTF-8 编码，完整支持中文）

## 🚀 启动指南 (How to Run)

1. 确保 Docker Desktop 已启动。
2. 在根目录执行：`docker compose up --build -d`
3. 等待容器启动完成：
   - 后端先启动，自动创建 SQLite 数据库并执行建表与 Seed
   - 后端健康检查通过后，前端才会启动（约 20-30 秒）
4. 查看启动状态：`docker compose ps`

## ⚙️ 环境变量 (Environment Variables)

所有环境变量统一使用 **UPPER_SNAKE_CASE** 命名规范：

| 变量名         | 说明                  | 默认值               |
|----------------|-----------------------|----------------------|
| `SQLITE_PATH`  | SQLite 数据库文件路径 | `/app/data/course.sqlite` |
| `PORT`         | 后端服务监听端口      | `8137`               |
| `LOG_LEVEL`    | 日志级别              | `info`               |

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

## 🔧 排障指南 (Troubleshooting)

### 服务启动顺序与健康检查

- **前端启动延迟**：前端依赖后端健康检查通过后才启动，属正常现象，等待约 20-30 秒即可。
- **查看服务状态**：`docker compose ps`，确认 `backend` 状态为 `healthy`，`frontend` 状态为 `Up`。

### 查看容器日志

```bash
# 查看所有服务日志
docker compose logs -f

# 仅查看后端日志
docker compose logs -f backend

# 仅查看前端日志
docker compose logs -f frontend
```

### 后端健康检查失败

1. 检查后端日志：`docker compose logs backend`
2. 确认端口未被占用：`netstat -ano | findstr :8137`
3. 手动执行健康检查：`docker compose exec backend npm run health`
4. 重启后端服务：`docker compose restart backend`

### 前端无法访问后端 API

1. 确认后端健康状态：`docker compose ps backend`
2. 检查后端日志是否有报错
3. 直接访问后端 API 测试：`curl http://localhost:8137/api/auth/ping`
4. 重启前端服务：`docker compose restart frontend`

### 服务异常退出

- 两服务均配置了 `restart: unless-stopped`，异常退出后会自动重启。
- 如需停止自动重启，手动停止服务：`docker compose stop`
- 如需彻底移除容器：`docker compose down`

### 数据重置

如需重置数据库数据：

```bash
docker compose down -v
docker compose up --build -d
```
