# ASAIR 多渠道 AI 客服平台 - 业务场景图

本文档从产品使用角度，使用 Mermaid 语法绘制各种业务场景图，展示不同角色如何使用系统。

---

## 一、用户与 AI 客服对话完整流程

```mermaid
flowchart TD
    START[用户打开聊天] --> CHANNEL{选择渠道}
    CHANNEL -->|Telegram| TG[Telegram Bot]
    CHANNEL -->|WhatsApp| WA[WhatsApp]
    CHANNEL -->|LINE| LINE[LINE]
    CHANNEL -->|Web| WEB[网站聊天窗口]
    
    TG --> SEND1[发送第一条消息]
    WA --> SEND1
    LINE --> SEND1
    WEB --> SEND1
    
    SEND1 --> CW_RECEIVE[Chatwoot 接收消息]
    CW_RECEIVE --> CREATE_CONV[创建会话]
    CREATE_CONV --> WELCOME[发送欢迎语]
    WELCOME --> AI_PROCESS[AI 处理消息]
    
    AI_PROCESS --> CHECK_MEMORY{检查用户记忆}
    CHECK_MEMORY -->|有记忆| USE_MEMORY[使用历史记忆]
    CHECK_MEMORY -->|无记忆| NEW_USER[新用户流程]
    
    USE_MEMORY --> CHECK_FAVOR{检查好感度}
    NEW_USER --> CHECK_FAVOR
    
    CHECK_FAVOR -->|高好感度| FRIENDLY_TONE[使用熟络语气]
    CHECK_FAVOR -->|低好感度| FORMAL_TONE[使用正式语气]
    
    FRIENDLY_TONE --> GEN_REPLY[生成 AI 回复]
    FORMAL_TONE --> GEN_REPLY
    
    GEN_REPLY --> SEND_REPLY[发送回复给用户]
    SEND_REPLY --> USER_READ[用户查看回复]
    
    USER_READ --> USER_REPLY{用户继续对话?}
    USER_REPLY -->|是| CONTINUE[继续对话循环]
    USER_REPLY -->|否| END_CONV[结束会话]
    
    CONTINUE --> AI_PROCESS
    
    END_CONV --> UPDATE_PROFILE[更新用户画像]
    UPDATE_PROFILE --> SAVE_LOG[保存对话记录]
    
    style START fill:#e1f5ff
    style AI_PROCESS fill:#ffe1f5
    style GEN_REPLY fill:#f3e5f5
    style END_CONV fill:#c8e6c9
```

---

## 二、商户管理员配置 AI 助手流程

```mermaid
flowchart LR
    LOGIN[登录管理后台] --> DASHBOARD[进入仪表盘]
    DASHBOARD --> AI_CONFIG[进入 AI 配置]
    
    AI_CONFIG --> PROMPT[Prompt 工程]
    AI_CONFIG --> KNOWLEDGE[知识库管理]
    AI_CONFIG --> AUTO_TAG[自动标签]
    AI_CONFIG --> HANDOFF[转人工规则]
    
    PROMPT --> EDIT_PROMPT[编辑系统提示词]
    EDIT_PROMPT --> SET_ROLE[设置角色定位]
    SET_ROLE --> SET_TONE[设置对话语气]
    SET_TONE --> SET_RULES[设置回复规则]
    SET_RULES --> PREVIEW[预览效果]
    PREVIEW --> SAVE_PROMPT[保存并发布]
    
    KNOWLEDGE --> ADD_KNOWLEDGE[新增知识条目]
    ADD_KNOWLEDGE --> SET_CATEGORY[设置分类]
    SET_CATEGORY --> UPLOAD_CONTENT[上传内容]
    UPLOAD_CONTENT --> SET_RETRIEVAL[设置检索配置]
    SET_RETRIEVAL --> SAVE_KNOWLEDGE[保存知识]
    
    AUTO_TAG --> CREATE_TAG[创建标签规则]
    CREATE_TAG --> SET_KEYWORDS[设置触发词]
    SET_KEYWORDS --> SET_COLOR[设置标签颜色]
    SET_COLOR --> ENABLE_AUTO[启用自动打标签]
    ENABLE_AUTO --> SAVE_TAG[保存标签规则]
    
    HANDOFF --> SET_CONDITIONS[设置触发条件]
    SET_CONDITIONS --> SET_THRESHOLD[设置阈值]
    SET_THRESHOLD --> SET_MESSAGE[设置转人工提示语]
    SET_MESSAGE --> SAVE_HANDOFF[保存转人工规则]
    
    SAVE_PROMPT --> TEST[测试 AI 回复]
    SAVE_KNOWLEDGE --> TEST
    SAVE_TAG --> TEST
    SAVE_HANDOFF --> TEST
    
    TEST --> VERIFY[验证配置效果]
    VERIFY --> COMPLETE[配置完成]
    
    style LOGIN fill:#e1f5ff
    style AI_CONFIG fill:#fff4e1
    style TEST fill:#ffe1f5
    style COMPLETE fill:#c8e6c9
```

