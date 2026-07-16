# 若水 Lead 数据采集极致版方案

更新时间：2026-07-07

## 0. 结论

前期获客不要先做聊天，先把数据采集、证据留存、线索分层和持续补资料做到足够强。

建议主线：

```text
阶段主题
-> 多源搜索与采集
-> 原始证据库
-> Lead 预备库
-> 补资料/交叉验证
-> 评分与分层
-> 待添加库
-> 自动加微信队列
-> C 类自然补档
```

核心原则：

- 先广泛收集，不怕脏。
- 所有线索必须保留来源、截图/正文/时间和提取证据。
- 微信号是第一阶段主键；电话、公司名、姓名、平台账号都是辅助线索。
- 重复不是问题，重复审核和重复加微信才是问题。
- 审核程序优先于人工逐条审核；人工主要调主题、调权重、抽样纠错。
- 采集系统只产生候选和建议，不直接触发客户承诺。

## 1. 成熟工具选型

### 1.1 基础静态网页采集：Scrapy

用途：

- 行业网站。
- 企业黄页。
- 招采公告。
- 新闻资讯。
- 公开供需页面。
- 可批量翻页、结构相对稳定的网站。

原因：

- Scrapy 是成熟 Python 爬虫框架，适合高并发、可扩展、可配置 pipeline。
- 若水 Core 当前主要是 Python，接入成本低。

官方参考：

- Scrapy docs: https://docs.scrapy.org/
- Scrapy GitHub: https://github.com/scrapy/scrapy

### 1.2 动态页面与复杂站点：Crawlee + Playwright

用途：

- 需要 JS 渲染的网站。
- 需要点击、滚动、分页加载的网站。
- 搜索结果页。
- 平台型页面。
- 需要浏览器上下文的内容。

原因：

- Crawlee 提供爬虫队列、去重、存储、代理、浏览器爬虫能力。
- Playwright 适合真实浏览器自动化、截图、DOM 抽取。

官方参考：

- Crawlee docs: https://crawlee.dev/
- Crawlee GitHub: https://github.com/apify/crawlee
- Playwright docs: https://playwright.dev/
- Playwright GitHub: https://github.com/microsoft/playwright

### 1.3 LLM-ready 页面抽取：Firecrawl 可选接入

用途：

- 快速把网页转成 Markdown。
- 低成本验证网页抽取质量。
- 给 Agent 做网页理解。
- 对单页、站点地图、整站抓取做快速实验。

原因：

- Firecrawl 提供 search / scrape / crawl / map 等能力，适合作为早期加速器。
- 不建议作为唯一底座；正式证据库仍保留原始 URL、原文、截图和本地抽取结果。

官方参考：

- Firecrawl docs: https://docs.firecrawl.dev/
- Firecrawl GitHub: https://github.com/firecrawl/firecrawl

## 2. 数据源矩阵

### 2.1 搜索引擎层

目标：

- 找公司。
- 找联系人。
- 找电话/微信。
- 找供需信息。
- 找招采/项目。
- 找行业文章。

搜索主题按品类拆：

```text
硫磺
磷矿
石膏
能源
固废
化工贸易
物流/港口/仓储
资金/供应链金融
```

硫磺优先关键词：

```text
硫磺 供应
硫磺 采购
硫磺 贸易
硫磺 进口
硫磺 港口
硫磺 现货
硫磺 颗粒
硫磺 粉
硫磺 中间人
硫磺 资源
硫磺 报价
硫磺 询盘
```

组合词：

```text
品种 + 地区
品种 + 港口
品种 + 供应
品种 + 采购
品种 + 电话
品种 + 微信
品种 + 公司
品种 + 招标
品种 + 现货
品种 + 贸易商
```

### 2.2 B2B/黄页/企业平台

采集对象：

- 公司名称。
- 主营产品。
- 联系人。
- 电话。
- 地址。
- 经营范围。
- 网站/店铺链接。
- 发布时间和更新时间。

处理方式：

- 先入原始证据库。
- 提取结构化字段。
- 按品类关键词评分。
- 有联系方式且品类命中高的进入 Lead 预备库。

### 2.3 招采/公告/项目

采集对象：

- 采购方。
- 中标方。
- 产品。
- 数量。
- 地区。
- 联系方式。
- 招标代理。
- 发布时间。

