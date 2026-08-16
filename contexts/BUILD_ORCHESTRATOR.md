# Context Pack — Build Full Orchestrator

## Required

- `features/AUTO_ORCHESTRATOR.md`
- `analysis/05_LD9_HOST_GUEST_ARCHITECTURE.md`
- `database/RUNTIME_SNAPSHOT_SCHEMA.md`
- `contexts/BUILD_AUTO_TRAIN.md`
- `contexts/BUILD_AUTO_REVIVE.md`
- `contexts/BUILD_AUTO_SELL.md`

Use this pack when work spans Train + Revive + Sell. Do not independently preload every analysis file.

## Core contract

Per PID/session:

```text
Snapshot -> priority decision -> one ActionGate mutation -> proof -> fresh Snapshot
```

Keep feature state independent across LD9 instances.
