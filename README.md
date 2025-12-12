# ASAIR 项目文档中心

<div align="center">

![Status](https://img.shields.io/badge/status-active-success.svg)
![Docs](https://img.shields.io/badge/docs-latest-blue.svg)

**ASAIR 多渠道 AI 客服平台项目文档**

[快速开始](#-快速开始) | [文档目录](#-文档目录) | [相关链接](#-相关链接)

</div>

---

## 📖 项目简介

本仓库包含 **ASAIR 多渠道 AI 客服平台**的所有项目文档，包括项目现状分析、系统架构设计、业务场景说明等核心文档。

> **⚠️ 重要说明**：由于时间关系和资料缺少的关系，目前只是梳理出这么多，不一定完全正确。如果后续在开发过程中发现不对会立即纠正。

### 🎯 文档分类

文档按照功能模块进行分类组织，便于查找和维护：

- **产品文档** (`01-product/`) - 项目现状、业务场景等产品相关文档
- **设计文档** (`02-design/`) - 系统架构、技术设计等设计相关文档

---

## 📚 文档目录

### 产品文档 (`01-product/`)

#### 1. [项目现状梳理报告](./01-product/01-project-overview.md)

全面的项目现状分析报告，包含：

- ✅ 项目总体概况和定位
- ✅ 四个核心工程的详细分析
- ✅ 功能模块现状梳理
- ✅ 技术架构说明
- ✅ 项目进展与里程碑
- ✅ 关键问题与风险识别
- ✅ 技术路线决策分析

#### 2. [业务场景图](./01-product/02-business-scenarios.md)

使用 Mermaid 语法绘制的业务场景流程图，包含：

- ✅ 用户与 AI 客服对话完整流程
- ✅ 商户管理员配置 AI 助手流程
- ✅ 平台管理员管理商户流程
- ✅ 人工客服接管流程
- ✅ 多渠道接入场景
- ✅ 好感度系统工作流程
- ✅ 转人工触发场景
- ✅ 会话节奏控制场景
- ✅ 商户运营工作流
- ✅ 用户旅程图
- ✅ 知识库检索场景
- ✅ 报表查看与分析场景
- ✅ 多租户数据隔离场景
- ✅ 自动标签工作流程
- ✅ 错误处理与降级策略

### 设计文档 (`02-design/`)

#### 3. [系统架构图](./02-design/01-system-architecture.md)

使用 Mermaid 语法绘制的系统架构图，包含：

- ✅ 整体架构图（Mermaid）
- ✅ 数据流架构图
- ✅ 模块依赖关系图
- ✅ 部署架构图
- ✅ API 调用关系图
- ✅ 数据库关系图
- ✅ 技术栈依赖图
- ✅ 服务启动顺序图
- ✅ 消息流转完整路径
- ✅ 环境变量配置关系

---

## 🚀 快速开始

### 1. 查看项目现状

阅读 [项目现状梳理报告](./01-product/01-project-overview.md) 了解项目整体情况、技术路线决策和关键问题。

**推荐阅读顺序**：
- 新加入项目的开发者：先阅读项目现状报告，了解项目背景和技术路线
- 产品经理/业务人员：重点关注业务场景图和项目现状中的功能模块部分
- 架构师/技术负责人：重点关注系统架构图和项目现状中的技术架构部分

### 2. 理解系统架构

查看 [系统架构图](./02-design/01-system-architecture.md) 了解技术架构、模块关系和部署方案。

**包含内容**：
- 整体架构设计
- 各模块之间的依赖关系
- 数据流转路径
- 部署架构方案

### 3. 了解业务场景

查看 [业务场景图](./01-product/02-business-scenarios.md) 了解不同角色的使用流程和业务逻辑。

**适用场景**：
- 理解用户如何使用系统
- 了解商户管理员如何配置 AI 助手
- 理解平台管理员的管理流程
- 了解多渠道接入的实现方式

---

## 📖 文档使用说明

### Mermaid 图表查看

文档中包含大量 Mermaid 图表，可以在以下平台查看：

- **GitHub/GitLab**: 直接支持 Mermaid 渲染，推荐在 GitHub 上查看
- **VS Code**: 安装 "Markdown Preview Mermaid Support" 插件
- **在线编辑器**: [Mermaid Live Editor](https://mermaid.live/)
- **Notion/Obsidian**: 支持 Mermaid 语法

### 文档阅读建议

1. **首次阅读**：建议按照"快速开始"的顺序阅读文档
2. **深入理解**：可以结合代码仓库一起阅读，理解实现细节
3. **定期更新**：文档会随着项目进展持续更新，建议定期查看最新版本

### 文档维护

- 📝 文档会随着项目进展持续更新
- 🔄 建议定期查看最新版本
- 💡 如有问题或建议，请提交 Issue 或 PR
- 📧 文档相关问题请联系项目维护者

---

## 🔗 相关链接

### 核心项目仓库

- **[asair-admin-backend](https://github.com/zhouyuChina/asair-admin-backend)** - 管理后台后端服务
  - Flask + PostgreSQL
  - 提供平台管理和租户管理 API
  - 查看 [CONTRIBUTING.md](https://github.com/zhouyuChina/asair-admin-backend/blob/master/CONTRIBUTING.md) 了解开发指南

- **[asair-admin-frontend](https://github.com/zhouyuChina/asair-admin-frontend)** - 管理后台前端
  - 商户和平台管理界面

- **[asair-ai-server](https://github.com/zhouyuChina/asair-ai-server)** - AI 大脑服务
  - LLM 大模型集成
  - AI 对话处理

- **[asair-infra](https://github.com/zhouyuChina/asair-infra)** - 基础设施配置
  - Docker 配置
  - 部署脚本

---

## 📁 文档结构

```
asair-doc/
├── README.md                          # 本文档（文档索引）
├── 01-product/                        # 产品相关文档
│   ├── 01-project-overview.md         # 项目现状梳理报告
│   └── 02-business-scenarios.md       # 业务场景图
└── 02-design/                         # 设计相关文档
    └── 01-system-architecture.md      # 系统架构图
```

## 📝 文档维护

**最后更新**: 2025-12-12

**维护者**: ASAIR 项目团队

**更新频率**: 文档会随着项目进展持续更新，重要变更会及时同步

---

## 📄 许可证

本文档遵循项目整体许可证。

## 📞 支持

如有问题或需要支持，请：

- 📧 在 GitHub 上提交 Issue
- 💬 联系项目维护者
- 📖 查看各模块仓库的详细文档