价值：

- 判断真实需求。
- 找买方/终端。
- 找供应商和服务商。
- 反查上下游关系。

### 2.4 新闻/公众号/行业媒体

采集对象：

- 公司动态。
- 项目投产。
- 检修。
- 价格行情。
- 港口库存。
- 政策变化。
- 行业会议。
- 人物和企业关系。

处理方式：

- 公众号文章不直接进客户库。
- 文章进入专业知识库。
- 文中出现公司/联系人/供需/招采信息时，抽取为 Lead 线索，并保留文章证据。

### 2.5 GitHub/开源与技术素材

采集对象：

- 爬虫框架。
- RAG/记忆框架。
- CRM/Lead 管理项目。
- 微信/私域运营工具。
- 数据清洗、实体解析、去重库。

处理方式：

- 不作为客户线索。
- 作为工程素材和可接入模块候选。
- 进入 `dev_references` 或项目文档，不混入业务 Lead。

### 2.6 地图/本地生活/企业名录

采集对象：

- 企业位置。
- 电话。
- 主营描述。
- 营业状态。
- 地址。
- 周边产业园/港口/化工园区。

价值：

- 补地区。
- 补公司真实性。
- 补电话。
- 判断是否在相关产业带。

### 2.7 微信内历史数据反哺

来源：

- 已解密历史聊天。
- 群聊。
- 公众号。
- 已有客户备注。

用途：

- 训练关键词。
- 发现公司名、联系人、群名。
- 生成外部搜索任务。
- 校准角色识别：中间人、资源方、买方、卖方、物流、资金方。

## 3. 分层数据库设计

### 3.1 raw_web_evidence

保存所有原始证据，不直接判断客户价值。

字段：

```text
id
source_type
source_platform
source_url
source_domain
source_title
query_text
topic_id
fetched_at
published_at
raw_html_path
raw_text
markdown_text
screenshot_path
content_hash
page_status
extract_status
error_message
```

### 3.2 lead_pre_pool

保存程序初筛后的候选。

字段：

```text
id
topic_id
primary_key_type
primary_key_value
wechat_id
phone
person_name
company_name
platform_account
main_products
region
guessed_role
evidence_summary
evidence_ids
confidence_score
business_relevance_score
risk_score
freshness_score
contactability_score
source_quality_score
total_score
status
suggested_action
created_at
last_seen_at
last_scored_at
```

### 3.3 lead_identity_links

解决同一主体多个名字的问题，但不强求一次合并完美。

字段：

```text
id
lead_id
identity_type
identity_value
normalized_value
source_evidence_id
confidence
created_at
```

identity_type：

```text
wechat_id
phone
company_name
person_name
platform_account
website
address
tax_id
```

### 3.4 lead_add_queue

只放可自动加微信的高相关线索。

字段：

```text
id
lead_id
wechat_id
phone
add_method
add_message_template
add_message_text
priority
queue_status
daily_batch_id
risk_bucket
attempt_count
last_attempt_at
next_allowed_at
created_at
```

### 3.5 lead_review_feedback

用于持续校准规则。

字段：

```text
id
lead_id
reviewer
review_decision
wrong_reason
corrected_role
corrected_products
corrected_region
notes
created_at
```

## 4. 采集流水线

### 4.1 Topic Planner

人工只设主题，不逐条审线索。

主题示例：

```text
topic_name: 华东硫磺现货供应商
products: 硫磺
regions: 江苏, 浙江, 上海, 山东
roles: 资源方, 贸易商, 中间人, 物流
target_count_per_day: 200 raw pages / 50 lead candidates / 10 add queue
```

### 4.2 Query Generator

自动生成搜索词：

```text
品种词 * 地区词 * 角色词 * 联系词 * 场景词
```

例：

```text
硫磺 江苏 现货 电话
硫磺 山东 贸易商 微信
硫磺 港口 供应 资源
硫磺 采购 招标 联系人
硫磺 颗粒 供应商
```

### 4.3 Fetcher

分三类执行：

```text
ScrapyFetcher: 静态站点、批量列表页
BrowserFetcher: Playwright/Crawlee 动态站点
LLMReadyFetcher: Firecrawl 可选，快速转 Markdown 和结构化
```

每次采集必须保存：

