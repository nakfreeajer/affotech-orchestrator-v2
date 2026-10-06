# V2 Contracts

## Project identity

Every configured project has a stable `projectId` and repository identity.

A task event whose project identity does not match the active project is rejected before any execution-side effect.

## Prompt identity

A committed prompt artifact records at minimum:

```json
{
  "projectId": "affotech-agent-orchestrator",
  "taskId": "000120",
  "state": "PROMPT_READY",
  "repository": "owner/repository",
  "branch": "main",
  "baseHead": "<sha>",
  "promptSha256": "<sha256>",
  "promptByteLength": 12345,
  "architectGeneration": 14
}
```

The prompt bytes are immutable once `PROMPT_READY` is committed.

## Result identity

A terminal Executor result is a durable artifact with task identity and SHA-256.

Transport may deliver the same result event more than once. The workflow transition must remain idempotent.

## Dispatch claim

The Orchestrator persists a dispatch claim before prompt submission.

Key:

`projectId / taskId / promptSha256`

One claim authorizes at most one normal launch.

## Event contract

Normalized incoming events have stable identity and do not directly authorize mutation.

Minimum logical shape:

```json
{
  "eventId": "<stable transport event id>",
  "projectId": "<project>",
  "eventType": "PROMPT_READY",
  "taskId": "<task>",
  "artifactId": "<provider-independent artifact identity>",
  "artifactSha256": "<sha256>",
  "providerVersion": "<opaque provider version>"
}
```

The Scheduler/Workflow Engine decides what to do with the event.

## Architect authority

Only the active `architectGeneration` may authorize new work.

A stale generation may still be referenced for historical evidence, but cannot create a new executable task.

## Rollover contract

Rollover:
- has higher control-lane priority than result delivery or new prompt dispatch;
- must not implicitly terminate a running Executor;
- must durably preserve pending results;
- must switch generation authority atomically;
- must retire the old generation before releasing queued Architect-facing events.

## Repository guard contract

Before dispatch, verify:
- configured repository identity;
- local root corresponds to that repository;
- branch is allowed;
- base/head contract is satisfied;
- task is not already claimed;
- protected worktree policy is satisfied.

## Human authority

Human STOP is P0 and overrides automatic scheduling.

Human retry is explicit. A duplicate Drive/local event is never interpreted as human retry.

## Dual-authority prohibition

For one project, only one workflow controller may have real-launch authority.

During migration:

```text
V1 ACTIVE + V2 OBSERVE_ONLY
```

At cutover:

```text
V1 RETIRED_CONTROL + V2 ACTIVE
```

No intermediate state may allow both to launch real Executor work.
