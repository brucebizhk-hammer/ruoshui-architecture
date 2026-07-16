# 若水系统优化架构方案：两级数据库 + Chatwoot + Agent Core

> 日期：2026-07-06  
> 输入依据：桌面三张架构图、当前运行代码、Chatwoot/Respond.io/Intercom/Zendesk 等市面方案  
> 当前核心判断：Chatwoot 做展示与协作，Agent Core 做业务大脑，两级数据库做长期事实与记忆，Hermes/Codex/MCP 做智能工具后端。

---

## 1. 市面方案给我们的启发

### 1.1 Chatwoot：适合做前台，不适合做大脑

Chatwoot 的 API Channel 能创建自定义 inbox，并按 contact -> conversation -> message 写入外部渠道消息。它适合解决：

- 多渠道消息展示。
- 联系人、会话、标签、团队协作。
- 人工回复工作台。
- 微信这种自定义渠道的旁路展示。

但 Chatwoot 不适合承载：

- 磷矿/硫磺/铜矿等行业知识库。
- 客户深度档案。
- 人脉关系图。
- AI 多 Agent 编排。
- 自动发送风控。
- 微信 DLL 真实发送。

结论：Chatwoot 是“前台工作台 + CRM 展示层”，不是唯一数据库和智能中枢。

### 1.2 Respond.io：适合参考“生命周期 + 动作执行”

Respond.io 的 AI Agent 方向是：跨 WhatsApp、TikTok、Instagram、Facebook 等渠道，自动回复、分配会话、更新生命周期阶段、调用外部 HTTP 动作。这个思路非常接近我们要做的：

```text
消息 -> 判断阶段/角色/业务状态 -> 调外部系统 -> 更新联系人/CRM -> 回复
```

但它也暴露一个问题：平台级 AI Agent 往往对完整历史、元数据、内部上下文可见性有限。因此我们的长期记忆不能依赖平台内置 AI，要自己建。

### 1.3 Intercom Fin：适合参考“知识源管理”

Intercom Fin 的核心不是只会聊天，而是有集中 Knowledge Sources：

- 帮助中心文章。
- 内部资料。
- PDF/网页。
- 历史对话。
- 外部同步内容。

这正好对应我们的一级库：

```text
行业知识库 = 若水自己的 Knowledge Sources
```

但 Intercom 的知识源偏客服知识，不能直接满足我们大宗商品市场价、矿种指标、合同报价、人脉关系等复杂业务知识。

### 1.4 Zendesk：适合参考“全渠道路由 + 优先级”

Zendesk 的强项是 omnichannel routing：按渠道、优先级、坐席状态、SLA 和队列把消息路由到正确处理人。我们要借鉴的不是它的重型客服体系，而是：

- 所有渠道进入统一队列。
- 每条消息有优先级。
- 按身份/客户等级/业务类型路由。
- 避免重要客户被普通消息淹没。

---

## 2. 当前系统真实状态

当前已具备：

```text
WeChat Hook: OK
Middleware 5000/5001: OK
Agent Core 8765: OK
Hermes API 8642: OK
Chatwoot 3000: 已旁路展示
Unified Memory DB: 已存在
```

当前 Unified DB 状态：

```text
contacts: 442
contact_identities: 442
conversations: 442
messages: 1507
outbox: 21
platform_accounts: 2
memory_items: 0
```

最关键缺口：

```text
memory_items = 0
```

说明当前系统已经能“收、存、发、同步”，但还没有真正形成长期记忆。Hermes 对话里看似有记忆，本质仍是 session 工作记忆，不是可控、可查询、可迁移的长期事实库。

---

## 3. 优化后的总架构

```text
前台层：
  Chatwoot 3000
  - 多渠道 inbox
  - 联系人/会话/标签/人工协作
  - 只展示，不做唯一事实库

渠道层：
  WeChat Adapter 5000/5001/5010/5011
  WhatsApp/Facebook/Instagram/TikTok Adapter
  - 收消息
  - 下载媒体
  - 标准化
  - 发送

中枢层：
  Ruoshui Agent Core 8765
  - 去重
  - 路由
  - 记忆包组装
  - 多 Agent 编排
  - 风控
  - outbox

数据库层：
  一级行业知识库
  二级客户个人档案库
  原始消息账本
  任务/承诺/业务事实
  向量索引/检索索引

智能层：
  Hermes API Server 8642
  Codex/本地工具
  MCP 工具集合
  搜索/浏览器/文件/表格/合同解析
```

