# Project A Node 3 Result

Updated: 2026-07-12
Status: Lightweight mature Provider foundation complete

## Environment

- Isolated runtime: `.venv-provider`
- Python: 3.11.15
- Exact dependency lock: `requirements-provider.lock`
- Main Python 3.14 Core runtime remains unchanged.

## Added Capabilities

- Trafilatura 2.1.0: precision-oriented HTML main-text extraction.
- RapidFuzz 3.14.5: company/name fuzzy matching with an explicit cutoff.
- phonenumbers 9.0.34: validity checks and E.164 normalization.
- APScheduler 3.11.3: single-instance, coalescing background scheduler factory.
- Prometheus Client 0.25.0: isolated Provider run, record, duration, and last-success metrics.

Implementation: `services/lead_collection/mature_tools.py`

## Verification

- New mature-tools checks: 7/7 passed.
- Existing public-search Provider regression: passed.
- Existing continuous-collector offline safety regression: passed.
- Scheduler was configured but deliberately not started by the offline test.
- No database access or network access occurred in the new tool verification.

## Safety Record

- Core authority changed: no
- Main runtime dependencies changed: no
- Formal database writes from Node 3: no
- Customer/profile/outbox changes: no
- WeChat actions: no
- Existing continuous public-search collector remains separate from the new scheduler factory.

## Next Development

Build an isolated enrichment Worker that accepts raw public evidence through a JSON contract and returns extracted text, normalized phone identities, fuzzy identity matches, and metrics without direct database authority. Scrapy and Playwright remain deferred until this Worker contract is stable.
