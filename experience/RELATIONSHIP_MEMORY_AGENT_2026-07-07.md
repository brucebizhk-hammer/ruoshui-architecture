# Relationship And Memory Agent

Date: 2026-07-07

## Core Correction

The customer system should not depend on heavy upfront manual labeling.

Default customer handling is now relationship-first:

- New or imported sulfur customers enter `C` by default.
- The relationship and memory agent observes real conversations.
- The agent completes customer profiles during natural communication.
- The agent records relationship evidence, role evidence, resource value, risk, and deal progress.
- The agent creates upgrade requests when the business situation needs more autonomy.
- Human approval decides whether the customer moves to `B`, `A`, or `S`.

## Why This Changed

The earlier sulfur-default-`B` model made the database look tidy, but it pushed too much work onto manual review and did not match real sulfur trading.

In sulfur trading, many people are intermediaries. The value is often in upstream information transfer, coordination, introductions, route clarity, and deal progress. The system should learn this through contact, not force full classification before the relationship develops.

## Levels After Correction

| Level | Meaning | Runtime Behavior |
| --- | --- | --- |
| `S` | Fully trusted and explicitly authorized | Free automatic replies. |
| `A` | Trusted, but sensitive facts guarded | Auto for normal chat; person/company/price/internal-profit/source-ownership/customer-channel goes to human confirmation. |
| `B` | Approved materials and non-local-database Q&A | Can answer general materials/knowledge questions after scope detection. |
| `C` | Default relationship-building state | No autonomous business reply. Focus on profile completion, relationship understanding, and upgrade requests. |
| `D` | Excluded / do not engage | Silent. |

## Agent Responsibilities

The relationship and memory agent owns:

- profile completion: name, company, role, main products, region, relationship strength;
- relationship style: tone, preference, personality, trust level, response habits;
- business role detection: middleman, resource side, buyer, seller, logistics, funding, partner;
- deal-progress evidence: real demand, upstream resource, price quote, route, payment condition, contract action;
- risk evidence: unclear identity, overclaiming, sensitive information, contradictory resource chain;
- upgrade request drafting.

## Upgrade Request Standard

The agent may request:

- `C -> B`: the contact needs general material/market/knowledge Q&A and has basic profile evidence.
- `C/B -> A`: the contact is involved in real deal progress but still needs sensitive-field review.
- `A -> S`: only after repeated trusted interaction and explicit human authorization.
- any level -> `D`: spam, unrelated, risky, or not worth engagement.

Each request must include:

- current level;
- requested level;
- reason;
- evidence messages or summarized evidence;
- suggested reply boundary.

## Database Objects

Added:

- `relationship_agent_events`: observations and profile/relationship events.
- `customer_upgrade_requests`: pending upgrade/downgrade requests for human approval.
- `friend_requests`: staged add-friend requests for human review; Core never executes WeChat friend add from this table.
- `action_requests`: staged high-risk action requests for file send, group invite, contract/payment/price commitments, and customer private information.

Existing:

- `customer_profiles`: stores the current profile state.
- `relationship_styles`: stores communication style and relationship summary.
- `memory_items`: stores long-term memory.
- `business_facts`: stores business facts.
- `customer_authorizations`: stores final authorization level and runtime boundary.

## Current Migration

The 20 reviewed sulfur contacts were changed from default `B` to default `C`.

They now have:

- `customer_authorizations.customer_level = C`
- `customer_profiles.customer_level = C`
- `auto_reply_mode = draft`
- one initial `relationship_agent_events` record

This means they are valid business contacts, but not automatically authorized. The next move is for the relationship and memory agent to learn from ongoing communication and request upgrades when useful.

## Runtime Interfaces

Implemented on 2026-07-07:

- `GET /relationship/events`
- `POST /relationship/events/upsert`
- `GET /relationship/upgrade-requests`
- `POST /relationship/upgrade-requests/upsert`
- `POST /relationship/upgrade-requests/decide`
- `GET /relationship/dashboard`
- `GET /relationship/readiness`
- `GET /relationship/profile-patch-audit`
- `GET /relationship/profile-patch/preview`
- `POST /relationship/profile-patch/apply`
- `POST /relationship/profile-gaps/create-tasks`
- `GET /relationship/profile-gap-tasks`
- `POST /relationship/profile-gap-tasks/update`
- `GET /relationship/agent-tick`
- `POST /relationship/agent-tick`
- `GET /friend-requests/list`
- `POST /friend-requests/stage`
- `POST /friend-requests/decide`
- `GET /action-requests/list`
- `POST /action-requests/stage`
- `POST /action-requests/decide`
- `GET /task-events`
- `POST /task-events/upsert`
- `POST /conversation/runtime-state/maintain`
- `GET /chat-bridge/queue`
- `POST /chat-bridge/tick`
- `GET /knowledge/query`

Hermes can now return these arrays in its JSON response:

```json
{
  "relationship_events": [],
  "upgrade_requests": []
}
```

Core writes them through the same memory pipeline as `memory_updates`, `tasks`, and `business_facts`.

Realtime inbound event:

- `POST /events/inbound-message` is implemented.
- It writes inbound middleware messages into Core immediately, with the same contact, conversation, classification, and de-duplication path as polling.
- It defaults to store-only and does not generate or send replies.
- If explicitly called with `respond=true`, it enters the existing `/agent/respond` path with `poll_limit=0`; all authorization, outbox, guard, and auto-send rules still apply.

Conversation runtime state:

- `conversation_runtime_state` is implemented.
- `GET /conversation/runtime-state` shows whether a conversation is `debouncing` or `idle`.
- Consecutive inbound messages increment `pending_inbound_count` and refresh `debounce_until`.
- Outbound messages clear the pending count and return the conversation to `idle`.
- This is the state base for the next debounced/asynchronous reply worker.

Debounced reply worker:

- `POST /conversation/reply-worker/run` is implemented.
- Default mode is dry-run: it only lists due debouncing conversations and the latest inbound message that would be handled.
- With `apply=true`, it calls the existing `/agent/respond` path with `poll_limit=0` and defaults to `mode=draft`.
- It handles only `latest_inbound_message_id`, so old messages in a burst do not each receive separate replies.
- It does not bypass authorization, outbox, guard, or human review policy.
- Each run is written to `conversation_reply_worker_runs`.
- `GET /conversation/reply-worker/runs` exposes worker audit records for troubleshooting and later background scheduling.
- Background scheduling is implemented but disabled by default.
- Enable with `CONVERSATION_REPLY_WORKER_ENABLED=1`; default mode remains `draft`.
- Dashboard/readiness now include conversation runtime state, reply worker runs, and background worker enabled state.

Runtime state maintenance:

- `POST /conversation/runtime-state/maintain` is implemented.
- Default mode is dry-run: it only lists stale runtime states and writes nothing.
- With `apply=true`, it marks stuck states as `stale_runtime_state`, clears pending inbound count, and records maintenance metadata.
- It does not call Hermes, does not generate replies, and does not send WeChat messages.
- This is the cleanup path for conversations that got stuck in `debouncing`, `generating_reply`, `awaiting_outbox`, or `reply_error`.

Knowledge query:

- `GET /knowledge/query` is implemented.
- It is read-only and does not send WeChat messages.
- It searches `industry_facts`, `business_facts`, and `knowledge_articles`.
- It supports `q`, `module`, `category`, `sources`, and `limit`.
- This gives B-level materials Q&A and Hermes replies a Core-side knowledge lookup path.
- Hermes memory pack now includes `knowledge_context`.
- `knowledge_context` is compacted before sending to Hermes: title/subject, content summary, source, confidence, observed time, and expiry only.
- Knowledge context does not change authorization or auto-send behavior.

Task events:

- `GET /task-events` is implemented.
- `POST /task-events/upsert` is implemented.
- Task events are stored as `relationship_agent_events.event_type=task_event`.
- By default, task events only record progress and do not change `tasks.status`.
- If `update_task_status=true` is provided, Core updates the linked task status.
- This gives the relationship/memory workflow a task lifecycle trail: progress, waiting, done, confirmed, blocked.

Friend request staging:

- `GET /friend-requests/list` is implemented.
- `POST /friend-requests/stage` is implemented.
- `POST /friend-requests/decide` is implemented.
- Relationship Agent may stage a request when a customer relationship needs a direct friend connection.
- Duplicate pending requests for the same target are updated instead of creating repeated queue items.
- `dry_run=true` on decide previews the approval result without writing DB.
- Formal approval records the decision and writes a `friend_request_decision` event.
- Even approved requests do not call WeChat middleware and do not execute add-friend; the next action remains manual or a future guarded executor.

High-risk action staging:

- `GET /action-requests/list` is implemented.
- `POST /action-requests/stage` is implemented.
- `POST /action-requests/decide` is implemented.
- This covers `file_send`, `group_invite`, `room_invite`, `price_commitment`, `contract_commitment`, `payment_commitment`, `customer_private_info`, `external_send`, and `manual_action`.
- `dry_run=true` on decide previews the decision without writing DB.
- Formal approval/rejection/cancel only records the decision and writes `relationship_agent_events.event_type=action_request_decision`.
- Even approved action requests do not call WeChat middleware and do not execute external actions.

Chat Bridge Agent proposal:

- The dedicated chat/onboarding agent should sit between WeChat middleware and Core/Hermes.
- It owns conversation continuity: debounce, latest-message selection, reply state, approval state, and stuck-state recovery.
- It calls Hermes only to generate reply text.
- It calls Core for authorization, memory, profile gaps, knowledge context, upgrade requests, task events, friend requests, and high-risk action requests.
- It never lets Hermes directly decide whether to send, add friends, send files, invite groups, or make commitments.
- Current no-reply diagnosis: Core, middleware, and Hermes were online; the Vincentss conversation was in `debouncing`; background reply worker is disabled by default; some historical replies were pending approval or blocked by outbound guard.

Dedicated chat agent operating model:

- WeChat remains the same conversation channel; the agent does not open a separate chat.
- The chat agent first checks Core state: authorization level, debounce state, latest inbound message, pending/stale outbox, profile gaps, knowledge context, and high-risk action boundaries.
- If the conversation is still waiting, it reports the reason.
- If the conversation is due and clean, it may generate a draft through Hermes only after explicit controlled execution.
- If an old draft or pending item blocks the conversation, it may only propose cleanup by default.
- Cleanup execution requires `apply=true`, `confirm=true`, `allow_cleanup_outbox=true`, and a non-empty `cleanup_reason`.
- Draft generation requires `apply=true`, `confirm=true`, and `allow_generate_draft=true`, and still forces `disable_auto_send=true`.
- The send decision remains in Core outbox and human approval rules.

Why Ruoshui may not reply:

- The conversation is in `debouncing` and background reply worker is disabled.
- A draft, pending approval, stale, or failed outbox already exists, so Core avoids duplicate replies.
- A newer inbound message has made an older draft unsafe to send.
- The contact is `C`, `D`, or outside the current `B` scope.
- The conversation belongs to the secondary/store-only account.
- A high-risk request requires staging or human confirmation first.

Chat Bridge preflight implemented:

- `GET /chat-bridge/queue` is available as a read-only queue.
- `POST /chat-bridge/tick` is available as a preflight scheduler by default.
- In default mode, both endpoints explicitly do not write DB, do not call Hermes, and do not send WeChat.
- The queue returns why a conversation is waiting or stuck:
  - still inside debounce window;
  - due for reply-worker dry-run;
  - existing pending/stale outbox needs human review;
  - stale runtime state needs maintenance dry-run;
  - secondary/store-only account should be observed only.
- Controlled draft generation is now available behind three locks:
  - `apply=true`;
  - `confirm=true`;
  - `allow_generate_draft=true`.
- Controlled draft mode calls the existing reply worker with `mode=draft` and `disable_auto_send=true`.
- It may call Hermes and create/update outbox through the normal Core path, but it never auto-sends WeChat.
- Controlled outbox cleanup is now available behind four locks:
  - `apply=true`;
  - `confirm=true`;
  - `allow_cleanup_outbox=true`;
  - `cleanup_reason` is non-empty.