---

## 4. 两级数据库优化设计

### 4.1 一级库：行业专业知识库

定位：

```text
回答“现在市场什么价、什么指标、什么政策、什么供应信息”
```

特点：

- 板块隔离。
- 每个板块独立 schema。
- 每日自动抓取。
- 只存行业事实，不存具体客户隐私。
- AI 查询时即时读取，不靠长期聊天 session 记忆。

推荐目录：

```text
data/industry/
  sulfur.db
  phosphate.db
  copper.db
  gypsum.db
  energy.db
  sugar.db
```

推荐统一字段：

```text
industry_facts
- id
- module
- fact_type
- subject
- value
- unit
- region
- source
- source_url
- confidence
- observed_at
- expires_at
- raw_json
```

板块示例：

```text
硫磺：
- CFR中国现货/期货
- SOCAR/SAGIZ/Sirakopol 报价
- 港口库存
- 下游开工率

磷矿：
- 湖北矿区出厂价
- 运矿成本
- 磷酸/磷肥联动
- 出口政策

铜矿：
- LME/沪铜期货
- TC/RC
- 全球矿山产能
- 港口库存

a石膏粉：
- a50出厂价
- 华南/华东需求
- 贸易壁垒

能源/煤炭：
- 动力煤/焦煤价格
- 供需缺口
- 运价指数

白糖/萤石/其他：
- 白糖期货
- 萤石报价
- 台账状态
```

### 4.2 二级库：客户个人档案库

定位：

```text
回答“这个人是谁、什么风格、什么关系、之前聊了什么、承诺了什么”
```

特点：

- 每个人独立多维档案。
- 人和人不交叉。
- 支持跨平台身份合并。
- 与一级库分离，避免行业数据污染客户档案。

推荐表：

```text
contacts
contact_identities
conversations
messages
customer_profiles
relationship_edges
relationship_styles
contact_preferences
business_facts
tasks
memory_items
```

其中 `memory_items` 是 P0 核心：

```text
memory_items
- id
- contact_id
- identity_id
- memory_type
- scope
- title
- content
- structured_json
- confidence
- evidence_message_ids
- status
- created_at
- updated_at
- expires_at
```

memory_type 建议：

```text
profile_fact        客户基本事实
preference          沟通偏好
relationship_style  关系语气
business_fact       业务事实
commitment          承诺事项
task                待办
risk_note           风险提示
style_rule          风格规则
```

---

## 5. AI 调用流程优化

当前图里的正确流程是：

```text
收到客户消息
  -> 判断意图
  -> 查一级库
  -> 查二级库
  -> 交叉分析
  -> 生成回复
  -> 写回记忆/任务/摘要
```

优化后的实际实现：

```text
1. Message Ingest
   写 messages，去重。

2. Intent Router
   判断是行情、报价、关系沟通、文件解析、命令、闲聊、投诉。

3. Memory Pack Builder
   拉取：
   - 最近 N 条聊天
   - contact profile
   - relationship_style
   - contact_preferences
   - active tasks
   - related business_facts

4. Industry Retrieval
   根据消息命中行业模块：
   - 硫磺
   - 磷矿
   - 铜矿
   - 石膏粉
   - 能源/煤炭
   - 白糖/其他

5. Agent Call
   调 Hermes/Codex/MCP。

6. Risk Review
   检查身份暴露、内部词、过度承诺、非授权价格、敏感内容。

7. Outbox
   自动发送或草稿。

8. Memory Writer
   把新偏好、新承诺、新业务事实写入 memory_items/tasks。
```

---

## 6. 两条微信管道拆开

根据当前汇总图，最优方案是明确拆成两条管道：

### 管道 A：若水号，AI 全自动

```text
19088
  -> 5001
  -> Agent Core 8765
  -> Hermes 8642
  -> outbox auto_send
  -> 5001 /api/ilink/approve-send
  -> 微信发送
  -> Chatwoot 监控展示
```

