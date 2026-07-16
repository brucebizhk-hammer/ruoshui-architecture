# Project A Node 5 Result

Updated: 2026-07-12
Status: Scrapy and Playwright isolated crawler foundation complete

## Environment

- Scrapy: 2.17.0
- Playwright: 1.61.0
- Chromium: Playwright build 1228 / Chrome for Testing 149.0.7827.55
- Direct dependency lock: `requirements-crawler.lock`

## Delivered

- `ruoshui.crawler.v1` crawler contract.
- Scrapy-based title, heading, and link extraction.
- Playwright headless dynamic-page rendering.
- Core-supplied domain allowlist enforcement.
- Rejection of credential URLs, non-HTTP protocols, localhost, and private/reserved IP targets.
- Same-host subresource routing and maximum rendered HTML size.
- No repository, SQLite, Core, authorization, or WeChat imports.

## Verification

- Offline parser and target-policy checks: 9/9 passed.
- Enrichment Worker regression: 10/10 passed.
- Controlled Playwright smoke test: `https://example.com/`, HTTP 200, title `Example Domain`.
- Smoke test authority: `public_get_only_no_database_no_core_no_wechat`.

## Database Counts

- `raw_web_evidence`: 16 -> 16
- `lead_pre_pool`: 14 -> 14
- `lead_add_queue`: 13 -> 13
- `contacts`: 866 -> 866
- `customer_profiles`: 22 -> 22
- `action_requests`: 19 -> 19

## WeChat Summary

Project A 节点5完成：Scrapy 2.17 与 Playwright 1.61 已接入隔离 Provider 环境，Chromium 动态渲染可用。新增 ruoshui.crawler.v1 合约、域名白名单和 SSRF 防护；离线安全检查 9/9，通过 example.com 联网渲染测试（HTTP 200）。正式数据库 0 变化，无任何微信动作。下一节点：将 crawler 输出接入 enrichment Worker，并建立受控公开证据流水线。
