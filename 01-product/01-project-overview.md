# ASAIR 多渠道 AI 客服平台 - 项目现状梳理报告

**生成时间：** 2025-12-11  
**项目类型：** AI 客服平台（多渠道、多租户、多语言）

---

## 一、项目总体概况

### 1.1 项目定位

ASAIR 是一个面向**东亚/东南亚市场**的多渠道 AI 客服平台，主要服务：
- **目标地区**：日本、韩国、越南、中国台湾等
- **主要渠道**：Telegram、WhatsApp、LINE、Web/H5、小程序
- **核心能力**：AI 智能对话、好感度系统、记忆管理、人工协同

### 1.2 技术路线决策（关键）

项目目前处于**技术路线选择的关键节点**，需要在两条路线中做出决策：

#### 路线 A：继续自研体系（现有路线）
- **现状**：已有完整自研系统（1233前端+后端 + ASAIR_Backend + 自研通讯层）
- **特点**：完全可控，但维护成本高
- **适用场景**：希望最大程度复用现有系统，短期内不想动架构

#### 路线 B：采用第三方客服系统（Chatwoot）
- **现状**：正在技术验证阶段（已完成 Chatwoot + Telegram 集成）
- **特点**：降低维护成本，专注 AI 能力，但需要迁移
- **适用场景**：希望把精力放在 AI 能力上，目标服务更多 B 端客户

**当前状态**：路线 B 已完成 PoC，正在推进中

---

## 二、项目工程结构

项目由 **4 个核心工程**组成：

### 2.1 asair-admin-backend（管理后台后端）

**技术栈：** Python + Flask + MySQL  
**职责：** 运营/管理后台的 API 服务

**核心功能：**
- ✅ 多租户管理（tenant、tenant_admin_user）
- ✅ 商户管理（merchant）
- ✅ 渠道配置（support_provider_account，支持 Chatwoot/Tiledesk/Zammad）
- ✅ Prompt 模板管理（prompt）
- ✅ JWT 鉴权系统（平台管理员 + 租户管理员）
- ✅ LLM 调用日志（llm_usage_log）

**当前状态：**
- 基础骨架已完成
- 数据库模型已建立
- API 接口部分实现（部分为 mock 数据）

**关键文件：**
- `src/asair_admin_backend/models/` - 数据模型
- `src/asair_admin_backend/routers/` - API 路由
- `scripts/init_db.py` - 数据库初始化脚本

---

### 2.2 asair-admin-frontend（管理后台前端）

**技术栈：** React 18 + Vite + React Router 6  
**职责：** 运营/管理后台的前端界面

**核心功能：**

#### 平台管理端（/platform）
- ✅ 商户列表管理
- ✅ 新建商户
- ✅ 模型网关配置

#### 商户管理端（/tenant）
- ✅ Dashboard（数据概览）
- ✅ CDP（客户数据平台）- 客户列表、客户详情
- ✅ AI 配置模块（4个子页面）：
  - Prompt 工程（系统提示词、角色设定、回复规则）
  - 知识库管理（知识条目 CRUD、检索配置）
  - 自动标签（标签规则配置、触发词管理）
  - 转人工规则（触发条件、阈值设置）
- ✅ 开发者与集成（API 凭证、Webhook 配置、连接状态监控）
- ✅ 系统设置（账户信息、团队成员、通知设置、安全设置、语言时区、数据管理）

**当前状态：**
- 已完成从 Vue 到 React 的迁移（2024-11-19）
- 已完成商户管理端 7 个核心功能页面（2024-12-07）
- 认证系统已实现（JWT + localStorage）
- 路由守卫已实现（AuthGuard）

**技术亮点：**
- 使用共享 CSS 架构（shared.css）减少代码重复
- iOS 风格 Toggle Switch 组件
- 终端风格的 Webhook 日志展示

---

### 2.3 asair-ai-server（AI 大脑服务）

**技术栈：** Python + Flask + OpenAI API  
**职责：** AI 对话处理的核心服务

**核心功能：**
- ✅ Chatwoot Webhook 接收（`/webhook/chatwoot/message`）
- ✅ 多阶段 AI 流程引擎（flow_engine）
- ✅ LLM 客户端封装（llm_client，支持 OpenAI）
- ✅ Prompt 服务（prompt_service）
- ✅ 记忆服务（memory_service）
- ✅ 敌意检测（hostility_service）
- ✅ 节奏管理器（rhythm_manager）
- ✅ Chatwoot API 客户端（chatwoot_client）
- ✅ 内部调试接口（`/internal/ai_echo`）

**当前状态：**
- ✅ 已完成 Chatwoot Webhook 到固定回复的闭环（2025-11-19）
- ✅ 已完成 Chatwoot + Telegram 集成（2025-12-01 第 3 周）
- ✅ 消息转发逻辑已实现（Telegram ↔ Chatwoot）
- ✅ 消息映射与存储（conversation_id ↔ tg_chat_id）