用途：

- S 级客户。
- 已授权客户。
- 指定群聊 @若水。
- 可自动回复的业务场景。

### 管道 B：宋生号，纯收集

```text
19089
  -> 5011
  -> Agent Core 8765
  -> Unified DB
  -> Chatwoot 展示
  -> 不调 Hermes 自动回复
  -> 不自动发送
```

用途：

- 只采集宋生聊天历史。
- 导入客户关系。
- 补全二级档案库。
- 不以宋生身份自动回复。

这是非常重要的边界。宋生号是“数据采集和关系理解”，若水号是“授权沟通执行”。

---

## 7. Chatwoot 的最优使用方式

Chatwoot 只作为：

```text
CRM 展示层 + 多渠道 inbox + 人工工作台
```

不要让它成为：

```text
行业数据库
客户深度记忆库
AI 主动回复决策器
微信发送器
```

Chatwoot 可直接使用：

- contacts
- conversations
- messages
- labels
- teams
- inboxes
- custom_attributes
- webhooks

Chatwoot custom_attributes 只同步轻量字段：

```text
contact_id
customer_level
industry_tags
last_intent
owner_identity_id
risk_level
memory_summary_short
```

不要把完整档案塞进 Chatwoot。

---

## 8. 相比当前方案的关键优化

### 8.1 从“一个 DB”改为“三层数据”

```text
Raw Ledger       原始消息账本，不可丢
Memory DB        客户档案和长期记忆
Industry DB      行业事实和行情库
```

这样避免行业数据、客户隐私、聊天流水混在一起。

### 8.2 从“一个 Hermes session”改为“会话隔离 + DB 记忆”

Hermes session 只做工作记忆：

```text
agent:{identity}:{platform}:{account}:{conversation}
```

长期事实全部进 DB。

### 8.3 从“AI 直接回复”改为“Agent Core 出站闸门”

AI 只能生成：

```json
{
  "decision": "reply|silent|needs_human",
  "customer_reply": "...",
  "memory_updates": [],
  "tasks": []
}
```

真实发送必须经过：

```text
outbound_guard -> outbox -> platform dispatcher
```

### 8.4 从“人工补文档”改为“自动沉淀记忆”

每次对话后提取：

- 新客户偏好。
- 新关系。
- 新承诺。
- 新报价。
- 新任务。
- 新业务事实。

写入 `memory_items` 和 `tasks`。

---

## 9. 当前最优 P0/P1 路线

### P0：必须马上做

1. `memory_items` 写入逻辑

```text
Hermes response.memory_updates
  -> validate
  -> merge/dedupe
  -> write memory_items
```

2. `/events/inbound-message` 从 501 改为真实入站

```text
middleware -> core 实时推送
```

3. 两级库 schema 建立

```text
data/industry/*.db
memory_items/tasks/business_facts/customer_profiles
```

4. 宋生号纯收集管道设计落地

```text
19089 -> 5011 -> Core -> DB/Chatwoot
```

### P1：尽快做

1. 行业库每日抓取 cronjob

```text
08:00
14:00
实时事件补充
```

2. Chatwoot access token 和 custom_attributes

3. 人脉关系图

4. outbox 任务队列

5. 图片/文件出站支持

### P2：后续扩展

1. WhatsApp/Facebook/Instagram 接入 Chatwoot 原生渠道。
2. TikTok Adapter 单独评估。
3. CRM/ERP/报价/库存工具接入 MCP。
4. 向量索引和语义检索。

---

## 10. 推荐最终结论

最优方案不是单纯：

```text
Chatwoot + Hermes
```

也不是：

```text
全部自研
```

而是：

```text
Chatwoot 做工作台
Agent Core 做业务中枢
两级数据库做长期事实
Hermes/Codex/MCP 做智能工具
WeChat Middleware 做微信适配器
```

当前最需要补的不是 UI，也不是再换模型，而是：

```text
memory_items 从 0 变成可持续增长
一级行业库从文档变成可查询 DB
宋生号纯收集和若水号自动执行严格拆开
```

只要这三点完成，若水系统就从“能自动聊天”升级为“能长期经营客户和业务”。

