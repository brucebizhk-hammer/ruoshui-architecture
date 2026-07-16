# Project A Node 1 Result

Updated: 2026-07-11
Status: Complete, stopped at approval gate A

## Changes

- Provider results expose structured errors, metrics, and top-level cost estimates.
- Public search reports duration, valid/error/duplicate counts, and source domains.
- Duplicate public-search results are removed within each collection batch.
- Continuous collection honors pre-existing stop, pause, and disable signals.
- Continuous collection includes PID locking, stale PID recovery, and a consecutive-failure circuit breaker.
- Added `tools/verify_continuous_public_lead_collection_offline.py` for focused offline safety verification.

## Verification

- Python compilation: passed for all five changed files and the new verification file.
- Public-search provider verification: passed.
- Provider contract verification: passed.
- Extractor and scoring verification: passed.
- Manual import provider verification: passed.
- Auto Lead schema verification against a temporary database copy: passed.
- Focused offline controls: 10/10 passed, including fetch exceptions, database duplicate insertion, protected-table invariants, PID handling, stop/pause/disable behavior, and circuit breaker state.

## Formal Database Counts

Counts before and after verification were identical:

- `raw_web_evidence`: 15 -> 15
- `lead_pre_pool`: 13 -> 13
- `lead_add_queue`: 13 -> 13
- `contacts`: 866 -> 866
- `customer_profiles`: 21 -> 21

All database mutation checks used a temporary copy. No formal database apply occurred.

## Safety Record

- Network access: no
- Formal database writes: no
- WeChat add-friend actions: no
- WeChat message-send actions: no
- Background collector started: no
- GitHub Pages push: no
- iLink report send: no

## Gate A

Approval is required before the first network dry-run using 2-4 sulfur queries.
