# Ruoshui Best Architecture Research Decision

Updated: 2026-07-12

## Objective

Preserve working assets, remove duplicated ownership, and introduce mature components only when they solve a measured problem. The target is one business system with four clear planes: business lifecycle, Core control, capability/data, and operations/experience.

## Project Evidence Reviewed

- Roadmap and current architecture rules.
- Database-first memory design.
- Relationship and memory agent design.
- Lead collection and customer-development plans.
- Auto Lead-to-A project plan.
- Provider/Core/allowlist rule system.
- Old-to-new structure and workload comparison.
- Project A handoff and Node 1-7 results.

## External Primary Sources

- Temporal: durable execution, retries, idempotency, pause and recovery patterns.
- Open Policy Agent: policy and execution separation.
- OpenTelemetry: common metrics, logs and traces semantics.
- OpenLineage: source, run and output lineage.
- Chatwoot: shared inbox and human collaboration.
- Scrapy: mature crawling framework.
- LangGraph and AutoGen: stateful agent orchestration comparison.

## Final Decisions

1. Keep Ruoshui Core as the sole control plane.
2. Keep Unified SQLite/FTS5 as the sole authoritative store until measured bottlenecks justify change.
3. Keep Project A as a business lifecycle application, not a second Core.
4. Keep Providers isolated and contract-driven.
5. Adopt Scrapy, Playwright, Trafilatura, phonenumbers and RapidFuzz for their narrow capabilities.
6. Keep APScheduler and Prometheus now; align future telemetry names with OpenTelemetry.
7. Add evidence lineage fields by borrowing OpenLineage concepts without deploying another service.
8. Borrow Temporal durable-workflow patterns without deploying Temporal at the current scale.
9. Borrow OPA policy-separation patterns while implementing a single Core PolicyContext first.
10. Keep Chatwoot as a replaceable human workspace only.
11. Do not add LangGraph, AutoGen, Dify or another agent framework to the main control path.
12. Do not add vector, graph or search databases without repeatable performance evidence.

## Investment Gates

Every new major component must pass five gates:

1. Measured problem evidence.
2. One unambiguous capability owner.
3. Isolated and reversible trial.
4. Acceptance benchmark and rollback plan.
5. Promotion on demonstrated benefit or deletion after two unsuccessful evaluation cycles.

## Research Coverage Limitation

Primary-source GitHub research and the project document set were available. Jina web reading was temporarily unavailable; Exa, Reddit and Xiaohongshu backends were not configured. Therefore the architecture is grounded in official repositories and existing project evidence, not claimed as an exhaustive survey of every community discussion.