**技术细节：**
- 使用 Pydantic v2 进行数据验证
- 使用 httpx 作为 HTTP 客户端
- 完善的错误处理和结构化日志

---

### 2.4 asair-infra（基础设施与部署）

**技术栈：** Shell 脚本 + Nginx + systemd  
**职责：** 部署、运维配置管理

**核心内容：**
- ✅ Nginx 反向代理配置（asair-admin、asair-ai-server、chatwoot）
- ✅ systemd 服务配置（asair-admin-backend、asair-ai-server）
- ✅ 部署脚本（setup-dev-server、deploy-*）
- ✅ 架构文档（architecture.md、env-dev.md、env-prod.md）

**当前状态：**
- 采用传统部署方式（venv + gunicorn + Nginx）
- 支持 Docker 化演进路径
- 配置模板已就绪

---

## 三、功能模块现状

### 3.1 已实现功能（P0 优先级）

| 模块 | 功能 | 状态 | 说明 |
|------|------|------|------|
| **M1 多租户管理** | 商户账号体系 | ✅ | 平台创建/管理商户 |
| | 商户基本信息 | ✅ | 名称、Logo、语言、时区 |
| | 商户管理员登录 | ✅ | JWT 鉴权 |
| **M2 多渠道接入** | Chatwoot 集成 | ✅ | Telegram 已打通 |
| | 会话路由 | ✅ | 用户消息 → AI → 用户 |
| **M3 AI 人设管理** | Prompt 配置 | ✅ | 系统提示词、角色设定 |
| | 版本管理 | ⚠️ | 基础版本管理（草稿/发布） |
| **M4 好感度系统** | 用户档案 | ⚠️ | 基础信息（部分实现） |
| | 好感度计算 | ❌ | 待实现 |
| **M5 会话节奏控制** | 多条输入合并 | ⚠️ | 节奏管理器占位实现 |
| **M6 客服工作台** | 会话列表 | ✅ | Chatwoot 原生支持 |
| | 人工接管 | ✅ | Chatwoot 原生支持 |
| **M7 商户自助配置** | AI 助手配置 | ✅ | 前端界面已完成 |
| | 欢迎语配置 | ✅ | 前端界面已完成 |
| **M8 报表分析** | 基础运营数据 | ⚠️ | 部分 mock 数据 |
| | AI 调用统计 | ⚠️ | 数据模型已建立，待完善 |

### 3.2 待实现功能（P1/P2 优先级）

- **M2**：第二/第三个渠道接入（LINE、WhatsApp）
- **M3**：Prompt 可视化编辑、历史版本回滚
- **M4**：好感度规则、记忆功能、好感度趋势图
- **M5**：可配置停顿时间、最大合并条数
- **M6**：会话标签、快捷回复、多会话处理
- **M8**：分模型成本统计、人工接管率、转化指标

---

## 四、技术架构现状

### 4.1 当前架构（路线 B - Chatwoot 路线）

```
用户端（Telegram/Web/LINE）
    ↓
Chatwoot（多渠道客服系统）
    ↓ Webhook
asair-ai-server（AI 大脑）
    ↓ API
asair-admin-backend（配置中心）
    ↓
MySQL 数据库
```

### 4.2 数据流

1. **用户消息流程：**
   - 用户 → Chatwoot → Webhook → asair-ai-server → LLM → Chatwoot → 用户

2. **配置管理流程：**
   - 运营人员 → asair-admin-frontend → asair-admin-backend → MySQL
   - asair-ai-server 从 asair-admin-backend 拉取配置

### 4.3 技术栈汇总

| 工程 | 语言 | 框架 | 数据库 | 部署方式 |
|------|------|------|--------|----------|
| asair-admin-backend | Python | Flask | MySQL | systemd/gunicorn |
| asair-admin-frontend | JavaScript | React 18 | - | Nginx 静态文件 |
| asair-ai-server | Python | Flask | - | systemd/gunicorn |
| asair-infra | Shell | - | - | 传统部署 |

---

## 五、项目进展与里程碑

### 5.1 已完成里程碑

- ✅ **2024-11-19**：完成前端 Vue → React 迁移
- ✅ **2024-11-19**：完成租户/平台数据库模型与 JWT 鉴权
- ✅ **2024-11-19**：完成 Chatwoot Webhook 到固定回复闭环
- ✅ **2024-11-25**：新增 AI Echo 内部调试接口
- ✅ **2024-12-07**：完成商户管理端 7 个核心功能页面
- ✅ **2024-12-01（第 3 周）**：完成 Chatwoot + Telegram 集成

### 5.2 当前阶段

