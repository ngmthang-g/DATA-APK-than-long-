# Context Pack — Build Tool Core / Multi-LD EXE

## Read first

- `AI_BOOTSTRAP.md`
- `AUTO_TOOL_SCOPE.md`
- `analysis/05_LD9_HOST_GUEST_ARCHITECTURE.md`
- `features/AUTO_ORCHESTRATOR.md`
- `database/RUNTIME_SNAPSHOT_SCHEMA.md`

## Goal

Build the Windows host that discovers LDPlayer instances, creates one BotSession per game client and communicates with guest-side semantic scanner/action bridge.

## Required invariants

- one BotSession per Android game PID;
- independent settings and action gate;
- guest returns semantic state, not raw pointers for host caching;
- process/map generation invalidates guest caches;
- no action without proof/timeout contract.

## Do not read unless needed

Inventory/network deep docs are unnecessary for basic instance manager/scanner plumbing.
