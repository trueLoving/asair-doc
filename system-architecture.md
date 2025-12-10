# ASAIR 多渠道 AI 客服平台 - 系统架构图

本文档使用 Mermaid 语法绘制系统架构图，展示四个核心模块与相关软件、第三方服务的依赖关系。

---

## 一、整体架构图

```mermaid
graph TB
    subgraph "用户端"
        TG[Telegram 用户]
        WA[WhatsApp 用户]
        LINE[LINE 用户]
        WEB[Web Widget 用户]
    end

    subgraph "第三方渠道服务"
        TG_API[Telegram Bot API]
        WA_API[WhatsApp Business API]
        LINE_API[LINE Messaging API]
    end

    subgraph "Chatwoot 客服系统"
        CW[Chatwoot Server<br/>:3000]
        CW_PG[(Chatwoot PostgreSQL)]
        CW_REDIS[(Redis Cache)]
    end

    subgraph "ASAIR 核心模块"
        subgraph "asair-admin-frontend"
            FE[React 前端<br/>:5173]
        end
        
        subgraph "asair-admin-backend"
            BE[Flask 管理后台<br/>:8100]
        end
        
        subgraph "asair-ai-server"
            AI[Flask AI 服务<br/>:8000]
        end
        
        subgraph "asair-infra"
            INFRA[部署配置<br/>Nginx/Systemd]
        end
    end

    subgraph "基础设施"
        MYSQL[(MySQL 8.0<br/>:3306)]
        REDIS_OPT[(Redis<br/>:6379<br/>可选)]
    end

    subgraph "第三方 AI 服务"
        OPENAI[OpenAI API<br/>GPT-4/GPT-3.5]
    end

    %% 用户到渠道
    TG -->|MTProto| TG_API
    WA --> WA_API
    LINE --> LINE_API
    WEB -->|HTTP| CW

    %% 渠道到 Chatwoot
    TG_API -->|Webhook| CW
    WA_API -->|Webhook| CW
    LINE_API -->|Webhook| CW

    %% Chatwoot 内部
    CW --> CW_PG
    CW --> CW_REDIS

    %% Chatwoot 到 AI Server
    CW -->|Webhook<br/>message_created| AI
    AI -->|REST API<br/>发送回复| CW

    %% AI Server 依赖
    AI -->|调用 API| OPENAI
    AI -->|读取配置| BE
    AI -->|读写数据| MYSQL

    %% Admin Backend 依赖
    BE -->|读写数据| MYSQL
    BE -->|可选缓存| REDIS_OPT

    %% Frontend 到 Backend
    FE -->|REST API<br/>JWT 鉴权| BE

    %% Infrastructure
    INFRA -.->|配置| FE
    INFRA -.->|配置| BE
    INFRA -.->|配置| AI
    INFRA -.->|配置| CW

    style FE fill:#e1f5ff
    style BE fill:#fff4e1
    style AI fill:#ffe1f5
    style INFRA fill:#f0f0f0
    style CW fill:#e8f5e9
    style MYSQL fill:#fff9c4
    style OPENAI fill:#f3e5f5
```

---

## 二、数据流架构图

