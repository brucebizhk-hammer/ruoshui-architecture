# Project A Node 8 Result

Updated: 2026-07-12
Status: Controlled CLI and real public-page end-to-end preview complete

## Delivered

- Core evidence review CLI with preview, approve, reject and explicit apply modes.
- End-to-end preview CLI: Playwright crawler -> evidence pipeline -> Core review.
- Output report persisted to `data/lead_collection/public_evidence_e2e_preview_latest.json`.

## Real Public Preview

- URL: `https://www.sci99.com/monitor-613-0.html`
- HTTP status: 200
- Rendered HTML: 444,946 characters
- Extracted main text: 208 characters
- Phone candidates: 0
- Core review score: 43.18
- Suggested action: `observe`
- Review accepted for candidate evidence: yes
- Applied to formal database: no

## Database Counts

- `raw_web_evidence`: 16 -> 16
- `lead_pre_pool`: 14 -> 14
- `lead_add_queue`: 13 -> 13
- `contacts`: 866 -> 866
- `customer_profiles`: 22 -> 22
- `action_requests`: 19 -> 19

## Safety

- Formal database writes: no
- Customer/profile/action changes: no
- WeChat actions: no

## WeChat Summary

Project A 节点8完成：受控 Core 审核 CLI 和真实公开页面端到端预览已完成。Playwright 抓取卓创硫磺页面返回 HTTP 200，正文抽取 208 字，Core 审核得分 43.18，建议 observe；全程 preview，正式库前后计数完全一致，无客户表或微信动作。下一节点：建立审核批次与候选晋级队列，保持显式 apply 和可回滚。