- URL。
- 查询词。
- 抓取时间。
- 原文。
- 正文摘要。
- 截图或 HTML 文件。
- 抽取版本号。
- 错误记录。

### 4.4 Extractor

抽取字段：

```text
公司名
姓名
电话
微信号
主营品种
地区
角色猜测
供需方向
数量
价格
港口
付款/合同关键词
发布时间
证据句
```

### 4.5 Normalizer

标准化：

- 手机号。
- 座机。
- 微信号。
- 公司后缀。
- 地区。
- 品种同义词。
- 角色同义词。

硫磺同义词：

```text
硫磺
硫黄
颗粒硫磺
粉硫磺
液体硫磺
进口硫磺
港口硫磺
```

### 4.6 Scorer

评分公式：

```text
总分 =
业务相关度 * 0.30
+ 联系方式质量 * 0.20
+ 主体可信度 * 0.15
+ 来源质量 * 0.12
+ 新鲜度 * 0.10
+ 角色价值 * 0.08
+ 地区价值 * 0.05
- 风险分
```

第一阶段阈值建议：

```text
total_score >= 75: 进入待添加库
50 <= total_score < 75: 留在预备库继续补资料
total_score < 50: 保留低价值记录或丢弃，视来源而定
risk_score >= 60: 不进入待添加库
```

### 4.7 Enricher

对中高价值线索继续补资料：

- 公司名反查。
- 电话反查。
- 微信号格式校验。
- 地区补全。
- 官网/黄页/招采交叉验证。
- 是否在历史微信数据中出现。
- 是否已添加过。
- 是否已被拒绝过。

### 4.8 Queue Builder

进入待添加库必须满足：

- 有微信号或可通过电话/平台尝试触达。
- 当前主题命中。
- 证据明确。
- 风险分低。
- 未重复添加。
- 未在 D 类静默名单。

## 5. 去重策略

去重不是为了合并成完美客户，而是防止重复动作。

强去重键：

```text
wechat_id
phone
```

中等去重键：

```text
company_name + region
person_name + company_name
platform_account + source_platform
```

弱去重键：

```text
地址
网站
相似公司名
```

规则：

- 同微信号：绝不重复加。
- 同手机号：默认合并为同一触达候选。
- 同公司不同人：可以保留多个 Lead。
- 同人不同平台：保留多证据，围绕微信号合并。
- 重复发现要增加 `last_seen_at` 和证据，不重走审核。

## 6. 风险和黑名单

直接丢弃或降权：

```text
博彩
贷款
刷单
招商加盟泛广告
明显虚假联系方式
纯 SEO 聚合页
无联系方式且无主体
与主题完全无关
```

进入预备库但不进待添加：

```text
旧信息
主体模糊
只有转载无原始来源
只有公司无联系方式
有联系方式但品种不清
疑似广告中介但未确认业务相关
```

## 7. 调度策略

每日节奏：

```text
早上：生成主题查询词
上午：轻量搜索和静态页采集
下午：动态页和补资料
晚上：评分、去重、入预备库、生成待添加队列
次日：按微信风控限额执行添加
```

队列平衡：

```text
预备库目标库存：待添加库的 5-10 倍
待添加库目标库存：日添加量的 3-7 倍
日添加量：按账号风控逐步增加
```

示例：

```text
日添加 10 人
待添加库维持 30-70 人
预备库维持 300-700 人
```

## 8. 和 Core 的接口

新增接口建议：

```text
POST /lead/topics/upsert
GET  /lead/topics

POST /lead/evidence/ingest
POST /lead/pre-pool/upsert
GET  /lead/pre-pool

POST /lead/score/run
POST /lead/enrich/run
POST /lead/add-queue/build
GET  /lead/add-queue

POST /lead/feedback/upsert
GET  /lead/audit/events
```

所有执行类接口先做 dry-run：

```text
apply=false: 只返回计划
apply=true + confirm=true: 才写库
```

## 9. 开发步骤

### P0：数据底座

目标：先能收、能存、能查、能留证据。

开发内容：

1. 建 `raw_web_evidence`。
2. 建 `lead_topics`。
3. 建 `lead_pre_pool`。
4. 建 `lead_identity_links`。
5. 建 `lead_review_feedback`。
6. 做 CSV/JSON 导入器，先允许人工或外部工具导入采集结果。
7. 做查询和导出接口。