```mermaid
sequenceDiagram
    participant User as 用户
    participant Channel as 渠道<br/>(Telegram/WhatsApp/LINE)
    participant Chatwoot as Chatwoot
    participant AIServer as asair-ai-server
    participant AdminBE as asair-admin-backend
    participant OpenAI as OpenAI API
    participant MySQL as MySQL

    Note over User,MySQL: 用户消息处理流程

    User->>Channel: 发送消息
    Channel->>Chatwoot: Webhook 推送消息
    Chatwoot->>Chatwoot: 创建 Conversation/Message
    Chatwoot->>AIServer: POST /webhook/chatwoot/message<br/>(message_created 事件)
    
    AIServer->>AdminBE: GET /api/prompts<br/>(获取 Prompt 配置)
    AdminBE->>MySQL: 查询 Prompt 模板
    MySQL-->>AdminBE: 返回 Prompt
    AdminBE-->>AIServer: 返回 Prompt 配置
    
    AIServer->>AIServer: 多阶段 AI 流程<br/>(敌意检测/记忆检索/RAG)
    AIServer->>OpenAI: POST /v1/chat/completions<br/>(生成 AI 回复)
    OpenAI-->>AIServer: 返回 AI 回复
    
    AIServer->>Chatwoot: POST /api/v1/accounts/:id/conversations/:id/messages<br/>(发送回复)
    Chatwoot->>Channel: 推送消息到渠道
    Channel->>User: 显示 AI 回复

    Note over User,MySQL: 管理后台操作流程

    User->>AdminBE: 登录 (POST /api/platform/login)
    AdminBE->>MySQL: 验证用户
    MySQL-->>AdminBE: 返回用户信息
    AdminBE-->>User: 返回 JWT Token
    
    User->>AdminBE: 配置 Prompt (POST /api/prompts)
    AdminBE->>MySQL: 保存 Prompt 配置
    MySQL-->>AdminBE: 确认保存
    AdminBE-->>User: 返回成功
```

---

## 三、模块依赖关系图

```mermaid
graph LR
    subgraph "asair-admin-frontend"
        FE[React 18 + Vite]
        FE_ROUTER[React Router 6]
        FE_AXIOS[Axios]
    end

    subgraph "asair-admin-backend"
        BE_FLASK[Flask 3.0]
        BE_SQLALCHEMY[SQLAlchemy 2.0]
        BE_JWT[PyJWT]
        BE_BCRYPT[bcrypt]
        BE_PYMYSQL[PyMySQL]
    end

    subgraph "asair-ai-server"
        AI_FLASK[Flask 3.0]
        AI_OPENAI[openai 1.0+]
        AI_HTTPX[httpx]
        AI_PYDANTIC[Pydantic 2.0]
        AI_PYMYSQL2[PyMySQL]
    end

    subgraph "asair-infra"
        INFRA_DOCKER[Docker Compose]
        INFRA_NGINX[Nginx]
        INFRA_SYSTEMD[systemd]
    end

    subgraph "基础设施服务"
        MYSQL[(MySQL 8.0)]
        REDIS[(Redis 7)]
    end

    subgraph "第三方服务"
        CHATWOOT[Chatwoot]
        OPENAI[OpenAI API]
        TELEGRAM[Telegram Bot API]
    end

    %% Frontend 依赖
    FE --> FE_ROUTER
    FE --> FE_AXIOS
    FE_AXIOS -->|HTTP| BE_FLASK

    %% Backend 依赖
    BE_FLASK --> BE_SQLALCHEMY
    BE_FLASK --> BE_JWT
    BE_FLASK --> BE_BCRYPT
    BE_SQLALCHEMY --> BE_PYMYSQL
    BE_PYMYSQL --> MYSQL

    %% AI Server 依赖
    AI_FLASK --> AI_OPENAI
    AI_FLASK --> AI_HTTPX
    AI_FLASK --> AI_PYDANTIC
    AI_OPENAI -->|HTTP| OPENAI
    AI_HTTPX -->|HTTP| CHATWOOT
    AI_FLASK --> AI_PYMYSQL2
    AI_PYMYSQL2 --> MYSQL

    %% Infrastructure 管理
    INFRA_DOCKER -.->|编排| FE
    INFRA_DOCKER -.->|编排| BE_FLASK
    INFRA_DOCKER -.->|编排| AI_FLASK
    INFRA_DOCKER -.->|编排| CHATWOOT
    INFRA_DOCKER -.->|编排| MYSQL
    INFRA_DOCKER -.->|编排| REDIS
    INFRA_NGINX -.->|反向代理| FE
    INFRA_NGINX -.->|反向代理| BE_FLASK
    INFRA_NGINX -.->|反向代理| AI_FLASK
    INFRA_NGINX -.->|反向代理| CHATWOOT

    %% 第三方服务连接
    CHATWOOT -->|Webhook| AI_FLASK
    TELEGRAM -->|Webhook| CHATWOOT

    style FE fill:#e1f5ff
    style BE_FLASK fill:#fff4e1
    style AI_FLASK fill:#ffe1f5
    style INFRA_DOCKER fill:#f0f0f0
    style MYSQL fill:#fff9c4
    style CHATWOOT fill:#e8f5e9
    style OPENAI fill:#f3e5f5
```