---

## 三、平台管理员管理商户流程

```mermaid
flowchart TD
    PLATFORM_LOGIN[平台管理员登录] --> PLATFORM_DASHBOARD[平台管理后台]
    
    PLATFORM_DASHBOARD --> TENANT_LIST[商户列表]
    PLATFORM_DASHBOARD --> MODEL_GATEWAY[模型网关配置]
    PLATFORM_DASHBOARD --> PLATFORM_REPORT[平台报表]
    
    TENANT_LIST --> VIEW_TENANT[查看商户详情]
    TENANT_LIST --> CREATE_TENANT[创建新商户]
    TENANT_LIST --> EDIT_TENANT[编辑商户信息]
    TENANT_LIST --> DISABLE_TENANT[停用商户]
    
    CREATE_TENANT --> FILL_INFO[填写商户信息]
    FILL_INFO --> SET_QUOTA[设置配额限制]
    SET_QUOTA --> CREATE_ADMIN[创建商户管理员]
    CREATE_ADMIN --> ACTIVATE[激活商户]
    
    VIEW_TENANT --> CHECK_STATUS[查看商户状态]
    VIEW_TENANT --> CHECK_USAGE[查看使用情况]
    VIEW_TENANT --> CHECK_CHANNELS[查看渠道配置]
    
    EDIT_TENANT --> UPDATE_INFO[更新基本信息]
    EDIT_TENANT --> ADJUST_QUOTA[调整配额]
    EDIT_TENANT --> CHANGE_STATUS[修改状态]
    
    MODEL_GATEWAY --> ADD_MODEL[添加模型]
    ADD_MODEL --> SET_CONFIG[配置模型参数]
    SET_CONFIG --> SET_PRICE[设置价格]
    SET_PRICE --> ENABLE_MODEL[启用模型]
    
    PLATFORM_REPORT --> VIEW_USAGE[查看使用量统计]
    VIEW_USAGE --> VIEW_COST[查看成本分析]
    VIEW_COST --> EXPORT_REPORT[导出报表]
    
    style PLATFORM_LOGIN fill:#e1f5ff
    style CREATE_TENANT fill:#fff4e1
    style ACTIVATE fill:#c8e6c9
```

---

## 四、人工客服接管流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant AI as AI 客服
    participant System as 系统
    participant Agent as 人工客服
    participant Chatwoot as Chatwoot 工作台

    Note over User,Chatwoot: AI 自动接待阶段
    
    User->>AI: 发送消息
    AI->>System: 处理消息
    System->>AI: 生成回复
    AI->>User: 发送 AI 回复
    
    User->>AI: 继续提问
    AI->>System: 检测到复杂问题
    System->>System: 判断需要转人工
    
    Note over User,Chatwoot: 触发转人工
    
    System->>Chatwoot: 标记会话需要人工
    Chatwoot->>Agent: 推送待处理会话通知
    System->>User: 发送转人工提示
    
    Note over User,Chatwoot: 人工客服接管
    
    Agent->>Chatwoot: 打开会话详情
    Chatwoot->>Agent: 显示完整对话历史
    Chatwoot->>Agent: 显示用户画像和好感度
    Agent->>Chatwoot: 点击"接管会话"
    Chatwoot->>System: 通知 AI 停止自动回复
    System->>AI: 暂停自动回复
    
    Note over User,Chatwoot: 人工回复阶段
    
    Agent->>Chatwoot: 输入回复内容
    Chatwoot->>User: 发送人工回复
    User->>Agent: 继续对话
    Agent->>Chatwoot: 使用快捷回复
    Chatwoot->>User: 发送回复
    
    Note over User,Chatwoot: 结束接管
    
    Agent->>Chatwoot: 标记会话已解决
    Agent->>Chatwoot: 点击"结束接管"
    Chatwoot->>System: 恢复 AI 自动回复
    System->>AI: 重新启用自动回复
    Agent->>Chatwoot: 添加会话标签和备注
