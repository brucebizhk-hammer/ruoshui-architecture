# Project A Node 7 Result

Updated: 2026-07-12
Status: Core-side evidence review and explicit Lead storage adapter complete

## Delivered

- `ruoshui.core_evidence_review.v1` result contract.
- Strict acceptance of successful `ruoshui.public_evidence_pipeline.v1` payloads only.
- Pipeline authority, public URL, minimum text, and relevance score checks.
- Default preview mode and explicit `preview / approve / reject` decisions.
- Transactional apply limited to Lead topic, evidence, candidate, identity, and evidence-link tables.
- Rollback on failure and duplicate evidence protection.

## Verification

- Node 7 adapter checks: 8/8 passed.
- Preview and reject paths produced zero writes.
- Approve path wrote evidence and candidate into a temporary database copy only.
- Duplicate apply did not duplicate evidence.
- Invalid authority declaration was rejected.
- Pipeline and continuous-collector safety regressions passed.

## Safety

- Formal database changes from Node 7 tests: no
- Customer/profile/action-request changes: no
- Real WeChat actions: no
- Adapter authority: `core_explicit_lead_tables_only_no_customer_no_wechat`

## WeChat Summary

Project A 节点7完成：Core 侧候选证据审核与显式入库适配器已建立。默认只预览，只有 decision=approve 且 apply=true 才事务性写入 Lead 专用表；拒绝、重复、伪造权限和低质量证据均受控。专项检查 8/8，上游与连续采集回归通过。正式库 0 变化，无客户表或微信动作。下一节点：增加受控 CLI/API 入口并执行一条真实公开页面端到端预览。
