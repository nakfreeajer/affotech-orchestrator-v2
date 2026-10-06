# V1 → V2 Migration

## Principle

This is a strangler migration, not a rewrite-in-place and not a big-bang replacement.

V1 remains the behavioral reference until V2 passes conformance, shadow, and controlled cutover gates.

## V1 reference

Repository:

`nakfreeajer/affotech-agent-orchestrator`

Initial reference head:

`ce8784657b858227e6297fd6f2097f2ae26a2e2d`

The protected V1 task `000103` and its recovery lineage remain V1-only and must not be imported as a V2 live transaction.

## Reuse classes

### Reuse or port with minimal semantic change
- prompt artifact identity/hash verification;
- deterministic parser contracts;
- pure state-machine rules;
- exactly-once task/prompt identity rules;
- repository/worktree guards;
- accepted qualification cases that express reusable behavior.

### Reimplement behind clean interfaces
- Architect browser interaction;
- Executor process/session supervision;
- rollover coordination;
- durable state access;
- relay integration;
- dispatch adapter.

### Leave in V1 compatibility/reference
- incident-specific browser DOM archaeology;
- historical one-off recovery paths;
- legacy transaction-specific exceptions;
- machinery needed only to recover old V1 state.

## Migration stages

### M0 — Architecture bootstrap
Canonical architecture, contracts, migration policy, no production authority.

### M1 — Core state and scheduler
Implement project identity, durable store, normalized event model, priority scheduler, pure workflow transitions, and tests.

### M2 — Relay boundary
Implement provider-independent Relay Client against Artifact Relay Manager. Observation only.

### M3 — Executor boundary
Implement guarded dispatch claim, Codex adapter, result capture, and exactly-once tests using disposable/synthetic tasks only.

### M4 — Architect lifecycle
Implement generation ownership, memory observation, rollover coordinator, snapshot/handover flow, and stale-generation rejection.

### M5 — V1 conformance suite
Port reusable V1 regression cases:
- duplicate prompt/event;
- stale Architect generation;
- prompt hash mismatch;
- wrong repository/task ownership;
- Executor crash/restart;
- rollover while Executor runs;
- result arriving during rollover;
- HUMAN_REQUIRED boundaries.

### M6 — Shadow mode
V1 remains ACTIVE. V2 receives the same normalized observations and records the action it would take. V2 cannot launch.

Compare decisions and investigate differences.

### M7 — Controlled qualification
Synthetic/disposable end-to-end tasks through Drive Relay, Architect lifecycle, Executor, restart, and rollover.

### M8 — One controlled real transaction
Explicit human authorization. V1 prevented from launching the selected transaction. V2 completes exactly one bounded transaction.

### M9 — Cutover
Set V1 to RETIRED_CONTROL and V2 to ACTIVE only after accepted evidence.

## Cutover gates

Before V2 becomes active:
- no in-flight V1 transaction requires migration;
- V2 exactly-once and repository guards pass;
- Relay restart/replay is proven;
- Watcher restart/reconciliation is proven;
- Architect rollover while Executor runs is proven;
- result-during-rollover queue/release is proven;
- stale Architect authority is rejected;
- human STOP/retry behavior is proven;
- V1 and V2 cannot both launch.

## After cutover

Keep V1 repository as:
- reference implementation;
- compatibility oracle;
- regression source;
- incident history.

Do not delete or rewrite V1 history merely because V2 is active.