验收：

- 能导入 1000 条原始网页证据。
- 能生成 100 条 Lead 预备库记录。
- 每条 Lead 能回溯证据。

### P1：采集器 MVP

目标：硫磺优先，跑通自动采集。

开发内容：

1. Scrapy 静态采集器。
2. Playwright/Crawlee 动态采集器。
3. Query Generator。
4. Extractor。
5. Normalizer。
6. 基础评分器。

验收：

- 每天自动采集硫磺相关网页 200-500 页。
- 生成 30-100 条 Lead 候选。
- 证据、时间、来源完整。

### P2：评分和补资料

目标：让预备库自动变成待添加库。

开发内容：

1. Lead Scorer。
2. Lead Enricher。
3. 去重器。
4. 风险规则。
5. 待添加库生成器。

验收：

- 预备库中高相关线索可自动进入待添加库。
- 重复微信号/手机号不重复添加。
- 人工抽样能看到评分原因。

### P3：微信添加前置

目标：接入自动加微信，但先不追求大量。

开发内容：

1. `lead_add_queue`。
2. 加好友文案生成。
3. 日限额策略。
4. 风控暂停。
5. 结果回写。

验收：

- 每天按限额均匀执行。
- 失败、拒绝、通过、无响应都有记录。
- 通过后自动建立 C 类档案。

### P4：反馈闭环

目标：让规则越跑越准。

开发内容：

1. 人工抽样复核。
2. 误判记录。
3. 规则权重版本。
4. 品类专用评分规则。
5. 周报和主题建议。

验收：

- 每周能看到不同来源命中率。
- 能调整硫磺/磷矿/石膏等不同品类权重。
- 能自动建议下周采集主题。

## 10. 第一阶段硫磺优先执行标准

优先采：

- 有硫磺供应/采购/贸易/进口/港口/现货描述。
- 有电话或微信。
- 有公司名或联系人。
- 时间较新。
- 地区在港口、化工、磷化工产业链相关区域。
- 出现在招采、B2B、行业文章、历史聊天线索中。

优先角色：

```text
中间人
资源方
贸易商
买方
卖方
物流/港口/仓储
资金/供应链服务
```

暂不重视：

- 生活类。
- 纯百科/科普。
- 无联系方式的旧新闻。
- 无主体转载。
- 与硫磺无直接关系的泛化工信息。

## 11. 监控指标

采集指标：

```text
每天抓取页面数
成功率
有效正文率
联系方式提取率
重复率
新 Lead 数
待添加数
高风险排除数
```

业务指标：

```text
加微信通过率
C 类补档完成率
C->B 申请率
B 类有效沟通率
人工判定误判率
来源平台转化率
```

规则指标：

```text
评分命中率
误判样本数
不同来源质量排行
不同关键词质量排行
不同地区质量排行
```

## 12. 近期落地建议

第一周只做硫磺：

```text
1. 建 Lead 数据表和接口。
2. 做 CSV/JSON 导入器。
3. 做硫磺关键词和评分规则 v1。
4. 用搜索/手工/脚本先导入 200-500 条 raw evidence。
5. 生成 50-100 条 lead_pre_pool。
6. 跑评分，选 10-30 条进入待添加库 dry-run。
```

第二周：

```text
1. 接 Scrapy 静态采集器。
2. 接 Playwright/Crawlee 动态采集器。
3. 做去重和补资料。
4. 形成每日采集报告。
```

第三周：

```text
1. 接自动加微信 dry-run。
2. 小规模真实执行。
3. 通过后建 C 类档案。
4. 接 C 类自然补档 Agent。
```

## 13. 与现有方案的关系

这份方案是 `RUOSHUI_CUSTOMER_DEVELOPMENT_AGENT_PLAN_2026-07-07.md` 的前置数据采集细化版。

当前优先级调整为：

```text
Lead 数据采集极致化
> Lead 评分和预备库
> 待添加库和自动加微信
> C 类聊天补档
> B/A 深度对接
```

原因：

- 没有高质量、可追溯、持续增长的 Lead，后面的聊天 Agent 没有足够对象。
- 采集阶段越完整，后续 C 类补档越自然。
- 采集证据越强，升 B/A 的判断越稳。