---

## 四、部署架构图

```mermaid
graph TB
    subgraph "Docker Compose 编排"
        subgraph "网络: asair-network"
            subgraph "前端服务"
                FE_CONTAINER[asair-admin-frontend<br/>Container<br/>:5173]
            end
            
            subgraph "后端服务"
                BE_CONTAINER[asair-admin-backend<br/>Container<br/>:8100]
                AI_CONTAINER[asair-ai-server<br/>Container<br/>:8000]
            end
            
            subgraph "第三方服务"
                CW_CONTAINER[asair-chatwoot<br/>Container<br/>:3000]
                CW_PG_CONTAINER[asair-chatwoot-postgres<br/>Container]
            end
            
            subgraph "基础设施"
                MYSQL_CONTAINER[asair-mysql<br/>Container<br/>:3306]
                REDIS_CONTAINER[asair-redis<br/>Container<br/>:6379]
            end
        end
    end

    subgraph "外部服务"
        OPENAI_EXT[OpenAI API<br/>https://api.openai.com]
        TG_EXT[Telegram Bot API<br/>https://api.telegram.org]
    end

    subgraph "数据卷"
        MYSQL_VOL[mysql_data<br/>Volume]
        REDIS_VOL[redis_data<br/>Volume]
        CW_VOL[chatwoot_data<br/>Volume]
        CW_PG_VOL[chatwoot_postgres_data<br/>Volume]
    end

    subgraph "宿主机端口映射"
        PORT_5173[localhost:5173]
        PORT_8100[localhost:8100]
        PORT_8000[localhost:8000]
        PORT_3000[localhost:3000]
        PORT_3306[localhost:3306]
        PORT_6379[localhost:6379]
    end

    %% 容器连接
    FE_CONTAINER -->|HTTP| BE_CONTAINER
    BE_CONTAINER -->|SQL| MYSQL_CONTAINER
    AI_CONTAINER -->|SQL| MYSQL_CONTAINER
    AI_CONTAINER -->|HTTP| CW_CONTAINER
    CW_CONTAINER -->|SQL| CW_PG_CONTAINER
    CW_CONTAINER -->|Cache| REDIS_CONTAINER
    
    %% 外部连接
    AI_CONTAINER -->|HTTPS| OPENAI_EXT
    CW_CONTAINER -->|HTTPS| TG_EXT
    
    %% 数据持久化
    MYSQL_CONTAINER --> MYSQL_VOL
    REDIS_CONTAINER --> REDIS_VOL
    CW_CONTAINER --> CW_VOL
    CW_PG_CONTAINER --> CW_PG_VOL
    
    %% 端口映射
    FE_CONTAINER -.->|映射| PORT_5173
    BE_CONTAINER -.->|映射| PORT_8100
    AI_CONTAINER -.->|映射| PORT_8000
    CW_CONTAINER -.->|映射| PORT_3000
    MYSQL_CONTAINER -.->|映射| PORT_3306
    REDIS_CONTAINER -.->|映射| PORT_6379

    style FE_CONTAINER fill:#e1f5ff
    style BE_CONTAINER fill:#fff4e1
    style AI_CONTAINER fill:#ffe1f5
    style CW_CONTAINER fill:#e8f5e9
    style MYSQL_CONTAINER fill:#fff9c4
    style REDIS_CONTAINER fill:#ffebee
```

---

## 五、API 调用关系图

