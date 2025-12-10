# ASAIR 本地运行指南

本文档按照标准启动流程，详细说明如何在本地环境运行 ASAIR 系统。

---

## 📋 启动流程概览

```mermaid
flowchart TD
    START[开始] --> STEP1[1. 下载项目代码]
    STEP1 --> STEP2[2. 分别安装依赖]
    STEP2 --> STEP3[3. 安装三方依赖]
    STEP3 --> STEP4[4. 配置相关内容]
    STEP4 --> STEP5[5. 验证启动]
    STEP5 --> SUCCESS[启动成功]
    
    STEP3 --> DOCKER[Docker 方式]
    STEP3 --> LOCAL[本地安装方式]
    
    STEP4 --> CONFIG_FE[配置前端]
    STEP4 --> CONFIG_BE[配置后端]
    STEP4 --> CONFIG_AI[配置 AI Server]
    
    style START fill:#e1f5ff
    style SUCCESS fill:#c8e6c9
    style STEP3 fill:#fff4e1
    style STEP4 fill:#ffe1f5
```

---

## 一、下载项目代码

### 1.1 克隆项目仓库

```bash
# 创建项目目录
mkdir -p ~/works/asair
cd ~/works/asair

# 克隆各个项目仓库
git clone https://github.com/zhouyuChina/asair-admin-backend.git projects/asair-admin-backend
git clone https://github.com/zhouyuChina/asair-admin-frontend.git projects/asair-admin-frontend
git clone https://github.com/zhouyuChina/asair-ai-server.git projects/asair-ai-server
git clone https://github.com/zhouyuChina/asair-infra.git projects/asair-infra
```

### 1.2 验证代码结构

```bash
# 检查项目结构
tree projects/ -L 1

# 应该看到以下目录：
# projects/
# ├── asair-admin-backend/
# ├── asair-admin-frontend/
# ├── asair-ai-server/
# └── asair-infra/
```

---

## 二、分别安装依赖

### 2.1 安装 Admin Backend 依赖

```bash
cd projects/asair-admin-backend

# 创建 Python 虚拟环境
python -m venv venv

# 激活虚拟环境
source venv/bin/activate  # Mac/Linux
# 或
# venv\Scripts\activate  # Windows

# 安装 Python 依赖
pip install -r requirements.txt
```

**依赖列表**：
- Flask>=3.0.0
- gunicorn>=21.2.0
- python-dotenv>=1.0.1
- SQLAlchemy>=2.0.0
- pymysql>=1.1.0
- PyJWT>=2.8.0
- bcrypt>=4.0.0

### 2.2 安装 AI Server 依赖

```bash
cd projects/asair-ai-server

# 创建 Python 虚拟环境
python -m venv venv

# 激活虚拟环境
source venv/bin/activate  # Mac/Linux

# 安装 Python 依赖
pip install -r requirements.txt
```

**依赖列表**：
- Flask>=3.0.0
- gunicorn>=21.2.0
- httpx>=0.27.0
- python-dotenv>=1.0.1
- pydantic>=2.0.0
- openai>=1.0.0
- sqlalchemy>=2.0.0
- pymysql>=1.1.0

### 2.3 安装 Frontend 依赖

```bash
cd projects/asair-admin-frontend

# 安装 Node.js 依赖
npm install
```

**依赖列表**：
- react: ^18.3.1
- react-dom: ^18.3.1
- react-router-dom: ^6.28.0
- axios: ^1.13.2
- vite: ^5.0.0
- @vitejs/plugin-react: ^4.3.3

---

## 三、安装三方依赖

选择以下两种方式之一：

### 方式 A：使用 Docker（推荐）

#### 3.1 安装 Docker Desktop

**Mac 系统**：
```bash
# 使用 Homebrew 安装
brew install --cask docker

# 或从官网下载
# https://www.docker.com/products/docker-desktop/
```

**验证安装**：
```bash
docker --version
docker compose version
```

#### 3.2 创建 docker-compose.yml 文件

在项目根目录（与 `projects` 同级）创建 `docker-compose.yml` 文件：

