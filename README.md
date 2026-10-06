# AFFOTECH Orchestrator V2

Deterministic workflow control plane for Architect ↔ Executor development automation.

## Status

Architecture bootstrap only. This repository has **no production authority yet**.

The existing `nakfreeajer/affotech-agent-orchestrator` repository remains the V1 reference implementation and compatibility oracle until a controlled cutover is explicitly accepted.

Initial V1 reference baseline:

`ce8784657b858227e6297fd6f2097f2ae26a2e2d`

The protected V1 `000103` lineage is not a migration target.

## Design rule

**Architect decides what should be done. Watcher/control plane decides whether and when workflow rules allow it. Executor performs the work. Artifact Relay transports durable artifacts.**

Google Drive is a durable development artifact bus through the separate `artifact-relay-manager`; it is not itself workflow authority.

## Core components

- Workflow Engine — deterministic task-state transitions.
- Priority Scheduler — resolves human stop, rollover, result, prompt, and maintenance events.
- State Store — durable workflow, task claims, Architect generation, and recovery checkpoints.
- Rollover Coordinator — replaces Architect sessions without transferring Executor ownership.
- Architect Adapter — browser/session lifecycle only.
- Executor Adapter — Codex dispatch/process/result integration.
- Relay Client — consumes normalized Artifact Relay events and publishes artifacts.
- Repository Guard — verifies repository, branch, head, and task ownership before mutation.

## Priority classes

1. P0 — human stop / integrity failure.
2. P1 — Architect rollover.
3. P2 — Executor result ready.
4. P3 — Executor prompt ready.
5. P4 — maintenance / diagnostics.

A higher-priority event controls the control lane but does not automatically cancel lower-priority work already running. In particular, Architect rollover must not terminate an Executor task that already owns its execution boundary.

## Non-negotiable invariants

- Only one active workflow controller may authorize real Executor launch for a project.
- A task/prompt identity may be claimed for execution at most once unless explicit human retry authority exists.
- Stale Architect generations cannot authorize new work.
- Repository identity is verified before dispatch.
- Prompt/result artifacts are immutable once committed READY.
- Browser DOM history is observation, not durable workflow authority.
- V1 and V2 must never simultaneously control real execution for the same project.

## Migration approach

V2 is built beside V1 using conformance tests and shadow-mode comparison. No big-bang rewrite and no migration of in-flight V1 transactions.

See:

- `docs/ARCHITECTURE.md`
- `docs/CONTRACTS.md`
- `docs/MIGRATION.md`
- `docs/GOVERNANCE.md`
