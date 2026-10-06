# Development Governance

## Authority chain

```text
Rony
→ Architect
→ Executor
→ Architect independent verification
→ Curator/docs when required
→ Architect verification
→ next human/Architect decision
```

Rony is final human authority.

Architect owns architecture, bounded milestones, review classification, verification, and next-task definition.

Executor owns source/runtime changes, tests, validation, concrete evidence, and Git mutation only when explicitly authorized.

Curator owns canonical documentation updates only when Architect explicitly requests docs synchronization after accepted source work.

## Development transport vs product protocol

Strict machine envelopes, task IDs, prompt identity, browser qualification, and rollover protocols may be product features.

They are not automatically requirements for ordinary human development discussion.

Normal discussion remains conversational.

## Milestone discipline

Prefer one bounded question or repair per milestone.

Initial source/review work should normally be NO COMMIT / NO PUSH until Architect accepts the patch.

After source acceptance, use a separate commit/push closure when appropriate.

## No dual authority

V2 development and qualification must never create a situation where both V1 and V2 can launch real work for the same project.

## Protected legacy work

Do not mutate V1 solely to simplify V2 architecture.

Do not migrate the protected V1 `000103` transaction.

## Documentation

Architecture and contracts in this repository are canonical for V2 intent. They must not claim implementation or production authority that has not been proven.
