# Orchestrator V2 Architecture

## 1. Purpose

V2 is a deterministic project workflow control plane.

It coordinates durable task artifacts, Architect session lifecycle, Executor ownership, repository integrity, priority handling, and crash recovery without treating browser transcript history as durable workflow state.

## 2. Authority hierarchy

```text
Rony / human authority
        ↓
Architect — architecture and review decisions
        ↓
Orchestrator V2 — workflow authorization and ordering
       ↙ ↘
Artifact Relay   Executor
 transport       execution
        ↓
 durable artifact store
```

Artifact Relay never becomes a second workflow authority.

## 3. Runtime layers

### Workflow Engine

Pure transition logic. Input is durable state plus one normalized event. Output is one decision and bounded effects to perform.

It must not:
- discover browser state;
- call Git;
- launch Executor;
- read Google Drive directly.

### Priority Scheduler

Orders pending events:

- P0 HUMAN_STOP / integrity failure
- P1 ARCHITECT_ROLLOVER
- P2 EXECUTOR_RESULT_READY
- P3 EXECUTOR_PROMPT_READY
- P4 MAINTENANCE / DIAGNOSTIC

Priority chooses the next control-lane action. It does not imply cancellation of already-owned execution.

### State Store

Durably records:
- project identity;
- active task;
- task/prompt claims;
- active Architect generation;
- Executor ownership;
- rollover phase;
- pending event references;
- accepted/rejected event identities;
- recovery checkpoints.

Initial implementation should use a local transactional store such as SQLite. External artifacts remain in the artifact store.

### Repository Guard

Before mutation or dispatch it verifies:
- configured repository identity;
- expected local root;
- branch;
- allowed base/head;
- dirty/protected worktree constraints;
- task ownership.

### Architect Adapter

Owns only interaction with the Architect surface:
- session discovery/opening;
- response completion observation;
- memory/health observation;
- bootstrap/rollover interaction.

It does not define workflow truth.

### Executor Adapter

Owns:
- bounded prompt submission;
- Executor process/session observation;
- terminal result capture;
- local result artifact creation.

It does not choose prompts or authorize retries.

### Relay Client

Consumes normalized events and artifacts from Artifact Relay Manager and publishes outbound artifacts through the same boundary.

The Orchestrator does not embed Google Drive UI/OAuth/folder-selection logic.

## 4. Canonical task lifecycle

```text
DRAFT
  ↓
PROMPT_READY
  ↓
DISPATCH_CLAIMED
  ↓
EXECUTOR_RUNNING
  ↓
RESULT_READY
  ↓
ARCHITECT_REVIEWING
  ├─→ ACCEPTED
  ├─→ BLOCKED
  ├─→ INCONCLUSIVE
  └─→ NO_NEW_REPORT
```

A prompt becomes executable only after an immutable prompt artifact and its manifest are committed READY.

## 5. Exactly-once dispatch

Canonical dispatch identity:

`projectId + taskId + promptSha256`

Before crossing the Executor submission boundary the Orchestrator persists a claim.

Repeated transport events for an already-claimed identity do not cause another launch.

A retry requires explicit retry authority and a new durable retry record; transport redelivery alone is never retry authority.

## 6. Architect generations

Each active Architect session owns an integer generation.

```text
generation N = ACTIVE
rollover
generation N = RETIRED
generation N+1 = ACTIVE
```

After authority switches, late output from a retired generation cannot create new tasks.

## 7. Rollover

Rollover controls the Architect lane, not the Executor lane.

Phases:

```text
ACTIVE
→ ROLLOVER_REQUESTED
→ HANDOVER_REQUESTED
→ HANDOVER_READY
→ NEW_ARCHITECT_STARTING
→ NEW_ARCHITECT_READY
→ AUTHORITY_SWITCH
→ ACTIVE
```

While rollover is active:
- a running Executor continues;
- new prompt dispatch is held;
- completed Executor results are durably stored and queued;
- after the fresh Architect becomes active, queued P2 results are released to it.

## 8. Durable handover

Watcher-generated:
- `SYSTEM_SNAPSHOT.json`

Architect-generated:
- `ARCHITECT_HANDOVER.md`

The system snapshot contains mechanical state: repository/head, active task, Executor status, pending events, generation, and rollover reason.

The handover contains reasoning context: accepted decisions, protected boundaries, blocker, and next authorized action.

## 9. Crash recovery

On restart the Orchestrator reconciles:
1. local transactional state;
2. normalized Relay inbox/outbox state;
3. durable task manifests;
4. Executor process/session evidence;
5. repository state.

Browser transcript reconstruction is not the primary recovery mechanism.

## 10. Compatibility boundary

V1 parser, prompt identity, rollover, exactly-once, worktree, and qualification contracts are inputs to V2 conformance tests.

V2 does not inherit incident-specific V1 recovery machinery unless a reusable contract is explicitly identified.