- Controlled cleanup cancels only draft/pending/stale/failed outbox items in the selected conversation.
- Controlled cleanup does not call Hermes, does not generate new replies, and does not send WeChat.
- This is the first controlled execution layer of the dedicated chat/onboarding agent. The next step is a limited live smoke test for Vincentss and test groups only.

Approval rule:

- The agent may create a pending upgrade request.
- Only human approval through `/relationship/upgrade-requests/decide` changes `customer_authorizations`.
- Rejection closes the request without changing the customer level.

Smoke test result:

- A profile observation event was written for contact `625`.
- A test `C -> B` upgrade request was created and then rejected.
- No customer level was changed by the test.

## C-Level Profile Gap Injection

Implemented on 2026-07-07:

- Core loads `customer_profiles.profile_json` with inbound messages.
- Hermes memory pack includes `contact_profile.profile`.
- Hermes memory pack includes `contact_profile.profile_gap`.
- Hermes memory pack includes `knowledge_context` from local Core knowledge query.
- `profile_gap` includes:
  - authorization level;
  - missing profile fields;
  - missing count;
  - suggested natural questions.
- Live queue API: `GET /relationship/profile-gaps`.
- Dashboard API: `GET /relationship/dashboard`, combining profile gaps, pending upgrade requests, recent relationship events, and next actions.
- Dashboard now also exposes the full relationship workflow routes: profile patch preview, profile patch audit, gap task create/query/update, relationship events, and upgrade decision.
- Readiness API: `GET /relationship/readiness`, checking whether the relationship workflow is operational and surfacing warnings without writing DB or sending WeChat.
- Profile patch audit API: `GET /relationship/profile-patch-audit`, showing applied fields, ignored reasons, and auto-created upgrade requests.
- Profile patch preview API: `GET /relationship/profile-patch/preview`, dry-running a proposed patch and C->B eligibility without writing DB or sending WeChat.
- Profile patch apply API: `POST /relationship/profile-patch/apply`, default dry-run; with `confirm=true` it writes a relationship event and reuses the existing audit/upgrade-request pipeline without sending WeChat.
- Profile gap task API: `POST /relationship/profile-gaps/create-tasks`, default dry-run; with `apply=true` it creates internal tasks and `profile_gap_task_created` events without sending WeChat messages.
- Profile gap task query API: `GET /relationship/profile-gap-tasks`, listing generated internal gap tasks with the latest task event and current profile gap.
- Profile gap task update API: `POST /relationship/profile-gap-tasks/update`, changing profile-gap task status and writing a `profile_gap_task_updated` audit event; it does not send WeChat messages.
- The update API supports `dry_run=true` / `preview=true`: it returns the proposed status change, profile patch preview, ignored fields, and possible pending C->B upgrade preview without writing DB.
- Profile gap task update can now carry a confirmed `profile_patch`. When `confirm_profile_patch=true`, Core applies only whitelisted empty profile fields, writes `profile_patch_audit`, and may create a pending `C -> B` upgrade request. It still does not change authorization level, call Hermes, or send WeChat.
- Relationship Agent tick API: `GET /relationship/agent-tick` and `POST /relationship/agent-tick`.
- Agent tick default mode is dry-run: it plans C-level profile-gap tasks, skips contacts that already have open/in-progress profile-gap tasks, does not call Hermes, does not write DB, and does not send WeChat.
- Agent tick apply mode requires both `apply=true` and `confirm=true`; it creates only internal profile-gap tasks plus `profile_gap_task_created` audit events.
- Friend request staging API: `POST /friend-requests/stage`, creating or updating a pending add-friend request without sending WeChat and without executing friend add.
- Friend request decision API: `POST /friend-requests/decide`, default preview supported through `dry_run=true`; formal decisions only update DB and write a relationship event.
- High-risk action staging API: `POST /action-requests/stage`, creating or updating pending file-send/group-invite/commitment/private-info actions without external execution.
- High-risk action decision API: `POST /action-requests/decide`, default preview supported through `dry_run=true`; formal decisions only update DB and write a relationship event.
- Relationship events can carry `event.profile_patch`.
- Core only applies whitelisted profile fields into empty/unknown slots:
  - display name;
  - company name/type;
  - business role;
  - main products;
  - region;
  - relationship strength;
  - auto reply permission.