```

---

## 五、多渠道接入场景

```mermaid
graph TB
    subgraph "用户端"
        USER1[用户 A<br/>Telegram]
        USER2[用户 B<br/>WhatsApp]
        USER3[用户 C<br/>LINE]
        USER4[用户 D<br/>Web Widget]
    end
    
    subgraph "渠道层"
        TG_BOT[Telegram Bot]
        WA_BOT[WhatsApp Business]
        LINE_BOT[LINE Messaging API]
        WEB_WIDGET[Web Chat Widget]
    end
    
    subgraph "Chatwoot 统一管理"
        CW_INBOX1[Telegram Inbox]
        CW_INBOX2[WhatsApp Inbox]
        CW_INBOX3[LINE Inbox]
        CW_INBOX4[Web Widget Inbox]
        CW_WORKSPACE[Chatwoot 工作空间]
    end
    
    subgraph "AI 处理层"
        AI_SERVER[asair-ai-server]
        AI_ENGINE[AI 流程引擎]
    end
    
    subgraph "配置层"
        ADMIN_BACKEND[asair-admin-backend]
        CONFIG_DB[(配置数据库)]
    end
    
    USER1 -->|发送消息| TG_BOT
    USER2 -->|发送消息| WA_BOT
    USER3 -->|发送消息| LINE_BOT
    USER4 -->|发送消息| WEB_WIDGET
    
    TG_BOT -->|Webhook| CW_INBOX1
    WA_BOT -->|Webhook| CW_INBOX2
    LINE_BOT -->|Webhook| CW_INBOX3
    WEB_WIDGET -->|HTTP| CW_INBOX4
    
    CW_INBOX1 --> CW_WORKSPACE
    CW_INBOX2 --> CW_WORKSPACE
    CW_INBOX3 --> CW_WORKSPACE
    CW_INBOX4 --> CW_WORKSPACE
    
    CW_WORKSPACE -->|Webhook| AI_SERVER
    AI_SERVER --> AI_ENGINE
    AI_ENGINE -->|获取配置| ADMIN_BACKEND
    ADMIN_BACKEND --> CONFIG_DB
    
    AI_ENGINE -->|生成回复| AI_SERVER
    AI_SERVER -->|API 调用| CW_WORKSPACE
    CW_WORKSPACE -->|推送消息| TG_BOT
    CW_WORKSPACE -->|推送消息| WA_BOT
    CW_WORKSPACE -->|推送消息| LINE_BOT
    CW_WORKSPACE -->|推送消息| WEB_WIDGET
    
    TG_BOT -->|显示回复| USER1
    WA_BOT -->|显示回复| USER2
    LINE_BOT -->|显示回复| USER3
    WEB_WIDGET -->|显示回复| USER4
    
    style USER1 fill:#e1f5ff
    style USER2 fill:#e1f5ff
    style USER3 fill:#e1f5ff
    style USER4 fill:#e1f5ff
    style AI_SERVER fill:#ffe1f5
    style CW_WORKSPACE fill:#e8f5e9
```

---

## 六、好感度系统工作流程

```mermaid
stateDiagram-v2
    [*] --> 新用户: 首次对话
    新用户 --> 冷淡: 好感度 < 20
    新用户 --> 普通: 好感度 20-50
    新用户 --> 熟悉: 好感度 50-80
    新用户 --> 亲密: 好感度 > 80
    
    冷淡 --> 普通: 积极互动 +5
    普通 --> 熟悉: 多次愉快对话 +10
    熟悉 --> 亲密: 长期良好关系 +15
    
    亲密 --> 熟悉: 长期未互动 -5
    熟悉 --> 普通: 投诉/负面情绪 -10
    普通 --> 冷淡: 严重投诉 -15
    
    冷淡: 语气: 正式礼貌<br/>策略: 标准回复
    普通: 语气: 友好专业<br/>策略: 个性化回复
    熟悉: 语气: 熟络亲切<br/>策略: 记忆增强
    亲密: 语气: 亲密自然<br/>策略: 深度个性化
    
    note right of 冷淡
        好感度计算规则:
        - 时间衰减: T(x) = 1.67 * exp(-0.0183 * x) + 0.33
        - 积极度正则: P(x) = 5.165 * tanh(0.0689 * x)
        - 分数权重: S(x) = 2.5 * exp(-(x-30)²/(2*14.57²)) + 0.5
    end note
