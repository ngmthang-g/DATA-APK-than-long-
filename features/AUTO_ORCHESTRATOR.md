# Feature Specification — Multi-LD Orchestrator

Status: **DESIGN — architecture defined, runtime implementation pending**.

## One state machine per LD9

Each emulator/game PID owns a separate state machine.

Suggested states:

```text
IDLE
SCANNING
TRAINING
REVIVE_WAIT
REVIVE_REQUEST
RETURN_TO_SPOT
SELL_PREPARE
GO_TO_VENDOR
OPEN_VENDOR
SELLING
RETURN_FROM_SELL
PAUSED
ERROR_RECOVERY
```

## Action priority

```text
manual/security pause
 > process/disconnect recovery
 > revive
 > map transition/return
 > active sell transaction
 > train combat
 > background metrics
```

## Action Gate

At most one mutable action may be in flight per game PID.

Read-only observers may execute concurrently if their resolver/object lifetime model is proven safe.

## Proof-driven transition

Every transition follows:

```text
fresh state
 -> guard
 -> one action
 -> expected proof
 -> timeout/failure branch
 -> fresh state
```

## Example full cycle

```text
TRAINING
 -> bag threshold reached
 -> SELL_PREPARE
 -> GO_TO_VENDOR
 -> OPEN_VENDOR
 -> SELLING until no candidate
 -> RETURN_FROM_SELL
 -> TRAINING
```

If death occurs anywhere:

```text
current state
 -> invalidate transient shop/target refs
 -> REVIVE flow
 -> RETURN_TO_SPOT
 -> TRAINING
```

## Multi-instance UI

Windows EXE may show a table such as:

```text
LD | Character | Map | HP | BagFree | State | LastProof
```

Start/Pause buttons can target selected instances. Settings remain per-instance unless explicitly copied by the user.

## No shared mutable state

Never share current shop, target, item instance ID, MainThread delegate, raw object pointer, or pending action token across BotSessions.