```mermaid
graph TB
    subgraph "asair-admin-frontend"
        FE_LOGIN[登录页面]
        FE_DASHBOARD[仪表盘]
        FE_TENANT[商户管理]
        FE_AI_CONFIG[AI 配置]
        FE_CDP[客户数据平台]
    end

    subgraph "asair-admin-backend API"
        BE_AUTH[POST /api/platform/login<br/>POST /api/tenant/login]
        BE_TENANT[GET /api/platform/tenants<br/>POST /api/platform/tenants]
        BE_PROMPT[GET /api/prompts<br/>POST /api/prompts]
        BE_REPORT[GET /api/reports/llm-usage]
        BE_CHANNEL[GET /api/channels]
    end

    subgraph "asair-ai-server API"
        AI_WEBHOOK[POST /webhook/chatwoot/message]
        AI_ECHO[POST /internal/ai_echo]
        AI_HEALTH[GET /healthz]
    end

    subgraph "Chatwoot API"
        CW_API["POST /api/v1/accounts/:id/conversations/:id/messages"]
        CW_WEBHOOK[Webhook: message_created]
    end

    subgraph "OpenAI API"
        OPENAI_API[POST /v1/chat/completions]
    end

    %% Frontend 调用 Backend
    FE_LOGIN --> BE_AUTH
    FE_DASHBOARD --> BE_REPORT
    FE_TENANT --> BE_TENANT
    FE_AI_CONFIG --> BE_PROMPT
    FE_CDP --> BE_CHANNEL

    %% Chatwoot 调用 AI Server
    CW_WEBHOOK --> AI_WEBHOOK

    %% AI Server 调用其他服务
    AI_WEBHOOK --> BE_PROMPT
    AI_WEBHOOK --> OPENAI_API
    AI_WEBHOOK --> CW_API

    %% 内部调试
    AI_ECHO --> OPENAI_API

    style FE_LOGIN fill:#e1f5ff
    style BE_AUTH fill:#fff4e1
    style AI_WEBHOOK fill:#ffe1f5
    style CW_WEBHOOK fill:#e8f5e9
    style OPENAI_API fill:#f3e5f5
```

---

## 六、数据库关系图

```mermaid
erDiagram
    TENANT ||--o{ TENANT_ADMIN_USER : "拥有"
    TENANT ||--o{ SUPPORT_PROVIDER_ACCOUNT : "配置"
    TENANT ||--o{ AI_ASSISTANT : "配置"
    TENANT ||--o{ PROMPT : "配置"
    
    TENANT {
        int id PK
        string name
        string logo_url
        string language
        string timezone
        datetime created_at
    }
    
    TENANT_ADMIN_USER {
        int id PK
        int tenant_id FK
        string username
        string password_hash
        string email
        datetime created_at
    }
    
    SUPPORT_PROVIDER_ACCOUNT {
        int id PK
        int tenant_id FK
        string provider
        string base_url
        string external_account_id
        string api_access_token
        json config_json
        boolean is_active
    }
    
    AI_ASSISTANT {
        int id PK
        int tenant_id FK
        string name
        string avatar_url
        text character_description
        json prompt_config
    }
    
    PROMPT {
        int id PK
        int tenant_id FK
        int assistant_id FK
        string scene
        string language
        text content
        string status
        int version
        datetime created_at
    }
    
    LLM_USAGE_LOG {
        int id PK
        int tenant_id FK
        string user_id
        string channel
        string model
        string stage
        int prompt_tokens
        int completion_tokens
        int total_tokens
        float cost_usd
        int latency_ms
        boolean success
        datetime created_at
    }
```

---

## 七、技术栈依赖图

