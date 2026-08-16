# Context Pack — Build MainThread Bridge

## Required

- `analysis/01_IL2CPP_RUNTIME_METADATA.md`
- `analysis/07_MAIN_THREAD_DISPATCHER.md`
- `analysis/05_LD9_HOST_GUEST_ARCHITECTURE.md`
- `research/AUTO_RUNTIME_PROOF_QUEUE.md`

## Goal

Prove a guest-side managed Action/delegate can be safely dispatched through game-owned MainThread semantics.

## First action

Use a harmless callback with no gameplay mutation.

## Pass conditions

- correct delegate signature/lifetime;
- callback runs on expected game/Unity thread;
- repeated calls remain stable;
- map transition does not leave stale cached dispatcher/object state.

Only after this proof should semantic movement/combat/network actions be promoted.