- `profile_patch` cannot change S/A/B/C/D authorization level; upgrades still require `upgrade_requests` and human decision.
- If a C-level sulfur profile has enough role/product/region evidence after patching, Core can auto-create a pending `C -> B` upgrade request.
- Auto-created upgrade requests do not change authorization by themselves; only `/relationship/upgrade-requests/decide` with human approval changes the level.
- Approval supports `dry_run=true` so the operator can preview the level/mode/topic changes without writing the database.

Rule:

- Ask at most one profile-completion question.
- Do not interrogate the customer.
- Do not ask profile questions when the latest message needs a direct business answer.
- C-level remains pending approval and does not auto-send.

Verification:

```text
python tools/verify_relationship_workflow_suite.py -> ok=true
python tools/verify_inbound_message_event.py -> ok=true
python tools/verify_conversation_runtime_state.py -> ok=true
python tools/verify_conversation_runtime_maintenance.py -> ok=true
python tools/verify_conversation_reply_worker.py -> ok=true
python tools/verify_background_reply_worker.py -> ok=true
python tools/verify_knowledge_query_api.py -> ok=true
python tools/verify_knowledge_context_pack.py -> ok=true
python tools/verify_task_events_api.py -> ok=true
python tools/verify_friend_request_staging.py -> ok=true
python tools/verify_action_request_staging.py -> ok=true
python tools/verify_c_profile_gap_pack.py -> ok=true
python tools/verify_profile_gaps_api.py -> ok=true
python tools/verify_profile_patch_pipeline.py -> ok=true
python tools/verify_profile_patch_preview.py -> ok=true
python tools/verify_profile_patch_apply.py -> ok=true
python tools/verify_profile_patch_apply_api_dry_run.py -> ok=true
python tools/verify_upgrade_decision_dry_run.py -> ok=true
python tools/verify_relationship_dashboard_api.py -> ok=true
python tools/verify_relationship_readiness_api.py -> ok=true
python tools/verify_profile_patch_audit_api.py -> ok=true
python tools/verify_relationship_agent_tick.py -> ok=true
python tools/verify_profile_gap_task_creation.py -> ok=true
python tools/verify_profile_gap_tasks_api.py -> ok=true
python tools/verify_profile_gap_task_update.py -> ok=true
python tools/verify_profile_gap_task_update_dry_run.py -> ok=true
python tools/verify_profile_gap_task_patch_upgrade.py -> ok=true
```

Suite result:

```text
python tools/verify_relationship_workflow_suite.py -> ok=true, 30/30 passed
```

Latest full suite:

```text
python tools/verify_relationship_workflow_suite.py -> ok=true, 30/30 passed
```

Latest suite report files:

```text
data/relationship_workflow_suite_latest.json
data/relationship_workflow_suite_latest.md
```

## 2026-07-07 Live Task Queue Update

- Created internal profile-gap tasks for all 20 current C-level sulfur customers.
- First batch: task IDs 6-15.
- Second batch: 10 more tasks created by `POST /relationship/agent-tick` with `apply=true` and `confirm=true`.
- Current open profile-gap tasks: 20.
- Policy stayed internal-only: no Hermes call, no WeChat send, no authorization change.

## 2026-07-07 Profile Gap Task To Upgrade Loop

The profile completion loop is now end-to-end:

1. Relationship Agent creates an internal C-level profile-gap task.
2. During later communication, the task can be updated with a confirmed `profile_patch`.
3. Core applies only allowed empty fields such as company, role, products, region, and relationship strength.
4. Forbidden fields such as `authorization_level` are ignored.
5. If the C-level sulfur profile now has role, sulfur product, and region evidence, Core creates a pending `C -> B` upgrade request.
6. The customer level still changes only after human approval through `/relationship/upgrade-requests/decide`.

Verification:

```text
python tools/verify_profile_gap_task_update_dry_run.py -> ok=true
python tools/verify_profile_gap_task_patch_upgrade.py -> ok=true
python tools/verify_relationship_workflow_suite.py -> ok=true, 30/30 passed
```