```

---

## 七、转人工触发场景

```mermaid
flowchart TD
    USER_MSG[用户发送消息] --> AI_PROCESS[AI 处理消息]
    
    AI_PROCESS --> CHECK1{用户明确要求<br/>转人工?}
    CHECK1 -->|是| TRIGGER1[触发转人工]
    CHECK1 -->|否| CHECK2{连续多次<br/>无法回答?}
    
    CHECK2 -->|是| CHECK_THRESHOLD{达到阈值?}
    CHECK_THRESHOLD -->|是| TRIGGER2[触发转人工]
    CHECK_THRESHOLD -->|否| CONTINUE_AI[继续 AI 回复]
    
    CHECK2 -->|否| CHECK3{检测到负面情绪?}
    CHECK3 -->|是| CHECK_SENTIMENT{情绪强度?}
    CHECK_SENTIMENT -->|高| TRIGGER3[触发转人工]
    CHECK_SENTIMENT -->|中| FLAG[标记关注]
    CHECK_SENTIMENT -->|低| CONTINUE_AI
    
    CHECK3 -->|否| CHECK4{命中高价值规则?}
    CHECK4 -->|是| TRIGGER4[触发转人工]
    CHECK4 -->|否| CHECK5{大金额交易?}
    
    CHECK5 -->|是| TRIGGER5[触发转人工]
    CHECK5 -->|否| CHECK6{投诉场景?}
    
    CHECK6 -->|是| TRIGGER6[触发转人工]
    CHECK6 -->|否| CONTINUE_AI
    
    TRIGGER1 --> NOTIFY[通知人工客服]
    TRIGGER2 --> NOTIFY
    TRIGGER3 --> NOTIFY
    TRIGGER4 --> NOTIFY
    TRIGGER5 --> NOTIFY
    TRIGGER6 --> NOTIFY
    
    NOTIFY --> SEND_TIP[发送转人工提示]
    SEND_TIP --> WAIT_AGENT[等待客服接管]
    WAIT_AGENT --> AGENT_TAKE[人工客服接管]
    
    CONTINUE_AI --> AI_REPLY[AI 继续回复]
    FLAG --> AI_REPLY
    
    style TRIGGER1 fill:#ffcdd2
    style TRIGGER2 fill:#ffcdd2
    style TRIGGER3 fill:#ffcdd2
    style TRIGGER4 fill:#ffcdd2
    style TRIGGER5 fill:#ffcdd2
    style TRIGGER6 fill:#ffcdd2
    style AGENT_TAKE fill:#c8e6c9
```

---

## 八、会话节奏控制场景

```mermaid
sequenceDiagram
    participant User as 用户
    participant System as 节奏管理器
    participant AI as AI 服务
    participant LLM as OpenAI API

    Note over User,LLM: 用户连续发送多条消息
    
    User->>System: 消息 1: "你好"
    System->>System: 记录消息时间戳
    System->>System: 启动计时器 (5秒)
    
    User->>System: 消息 2: "我想咨询"
    System->>System: 重置计时器
    
    User->>System: 消息 3: "产品价格"
    System->>System: 重置计时器
    
    Note over User,LLM: 用户停顿超过 5 秒
    
    System->>System: 计时器到期
    System->>System: 合并消息: "你好 我想咨询 产品价格"
    System->>AI: 发送合并后的消息
    AI->>LLM: 调用 LLM 生成回复
    LLM-->>AI: 返回 AI 回复
    AI-->>System: 返回回复内容
    System->>User: 发送综合回复
    
    Note over User,LLM: 用户继续发送消息
    
    User->>System: 消息 4: "还有优惠吗"
    System->>System: 检查: 距离上次回复 < 10秒
    System->>System: 合并到新批次
    System->>System: 启动新计时器
    
    User->>System: 消息 5: "什么时候发货"
    System->>System: 重置计时器
    
    Note over User,LLM: 达到最大合并条数 (5条)
    
    System->>System: 达到最大合并数
    System->>AI: 立即处理当前批次
    AI->>LLM: 调用 LLM
    LLM-->>AI: 返回回复
    AI-->>System: 返回回复
    System->>User: 发送回复