```yaml
version: '3.8'

services:
  # MySQL 数据库
  mysql:
    image: mysql:8.0
    container_name: asair-mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: asair_admin_backend
      MYSQL_USER: asair
      MYSQL_PASSWORD: asair123
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - asair-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis 缓存（可选）
  redis:
    image: redis:7-alpine
    container_name: asair-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - asair-network
    command: redis-server --appendonly yes

  # Chatwoot（自托管）
  chatwoot:
    image: chatwoot/chatwoot:latest
    container_name: asair-chatwoot
    environment:
      - RAILS_ENV=development
      - POSTGRES_HOST=chatwoot-postgres
      - POSTGRES_USERNAME=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DATABASE=chatwoot
      - REDIS_URL=redis://redis:6379
      - SECRET_KEY_BASE=your-secret-key-base-change-in-production-please-use-a-long-random-string
      - FRONTEND_URL=http://localhost:3000
      - FORCE_SSL=false
    ports:
      - "3000:3000"
    depends_on:
      - chatwoot-postgres
      - redis
    volumes:
      - chatwoot_data:/app/storage
    networks:
      - asair-network
    command: >
      sh -c "
        bundle exec rails db:chatwoot_prepare &&
        bundle exec rails s -b 0.0.0.0
      "

  # Chatwoot PostgreSQL
  chatwoot-postgres:
    image: postgres:15-alpine
    container_name: asair-chatwoot-postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: chatwoot
    volumes:
      - chatwoot_postgres_data:/var/lib/postgresql/data
    networks:
      - asair-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mysql_data:
  redis_data:
  chatwoot_data:
  chatwoot_postgres_data:

networks:
  asair-network:
    driver: bridge
```

**配置说明**：

1. **MySQL 服务**
   - 镜像：`mysql:8.0`
   - 端口：`3306:3306`（宿主机:容器）
   - 数据库：`asair_admin_backend`（默认）
   - 用户：`asair / asair123`
   - 数据持久化：`mysql_data` 卷

2. **Redis 服务**
   - 镜像：`redis:7-alpine`
   - 端口：`6379:6379`
   - 数据持久化：`redis_data` 卷（AOF 模式）

3. **Chatwoot 服务**
   - 镜像：`chatwoot/chatwoot:latest`
   - 端口：`3000:3000`
   - 环境变量：
     - `SECRET_KEY_BASE`: 需要替换为随机字符串（生产环境）
     - `FRONTEND_URL`: Chatwoot 访问地址
   - 依赖：PostgreSQL 和 Redis
   - 自动执行数据库迁移

4. **Chatwoot PostgreSQL**
   - 镜像：`postgres:15-alpine`
   - 内部服务，不暴露端口
   - 数据持久化：`chatwoot_postgres_data` 卷

#### 3.3 启动 Docker 服务

```bash
# 在项目根目录
cd ~/works/asair

# 启动所有三方服务（MySQL、Redis、Chatwoot、PostgreSQL）
docker compose up -d

# 查看服务状态
docker compose ps

# 查看服务日志
docker compose logs -f

# 查看特定服务日志
docker compose logs -f chatwoot
```

**启动顺序**：
1. MySQL 和 PostgreSQL 先启动（数据库服务）
2. Redis 启动（缓存服务）
3. Chatwoot 启动（依赖数据库和 Redis）

**首次启动注意事项**：
- Chatwoot 首次启动需要执行数据库迁移，可能需要 1-2 分钟
- 可以通过 `docker compose logs -f chatwoot` 查看迁移进度
- 等待看到 "Listening on tcp://0.0.0.0:3000" 表示启动成功

**Docker 服务包括**：
- MySQL 8.0（端口 3306）
- Redis 7（端口 6379）
- Chatwoot（端口 3000）
- Chatwoot PostgreSQL（内部使用）

---

### 方式 B：本地安装（不使用 Docker）

#### 3.1 安装 MySQL

**Mac 系统**：
```bash
# 使用 Homebrew 安装
brew install mysql

# 启动 MySQL 服务
brew services start mysql

# 验证安装
mysql --version
```

#### 3.2 安装 Redis（可选）

```bash
# 使用 Homebrew 安装
brew install redis

# 启动 Redis 服务
brew services start redis

# 验证安装
redis-cli ping
```

#### 3.3 配置数据库