```mermaid
graph TD
    subgraph "前端技术栈"
        REACT[React 18.3]
        VITE[Vite 5.0]
        ROUTER[React Router 6.28]
        AXIOS[Axios 1.13]
    end

    subgraph "后端技术栈 - Admin Backend"
        FLASK_BE[Flask 3.0]
        SQLALCHEMY[SQLAlchemy 2.0]
        JWT_BE[PyJWT 2.8]
        BCRYPT[bcrypt 4.0]
        PYMYSQL_BE[PyMySQL 1.1]
    end

    subgraph "后端技术栈 - AI Server"
        FLASK_AI[Flask 3.0]
        OPENAI_SDK[openai 1.0+]
        HTTPX[httpx 0.27]
        PYDANTIC[Pydantic 2.0]
        PYMYSQL_AI[PyMySQL 1.1]
    end

    subgraph "基础设施技术栈"
        DOCKER[Docker]
        COMPOSE[Docker Compose]
        NGINX[Nginx]
        SYSTEMD[systemd]
    end

    subgraph "运行时环境"
        NODE[Node.js 18+]
        PYTHON[Python 3.10+]
    end

    subgraph "数据库"
        MYSQL_DB[MySQL 8.0]
        POSTGRES[PostgreSQL 15]
        REDIS_DB[Redis 7]
    end

    %% 前端依赖
    REACT --> NODE
    VITE --> NODE
    ROUTER --> REACT
    AXIOS --> REACT

    %% Backend 依赖
    FLASK_BE --> PYTHON
    SQLALCHEMY --> PYTHON
    JWT_BE --> PYTHON
    BCRYPT --> PYTHON
    PYMYSQL_BE --> PYTHON
    SQLALCHEMY --> PYMYSQL_BE
    PYMYSQL_BE --> MYSQL_DB

    %% AI Server 依赖
    FLASK_AI --> PYTHON
    OPENAI_SDK --> PYTHON
    HTTPX --> PYTHON
    PYDANTIC --> PYTHON
    PYMYSQL_AI --> PYTHON
    PYMYSQL_AI --> MYSQL_DB

    %% 基础设施
    COMPOSE --> DOCKER
    NGINX --> DOCKER
    SYSTEMD --> DOCKER

    style REACT fill:#61dafb
    style FLASK_BE fill:#000000,color:#fff
    style FLASK_AI fill:#000000,color:#fff
    style DOCKER fill:#2496ed,color:#fff
    style MYSQL_DB fill:#4479a1,color:#fff
```

---

## 八、服务启动顺序图

```mermaid
graph TD
    START[开始启动] --> DOCKER_CHECK{检查 Docker}
    DOCKER_CHECK -->|未运行| ERROR1[错误: 启动 Docker Desktop]
    DOCKER_CHECK -->|已运行| START_INFRA[启动基础设施]
    
    START_INFRA --> MYSQL_START[启动 MySQL]
    MYSQL_START --> MYSQL_HEALTH{MySQL 健康检查}
    MYSQL_HEALTH -->|失败| ERROR2[错误: MySQL 启动失败]
    MYSQL_HEALTH -->|成功| REDIS_START[启动 Redis]
    
    REDIS_START --> PG_START[启动 Chatwoot PostgreSQL]
    PG_START --> PG_HEALTH{Chatwoot PG 健康检查}
    PG_HEALTH -->|失败| ERROR3[错误: PostgreSQL 启动失败]
    PG_HEALTH -->|成功| CW_START[启动 Chatwoot]
    
    CW_START --> CW_INIT[初始化 Chatwoot 数据库]
    CW_INIT --> CW_READY{Chatwoot 就绪?}
    CW_READY -->|否| WAIT[等待 30 秒]
    WAIT --> CW_READY
    CW_READY -->|是| BE_START[启动 Admin Backend]
    
    BE_START --> BE_INIT[初始化 Backend 数据库]
    BE_INIT --> BE_HEALTH{Backend 健康检查}
    BE_HEALTH -->|失败| ERROR4[错误: Backend 启动失败]
    BE_HEALTH -->|成功| AI_START[启动 AI Server]
    
    AI_START --> AI_HEALTH{AI Server 健康检查}
    AI_HEALTH -->|失败| ERROR5[错误: AI Server 启动失败]
    AI_HEALTH -->|成功| FE_START[启动 Frontend]
    
    FE_START --> FE_READY{Frontend 就绪?}
    FE_READY -->|失败| ERROR6[错误: Frontend 启动失败]
    FE_READY -->|成功| SUCCESS[✅ 所有服务启动成功]
    
    style START fill:#e3f2fd
    style SUCCESS fill:#c8e6c9
    style ERROR1 fill:#ffcdd2
    style ERROR2 fill:#ffcdd2
    style ERROR3 fill:#ffcdd2
    style ERROR4 fill:#ffcdd2
    style ERROR5 fill:#ffcdd2
    style ERROR6 fill:#ffcdd2
```