```

---

## 九、商户运营工作流

```mermaid
flowchart LR
    START[商户入驻] --> SETUP[初始配置]
    
    SETUP --> CONFIG1[配置 AI 助手]
    SETUP --> CONFIG2[配置渠道]
    SETUP --> CONFIG3[配置欢迎语]
    
    CONFIG1 --> TEST[测试对话]
    CONFIG2 --> TEST
    CONFIG3 --> TEST
    
    TEST --> LAUNCH[正式上线]
    
    LAUNCH --> MONITOR[监控运营]
    
    MONITOR --> ANALYZE[分析数据]
    ANALYZE --> OPTIMIZE{需要优化?}
    
    OPTIMIZE -->|是| ADJUST1[调整 Prompt]
    OPTIMIZE -->|是| ADJUST2[调整转人工规则]
    OPTIMIZE -->|是| ADJUST3[更新知识库]
    OPTIMIZE -->|否| CONTINUE[继续运营]
    
    ADJUST1 --> TEST
    ADJUST2 --> TEST
    ADJUST3 --> TEST
    
    CONTINUE --> MONITOR
    
    MONITOR --> REPORT[查看报表]
    REPORT --> COST[成本分析]
    REPORT --> EFFECT[效果分析]
    REPORT --> USER[用户分析]
    
    COST --> DECISION{成本合理?}
    EFFECT --> DECISION
    USER --> DECISION
    
    DECISION -->|是| CONTINUE
    DECISION -->|否| OPTIMIZE
    
    style START fill:#e1f5ff
    style LAUNCH fill:#c8e6c9
    style MONITOR fill:#fff4e1
    style OPTIMIZE fill:#ffe1f5
```

---

## 十、用户旅程图

```mermaid
journey
    title 用户使用 AI 客服的完整旅程
    
    section 发现阶段
      访问网站/打开 App: 5: 用户
      看到聊天入口: 4: 用户
      点击开始聊天: 5: 用户
    
    section 首次接触
      收到欢迎语: 3: AI
      了解服务介绍: 4: AI
      提出第一个问题: 5: 用户
    
    section 互动阶段
      AI 回答问题: 4: AI
      继续提问: 5: 用户
      AI 提供详细解答: 5: AI
      询问价格/优惠: 5: 用户
      AI 提供报价: 4: AI
    
    section 决策阶段
      考虑购买: 3: 用户
      提出疑虑: 4: 用户
      AI 解答疑虑: 4: AI
      要求转人工: 5: 用户
    
    section 人工服务
      人工客服接入: 5: 客服
      详细沟通: 5: 用户, 客服
      达成交易: 5: 用户, 客服
    
    section 后续服务
      收到确认信息: 4: AI
      询问物流: 4: 用户
      AI 提供物流信息: 4: AI
      服务完成: 5: 用户
```

---

## 十一、知识库检索场景

```mermaid
flowchart TD
    USER_QUESTION[用户提问] --> AI_RECEIVE[AI 接收问题]
    
    AI_RECEIVE --> EMBED[问题向量化]
    EMBED --> SEARCH[检索知识库]
    
    SEARCH --> CHECK_SIMILARITY{相似度检查}
    CHECK_SIMILARITY -->|相似度 < 阈值| NO_MATCH[无匹配知识]
    CHECK_SIMILARITY -->|相似度 >= 阈值| GET_TOP[获取 Top N 结果]
    
    GET_TOP --> RANK[按相似度排序]
    RANK --> FILTER[过滤低质量结果]
    FILTER --> CONTEXT[构建上下文]
    
    CONTEXT --> PROMPT[构建 Prompt]
    PROMPT --> LLM[调用 LLM]
    LLM --> GENERATE[生成回复]
    
    NO_MATCH --> FALLBACK[使用默认回复]
    FALLBACK --> CHECK_HANDOFF{需要转人工?}
    
    GENERATE --> CHECK_QUALITY{回复质量检查}
    CHECK_QUALITY -->|质量低| RETRY[重试生成]
    CHECK_QUALITY -->|质量高| SEND[发送回复]
    
    RETRY --> LLM
    CHECK_HANDOFF -->|是| HANDOFF[转人工]
    CHECK_HANDOFF -->|否| SEND
    
    SEND --> UPDATE_LOG[更新检索日志]
    HANDOFF --> UPDATE_LOG
    UPDATE_LOG --> END[完成]
    
    style USER_QUESTION fill:#e1f5ff
    style SEARCH fill:#fff4e1
    style LLM fill:#f3e5f5
    style SEND fill:#c8e6c9