```bash
# 登录 MySQL
mysql -u root -p

# 创建数据库和用户
CREATE DATABASE asair_admin_backend;
CREATE DATABASE asair_ai_server;
CREATE USER 'asair'@'localhost' IDENTIFIED BY 'asair123';
GRANT ALL PRIVILEGES ON asair_admin_backend.* TO 'asair'@'localhost';
GRANT ALL PRIVILEGES ON asair_ai_server.* TO 'asair'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

#### 3.4 安装 Chatwoot（可选，建议使用 Docker）

Chatwoot 建议使用 Docker 方式运行，因为其依赖较多。如果必须本地安装，请参考 [Chatwoot 官方文档](https://www.chatwoot.com/docs/self-hosted/overview)。

---

## 四、配置相关内容

### 4.1 配置 Admin Backend

#### 创建环境变量文件

在 `projects/asair-admin-backend/` 目录创建 `.env` 文件：

```bash
cd projects/asair-admin-backend
cat > .env << EOF
ASAIR_ADMIN_ENV=development
ASAIR_ADMIN_DEBUG=true
DATABASE_URL=mysql+pymysql://asair:asair123@localhost:3306/asair_admin_backend
JWT_SECRET_KEY=dev-secret-key-change-in-production
JWT_ALGORITHM=HS256
JWT_EXPIRE_HOURS=24
LOG_LEVEL=INFO
EOF
```

**配置说明**：
- `DATABASE_URL`: 数据库连接字符串
  - Docker 方式：`mysql+pymysql://asair:asair123@localhost:3306/asair_admin_backend`
  - 本地方式：`mysql+pymysql://asair:asair123@localhost:3306/asair_admin_backend`
- `JWT_SECRET_KEY`: JWT 密钥（生产环境请使用强随机字符串）

#### 初始化数据库

```bash
# 确保虚拟环境已激活
source venv/bin/activate

# 运行初始化脚本
python scripts/init_db.py
```

这将创建：
- 平台管理员账号：`admin / admin123`
- 示例租户：`demo_tenant_001`
- 示例租户管理员：`demo_admin / demo123`

---

### 4.2 配置 AI Server

#### 创建环境变量文件

在 `projects/asair-ai-server/` 目录创建 `.env` 文件：

```bash
cd projects/asair-ai-server
cat > .env << EOF
ASAIR_ENV=development
ASAIR_DEBUG=true
DATABASE_URL=mysql+pymysql://asair:asair123@localhost:3306/asair_ai_server
CHATWOOT_BASE_URL=http://localhost:3000
CHATWOOT_API_ACCESS_TOKEN=your-chatwoot-token-here
OPENAI_API_KEY=sk-your-openai-api-key-here
OPENAI_API_BASE=https://api.openai.com/v1
LOG_LEVEL=INFO
EOF
```

