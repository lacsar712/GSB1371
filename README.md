# 学生选课系统

## 🛠 技术栈

- **Frontend**: 纯静态 HTML + 原生 CSS + 原生 JavaScript
- **Backend**: Node.js + Express + Sequelize
- **Database**: SQLite（UTF-8 编码，完整支持中文）

## 🚀 启动指南 (How to Run)

1. 确保 Docker Desktop 已启动。
2. 在根目录执行：`docker compose up --build`
3. 后端容器启动后会自动创建 SQLite 数据库并执行建表与 Seed，随后通过健康检查端点 `/api/auth/ping` 验证服务就绪。
4. 前端容器会**等待后端健康检查通过后**再启动，确保访问时后端已完全可用。
5. 两个服务均配置了 `restart: unless-stopped`，异常退出后会自动重启。

启动完成后，前端地址：http://localhost:31371

## ⚙️ 环境变量 (Environment Variables)

所有环境变量名均采用 `UPPER_SNAKE_CASE` 命名规范。

### Backend 环境变量

| 变量名         | 默认值                  | 说明                     |
|----------------|-------------------------|--------------------------|
| `PORT`         | `8137`                  | 后端服务监听端口         |
| `SQLITE_PATH`  | `/app/data/course.sqlite` | SQLite 数据库文件路径  |
| `LOG_LEVEL`    | `info`                  | 日志级别（debug/info/warn/error） |

## 🔗 服务地址 (Services)

- **Frontend**: http://localhost:31371
- **Backend API**: http://localhost:8137
- **Health Check**: http://localhost:8137/api/auth/ping
- **Database**: SQLite 文件挂载于后端容器 `/app/data/course.sqlite`（通过 Volume `backend_data` 持久化）

## 🔍 排障指南 (Troubleshooting)

### 查看服务状态与健康检查

```bash
# 查看所有容器状态（含健康检查状态）
docker compose ps

# 查看后端健康检查详情
docker inspect gsb1371-backend-1 --format='{{json .State.Health}}'
```

### 查看日志

```bash
# 查看所有服务日志
docker compose logs -f

# 仅查看后端日志
docker compose logs -f backend

# 仅查看前端日志
docker compose logs -f frontend
```

### 常见问题

**1. 前端页面无法访问或显示 502？**
- 确认后端健康检查是否通过：`docker compose ps` 中 backend 的 `STATUS` 应为 `healthy`
- 若后端为 `starting` 或 `unhealthy`，查看后端日志排查启动失败原因
- 前端 Nginx 会将 `/api` 请求代理到 `backend:8137`，后端未就绪时会返回 502

**2. 后端反复重启？**
- 查看后端日志：`docker compose logs backend`
- 常见原因：数据库初始化失败、端口被占用、依赖缺失
- 可尝试删除 Volume 后重建：`docker compose down -v && docker compose up --build`

**3. 前端容器一直不启动？**
- 前端依赖后端健康检查通过，检查后端状态：`docker compose ps backend`
- 若后端长时间处于 `unhealthy`，进入后端容器手动测试健康检查：
  ```bash
  docker compose exec backend npm run healthcheck
  ```

**4. 数据重置（清空数据库重建）？**
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