```

---

## 十二、报表查看与分析场景

```mermaid
flowchart LR
    LOGIN[登录管理后台] --> DASHBOARD[进入仪表盘]
    
    DASHBOARD --> OVERVIEW[数据概览]
    OVERVIEW --> SESSION[会话统计]
    OVERVIEW --> USER[用户统计]
    OVERVIEW --> COST[成本统计]
    
    SESSION --> FILTER1[选择时间范围]
    FILTER1 --> FILTER2[选择渠道]
    FILTER2 --> VIEW1[查看会话趋势]
    VIEW1 --> EXPORT1[导出会话报表]
    
    USER --> FILTER3[选择商户]
    FILTER3 --> VIEW2[查看用户画像]
    VIEW2 --> VIEW3[查看好感度分布]
    VIEW3 --> EXPORT2[导出用户报表]
    
    COST --> FILTER4[选择模型]
    FILTER4 --> VIEW4[查看 Token 使用量]
    VIEW4 --> VIEW5[查看成本趋势]
    VIEW5 --> VIEW6[查看成本占比]
    VIEW6 --> EXPORT3[导出成本报表]
    
    DASHBOARD --> DETAIL[详细分析]
    DETAIL --> AI_EFFECT[AI 效果分析]
    DETAIL --> HANDOFF_RATE[转人工率分析]
    DETAIL --> SATISFACTION[满意度分析]
    
    AI_EFFECT --> COMPARE[对比不同配置]
    HANDOFF_RATE --> OPTIMIZE[优化转人工规则]
    SATISFACTION --> IMPROVE[改进 AI 回复]
    
    style LOGIN fill:#e1f5ff
    style DASHBOARD fill:#fff4e1
    style COST fill:#ffe1f5
    style OPTIMIZE fill:#c8e6c9
```

---

## 十三、多租户数据隔离场景

```mermaid
graph TB
    subgraph "平台管理员视图"
        PLATFORM_ADMIN[平台管理员]
        PLATFORM_VIEW[查看所有商户数据]
    end
    
    subgraph "商户 A"
        TENANT_A[商户 A 管理员]
        DATA_A[(商户 A 数据)]
        CONFIG_A[商户 A 配置]
        SESSION_A[商户 A 会话]
    end
    
    subgraph "商户 B"
        TENANT_B[商户 B 管理员]
        DATA_B[(商户 B 数据)]
        CONFIG_B[商户 B 配置]
        SESSION_B[商户 B 会话]
    end
    
    subgraph "商户 C"
        TENANT_C[商户 C 管理员]
        DATA_C[(商户 C 数据)]
        CONFIG_C[商户 C 配置]
        SESSION_C[商户 C 会话]
    end
    
    PLATFORM_ADMIN --> PLATFORM_VIEW
    PLATFORM_VIEW --> DATA_A
    PLATFORM_VIEW --> DATA_B
    PLATFORM_VIEW --> DATA_C
    
    TENANT_A --> CONFIG_A
    TENANT_A --> SESSION_A
    CONFIG_A --> DATA_A
    SESSION_A --> DATA_A
    
    TENANT_B --> CONFIG_B
    TENANT_B --> SESSION_B
    CONFIG_B --> DATA_B
    SESSION_B --> DATA_B
    
    TENANT_C --> CONFIG_C
    TENANT_C --> SESSION_C
    CONFIG_C --> DATA_C
    SESSION_C --> DATA_C
    
    DATA_A -.->|数据隔离| DATA_B
    DATA_B -.->|数据隔离| DATA_C
    DATA_C -.->|数据隔离| DATA_A
    
    style PLATFORM_ADMIN fill:#e1f5ff
    style TENANT_A fill:#fff4e1
    style TENANT_B fill:#fff4e1
    style TENANT_C fill:#fff4e1
    style DATA_A fill:#ffebee
    style DATA_B fill:#ffebee
    style DATA_C fill:#ffebee