**配置说明**：
- `OPENAI_API_KEY`: **必需**，从 [OpenAI Platform](https://platform.openai.com/api-keys) 获取
- `CHATWOOT_BASE_URL`: Chatwoot 服务地址
  - Docker 方式：`http://localhost:3000`
  - 本地方式：`http://localhost:3000`（如果本地运行 Chatwoot）
- `CHATWOOT_API_ACCESS_TOKEN`: Chatwoot API Token（在 Chatwoot 后台获取，见下方）

#### 获取 OpenAI API Key

1. 访问 [OpenAI Platform](https://platform.openai.com/)
2. 注册/登录账号
3. 进入 **API Keys** 页面：https://platform.openai.com/api-keys
4. 点击 **Create new secret key**
5. 复制生成的 API Key（格式：`sk-...`）
6. 更新 `.env` 文件中的 `OPENAI_API_KEY`

#### 获取 Chatwoot API Token

1. 启动 Chatwoot 服务（Docker 或本地）
2. 访问 http://localhost:3000
3. 首次访问创建管理员账号
4. 登录后，进入 **Settings** → **API Tokens**
5. 点击 **Create Token**
6. 复制生成的 Token
7. 更新 `.env` 文件中的 `CHATWOOT_API_ACCESS_TOKEN`

#### 配置 Chatwoot Webhook

1. 在 Chatwoot 后台，进入 **Settings** → **Applications** → **Webhooks**
2. 添加新的 Webhook：
   - **URL**: `http://localhost:8000/webhook/chatwoot/message`（本地开发）
   - **URL**: `http://ai-server:8000/webhook/chatwoot/message`（Docker 环境）
   - **Events**: 选择 `message_created`
3. 保存配置

---

### 4.3 配置 Frontend

#### 配置 API 代理（本地开发）

编辑 `projects/asair-admin-frontend/vite.config.js`：

```javascript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    host: "0.0.0.0",
    proxy: {
      "/api": {
        target: "http://localhost:8100",
        changeOrigin: true
      }
    }
  }
});
```

**配置说明**：
- `target`: Admin Backend 服务地址（默认 `http://localhost:8100`）
- 如果前后端部署在同一域名下，可以移除 proxy 配置

---

## 五、验证启动

### 5.1 启动服务

#### Docker 方式启动

**手动启动**：
```bash
# 1. 启动三方服务（MySQL、Redis、Chatwoot）
docker compose up -d

# 2. 等待 MySQL 启动（约 15 秒）
sleep 15

# 3. 初始化数据库
docker compose exec admin-backend python scripts/init_db.py

# 4. 启动各个服务（需要多个终端）
```

#### 本地方式启动

需要打开多个终端窗口：

**终端 1：启动 Admin Backend**
```bash
cd projects/asair-admin-backend
source venv/bin/activate
python -m asair_admin_backend.app
```

服务将在 `http://localhost:8100` 启动

**终端 2：启动 AI Server**
```bash
cd projects/asair-ai-server
source venv/bin/activate
python -m asair_ai_server.app
```

服务将在 `http://localhost:8000` 启动

**终端 3：启动 Frontend**
```bash
cd projects/asair-admin-frontend
npm run dev
```

服务将在 `http://localhost:5173` 启动

**终端 4：启动 Chatwoot（如果本地安装）**
```bash
# 如果本地安装了 Chatwoot，启动它
# 否则使用 Docker：docker compose up chatwoot chatwoot-postgres
```

---

### 5.2 验证服务状态

#### 检查 Admin Backend

```bash
# 健康检查
curl http://localhost:8100/healthz

# 预期响应
# {"status": "ok"}
```

#### 检查 AI Server

```bash
# 健康检查
curl http://localhost:8000/healthz

# 预期响应
# {"status": "ok", "service": "asair-ai-server"}
```

#### 检查 Frontend

```bash
# 访问前端
curl http://localhost:5173

# 或在浏览器中打开
open http://localhost:5173
```

#### 检查 Chatwoot

```bash
# 访问 Chatwoot
curl http://localhost:3000

# 或在浏览器中打开
open http://localhost:3000
```

#### 检查数据库连接

**Docker 方式**：
```bash
# 进入 MySQL 容器
docker compose exec mysql mysql -u asair -pasair123 asair_admin_backend

# 查看表
SHOW TABLES;
EXIT;
```

**本地方式**：
```bash
# 连接 MySQL
mysql -u asair -pasair123 asair_admin_backend

# 查看表
SHOW TABLES;
EXIT;
```

---

### 5.3 验证完整流程

#### 1. 测试登录功能

1. 打开浏览器访问 http://localhost:5173
2. 使用平台管理员账号登录：
   - 用户名：`admin`
   - 密码：`admin123`
3. 验证是否能成功登录

#### 2. 测试 AI 服务

```bash
# 测试 AI Echo 接口
curl -X POST http://localhost:8000/internal/ai_echo \
  -H "Content-Type: application/json" \
  -d '{"message": "你好"}'

# 预期响应
# {"reply": "...", "error": false}
```

#### 3. 测试 Chatwoot 集成

1. 在 Chatwoot 后台创建 Account 和 Inbox
2. 配置 Telegram Bot（可选）
3. 发送测试消息
4. 验证 AI 是否能正常回复

---

## 六、常见问题排查

### 6.1 依赖安装问题

**问题：pip install 失败**
```bash
# 升级 pip
pip install --upgrade pip

# 使用国内镜像源
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

**问题：npm install 失败**
```bash
# 清除缓存
npm cache clean --force

# 使用国内镜像源
npm install --registry=https://registry.npmmirror.com
```

### 6.2 数据库连接问题

**问题：无法连接 MySQL**

Docker 方式：
```bash
# 检查 MySQL 容器状态
docker compose ps mysql

# 查看 MySQL 日志
docker compose logs mysql

# 重启 MySQL 容器
docker compose restart mysql
```

本地方式：
```bash
# 检查 MySQL 服务状态
brew services list | grep mysql

# 启动 MySQL 服务
brew services start mysql

# 检查端口占用
lsof -i :3306
```

### 6.3 端口占用问题

```bash
# 查看端口占用
lsof -i :8100  # Admin Backend
lsof -i :8000  # AI Server
lsof -i :5173  # Frontend
lsof -i :3000  # Chatwoot
lsof -i :3306  # MySQL

# 停止占用端口的进程
kill -9 <PID>
```

### 6.4 环境变量问题

**问题：环境变量未生效**

```bash
# 检查环境变量文件是否存在
ls -la projects/asair-admin-backend/.env
ls -la projects/asair-ai-server/.env

# 检查环境变量内容
cat projects/asair-admin-backend/.env
cat projects/asair-ai-server/.env

# 验证环境变量是否加载（Python）
cd projects/asair-admin-backend
source venv/bin/activate
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print(os.getenv('DATABASE_URL'))"
```

### 6.5 OpenAI API 问题

**问题：API 调用失败**
```bash
# 验证 API Key 是否正确
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"

# 检查环境变量
echo $OPENAI_API_KEY

# 检查 .env 文件中的配置
cat projects/asair-ai-server/.env | grep OPENAI
```

---

## 七、启动检查清单

使用以下清单确保所有步骤都已完成：

### ✅ 代码准备
- [ ] 已克隆所有项目仓库
- [ ] 项目目录结构正确

### ✅ 依赖安装
- [ ] Admin Backend 依赖已安装（`pip install -r requirements.txt`）
- [ ] AI Server 依赖已安装（`pip install -r requirements.txt`）
- [ ] Frontend 依赖已安装（`npm install`）

### ✅ 三方服务
- [ ] MySQL 已安装并运行（Docker 或本地）
- [ ] Redis 已安装并运行（可选，Docker 或本地）
- [ ] Chatwoot 已安装并运行（Docker 或本地）

### ✅ 配置完成
- [ ] Admin Backend `.env` 文件已配置
- [ ] AI Server `.env` 文件已配置
- [ ] Frontend `vite.config.js` 已配置（如需要）
- [ ] 数据库已初始化（`python scripts/init_db.py`）
- [ ] OpenAI API Key 已配置
- [ ] Chatwoot API Token 已配置
- [ ] Chatwoot Webhook 已配置

### ✅ 服务启动
- [ ] Admin Backend 服务已启动（端口 8100）
- [ ] AI Server 服务已启动（端口 8000）
- [ ] Frontend 服务已启动（端口 5173）
- [ ] Chatwoot 服务已启动（端口 3000）

### ✅ 验证通过
- [ ] Admin Backend 健康检查通过
- [ ] AI Server 健康检查通过
- [ ] Frontend 可以访问
- [ ] Chatwoot 可以访问
- [ ] 登录功能正常
- [ ] AI 服务测试通过

---

## 八、快速参考

### 8.1 服务端口

| 服务 | 端口 | 访问地址 |
|------|------|----------|
| Admin Backend | 8100 | http://localhost:8100 |
| AI Server | 8000 | http://localhost:8000 |
| Frontend | 5173 | http://localhost:5173 |
| Chatwoot | 3000 | http://localhost:3000 |
| MySQL | 3306 | localhost:3306 |
| Redis | 6379 | localhost:6379 |

### 8.2 默认账号

- **平台管理员**：`admin / admin123`
- **示例租户管理员**：`demo_admin / demo123`

### 8.3 常用命令

```bash
# Docker 相关
docker compose up -d          # 启动所有服务
docker compose down           # 停止所有服务
docker compose ps             # 查看服务状态
docker compose logs -f        # 查看日志

# 数据库初始化
cd projects/asair-admin-backend
source venv/bin/activate
python scripts/init_db.py

# 启动服务
# Admin Backend
cd projects/asair-admin-backend && source venv/bin/activate && python -m asair_admin_backend.app

# AI Server
cd projects/asair-ai-server && source venv/bin/activate && python -m asair_ai_server.app

# Frontend
cd projects/asair-admin-frontend && npm run dev
```

---

**最后更新**: 2025-12-10