---

## 九、消息流转完整路径

```mermaid
flowchart LR
    A[用户发送消息] --> B[Telegram/WhatsApp/LINE]
    B --> C[渠道 Webhook]
    C --> D[Chatwoot 接收]
    D --> E[Chatwoot 创建 Message]
    E --> F[Chatwoot 触发 Webhook]
    F --> G[asair-ai-server 接收]
    G --> H[解析 Webhook Payload]
    H --> I[从 asair-admin-backend<br/>获取 Prompt 配置]
    I --> J[执行多阶段 AI 流程]
    J --> K[调用 OpenAI API]
    K --> L[生成 AI 回复]
    L --> M[调用 Chatwoot API<br/>发送回复]
    M --> N[Chatwoot 推送消息]
    N --> O[渠道推送]
    O --> P[用户收到回复]
    
    style A fill:#e1f5ff
    style G fill:#ffe1f5
    style K fill:#f3e5f5
    style P fill:#c8e6c9
```

---

## 十、环境变量配置关系

```mermaid
graph TB
    subgraph ".env 文件"
        ENV_FILE[.env]
    end

    subgraph "asair-admin-backend 环境变量"
        BE_ENV1[DATABASE_URL]
        BE_ENV2[JWT_SECRET_KEY]
        BE_ENV3[JWT_ALGORITHM]
        BE_ENV4[ASAIR_ADMIN_ENV]
    end

    subgraph "asair-ai-server 环境变量"
        AI_ENV1[DATABASE_URL]
        AI_ENV2[CHATWOOT_BASE_URL]
        AI_ENV3[CHATWOOT_API_ACCESS_TOKEN]
        AI_ENV4[OPENAI_API_KEY]
        AI_ENV5[OPENAI_API_BASE]
    end

    subgraph "Chatwoot 环境变量"
        CW_ENV1[POSTGRES_HOST]
        CW_ENV2[POSTGRES_PASSWORD]
        CW_ENV3[REDIS_URL]
        CW_ENV4[SECRET_KEY_BASE]
    end

    ENV_FILE -->|docker-compose.yml 读取| BE_ENV1
    ENV_FILE -->|docker-compose.yml 读取| BE_ENV2
    ENV_FILE -->|docker-compose.yml 读取| BE_ENV3
    ENV_FILE -->|docker-compose.yml 读取| BE_ENV4
    
    ENV_FILE -->|docker-compose.yml 读取| AI_ENV1
    ENV_FILE -->|docker-compose.yml 读取| AI_ENV2
    ENV_FILE -->|docker-compose.yml 读取| AI_ENV3
    ENV_FILE -->|docker-compose.yml 读取| AI_ENV4
    ENV_FILE -->|docker-compose.yml 读取| AI_ENV5
    
    ENV_FILE -->|docker-compose.yml 读取| CW_ENV1
    ENV_FILE -->|docker-compose.yml 读取| CW_ENV2
    ENV_FILE -->|docker-compose.yml 读取| CW_ENV3
    ENV_FILE -->|docker-compose.yml 读取| CW_ENV4

    style ENV_FILE fill:#fff9c4
    style BE_ENV1 fill:#fff4e1
    style AI_ENV4 fill:#ffe1f5
    style CW_ENV4 fill:#e8f5e9
```

---

## 使用说明

以上架构图使用 Mermaid 语法编写，可以在以下平台查看：

1. **GitHub/GitLab**: 直接支持 Mermaid 渲染
2. **VS Code**: 安装 "Markdown Preview Mermaid Support" 插件
3. **在线编辑器**: https://mermaid.live/
4. **Notion/Obsidian**: 支持 Mermaid 语法

## 图例说明

- **蓝色**: 前端相关
- **黄色**: 后端管理服务
- **粉色**: AI 服务
- **绿色**: 第三方服务（Chatwoot）
- **紫色**: 外部 API（OpenAI）
- **灰色**: 基础设施配置

---

**最后更新**: 2025-01-XX