```

---

## 十四、自动标签工作流程

```mermaid
flowchart TD
    USER_MSG[用户发送消息] --> AI_PROCESS[AI 处理消息]
    
    AI_PROCESS --> EXTRACT[提取关键词]
    EXTRACT --> MATCH[匹配标签规则]
    
    MATCH --> CHECK1{匹配规则1?}
    CHECK1 -->|是| TAG1[添加标签: 高意向]
    CHECK1 -->|否| CHECK2{匹配规则2?}
    
    CHECK2 -->|是| TAG2[添加标签: 待跟进]
    CHECK2 -->|否| CHECK3{匹配规则3?}
    
    CHECK3 -->|是| TAG3[添加标签: 技术咨询]
    CHECK3 -->|否| CHECK4{匹配规则N?}
    
    CHECK4 -->|是| TAGN[添加标签: 其他]
    CHECK4 -->|否| NO_TAG[不添加标签]
    
    TAG1 --> MULTI_CHECK{允许多标签?}
    TAG2 --> MULTI_CHECK
    TAG3 --> MULTI_CHECK
    TAGN --> MULTI_CHECK
    
    MULTI_CHECK -->|是| ADD_ALL[添加所有匹配标签]
    MULTI_CHECK -->|否| ADD_FIRST[只添加第一个匹配标签]
    
    ADD_ALL --> UPDATE_PROFILE[更新用户画像]
    ADD_FIRST --> UPDATE_PROFILE
    NO_TAG --> UPDATE_PROFILE
    
    UPDATE_PROFILE --> CHECK_EXPIRE{检查标签有效期}
    CHECK_EXPIRE -->|过期| REMOVE[移除过期标签]
    CHECK_EXPIRE -->|未过期| KEEP[保留标签]
    
    REMOVE --> NOTIFY[通知客服]
    KEEP --> NOTIFY
    NOTIFY --> END[完成]
    
    style USER_MSG fill:#e1f5ff
    style TAG1 fill:#c8e6c9
    style TAG2 fill:#fff9c4
    style TAG3 fill:#e1f5ff
    style UPDATE_PROFILE fill:#ffe1f5
```

---

## 十五、错误处理与降级策略

```mermaid
flowchart TD
    REQUEST[收到请求] --> VALIDATE{请求验证}
    
    VALIDATE -->|无效| ERROR1[返回 400 错误]
    VALIDATE -->|有效| PROCESS[处理请求]
    
    PROCESS --> CALL_LLM[调用 OpenAI API]
    
    CALL_LLM --> CHECK_RESPONSE{API 响应}
    CHECK_RESPONSE -->|成功| PARSE[解析回复]
    CHECK_RESPONSE -->|超时| RETRY1{重试次数 < 3?}
    CHECK_RESPONSE -->|限流| RETRY2{重试次数 < 3?}
    CHECK_RESPONSE -->|API 错误| FALLBACK1[使用兜底回复]
    
    RETRY1 -->|是| WAIT1[等待后重试]
    WAIT1 --> CALL_LLM
    RETRY1 -->|否| FALLBACK1
    
    RETRY2 -->|是| WAIT2[等待后重试]
    WAIT2 --> CALL_LLM
    RETRY2 -->|否| FALLBACK1
    
    PARSE --> CHECK_QUALITY{回复质量检查}
    CHECK_QUALITY -->|质量好| SEND[发送回复]
    CHECK_QUALITY -->|质量差| RETRY3{重试次数 < 3?}
    
    RETRY3 -->|是| CALL_LLM
    RETRY3 -->|否| FALLBACK2[使用固定回复]
    
    FALLBACK1 --> LOG_ERROR[记录错误日志]
    FALLBACK2 --> LOG_ERROR
    SEND --> LOG_SUCCESS[记录成功日志]
    
    LOG_ERROR --> NOTIFY[通知管理员]
    LOG_SUCCESS --> END[完成]
    NOTIFY --> END
    
    style REQUEST fill:#e1f5ff
    style CALL_LLM fill:#f3e5f5
    style FALLBACK1 fill:#ffcdd2
    style FALLBACK2 fill:#ffcdd2
    style SEND fill:#c8e6c9
```

---

## 使用说明

以上业务场景图使用 Mermaid 语法编写，展示了：

1. **用户视角**：用户与 AI 客服的完整对话流程
2. **商户管理员视角**：配置 AI 助手、知识库、标签规则等
3. **平台管理员视角**：管理商户、配置模型、查看报表
4. **人工客服视角**：接管会话、处理复杂问题
5. **系统视角**：多渠道接入、好感度系统、转人工触发、节奏控制等

这些图表可以在支持 Mermaid 的平台查看，如：
- GitHub/GitLab
- VS Code（安装 Mermaid 插件）
- https://mermaid.live/
- Notion/Obsidian

---

**最后更新**: 2025-01-XX