**阶段：** Phase 1 - AI 整合与管理后台改造（进行中）

**目标：**
- 实现正式版 Webhook handler + message_trace + 幂等重试
- 落地 llm_usage_log + 成本/性能 Dashboard
- 完善 Prompt 管理（按商户/inbox 维度配置）

**预计完成时间：** 6-8 周（从 2024-12-01 起）

### 5.3 下一步计划

1. **完善 AI Server 功能**
   - 实现多阶段 LLM Pipeline（敌意检测、记忆检索、RAG）
   - 完善节奏控制管理器
   - 从 admin-backend 动态拉取配置

2. **完善管理后台**
   - 实现真实数据接口（替换 mock 数据）
   - 完善报表功能（AI 成本统计、会话分析）
   - 完善 Prompt 版本管理

3. **渠道扩展**
   - 接入 LINE
   - 接入 WhatsApp
   - Web widget 集成

---

## 六、关键问题与风险

### 6.1 技术债务

1. **测试覆盖率低**
   - 当前：接近 0%
   - 风险：重构和功能扩展风险高
   - 建议：优先为核心模块添加单元测试

2. **依赖版本过旧**
   - OpenAI SDK：0.28.0 → 1.x（API 已变更）
   - 部分依赖未锁定版本
   - 建议：逐步升级并锁定版本

3. **缺少监控告警**
   - 无错误监控（Sentry）
   - 无性能监控（Prometheus）
   - 建议：接入基础监控体系

### 6.2 架构风险

1. **双轨过渡期**
   - 旧系统（1233 + 自研通讯层）与新系统（Chatwoot + ASAIR）并行
   - 风险：数据不一致、维护成本高
   - 建议：明确迁移时间表，尽快完成切换

2. **配置管理分散**
   - Chatwoot 配置在 Chatwoot 后台
   - AI 配置在 asair-admin-backend
   - 风险：配置同步问题
   - 建议：建立配置同步机制

### 6.3 业务风险

1. **多语言支持不完整**
   - 目标：支持日文、韩文、越南文、繁中
   - 现状：界面多语言支持待完善
   - 建议：优先完善核心语言支持

2. **成本监控不完善**
   - LLM 调用成本统计不完整
   - 无法及时发现异常消耗
   - 建议：完善成本 Dashboard

---

## 七、文档与规范

### 7.1 现有文档

- ✅ **需求文档**：`整理的需求文档.md`、`功能模块 + NFR + 优先级.md`
- ✅ **技术方案**：`第三方客服系统技术方案与迁移计划.md`、`Chatwoot 实现途径.md`
- ✅ **问题分析**：`业务问题回复与修复计划.md`、`AI_System_Issuess.md`
- ✅ **项目报告**：综合分析报告、工程分析报告、周报（第 1-3 周）
- ✅ **工程 README**：各工程均有 README 说明

### 7.2 文档质量

- **优点**：文档结构清晰，需求分析详细
- **不足**：API 文档不完整，缺少 Swagger 文档
- **建议**：补充 API 文档，建立文档更新机制

---

## 八、团队与资源

### 8.1 当前团队配置（推测）

- **后端工程师**：1-2 人（Python/Flask）
- **前端工程师**：1 人（React）
- **DevOps**：1 人（兼职，部署与监控）

### 8.2 资源需求

根据文档分析，完成 Phase 1 需要：
- **人力**：2-3 人核心团队
- **时间**：6-8 周
- **基础设施**：服务器、数据库、Chatwoot 部署环境

---

## 九、总结与建议

### 9.1 项目优势

1. ✅ **架构清晰**：前后端分离，模块化设计良好
2. ✅ **技术选型合理**：React 18、Flask 3.0、Chatwoot 成熟稳定
3. ✅ **需求明确**：文档详细，优先级清晰
4. ✅ **进展顺利**：核心功能已打通，基础框架已建立

### 9.2 关键挑战

1. ⚠️ **技术债务**：测试覆盖率低、依赖版本过旧
2. ⚠️ **监控缺失**：缺少完善的监控告警体系
3. ⚠️ **迁移风险**：双轨并行期的数据一致性风险

### 9.3 优先建议

**短期（1-2 周）：**
1. 🔴 完善 AI Server 的多阶段 LLM Pipeline
2. 🔴 实现真实的成本统计 Dashboard
3. 🟡 为核心模块添加单元测试（覆盖率 20%+）

**中期（1-3 月）：**
1. 🟡 接入第二个渠道（LINE 或 WhatsApp）
2. 🟡 完善 Prompt 版本管理功能
3. 🟡 接入基础监控体系（Sentry + Prometheus）

**长期（3-6 月）：**
1. 🟢 完成所有目标渠道接入
2. 🟢 完善多语言支持
3. 🟢 建立完整的测试体系（覆盖率 60%+）
