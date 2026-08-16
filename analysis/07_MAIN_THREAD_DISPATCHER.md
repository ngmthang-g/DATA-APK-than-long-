# MainThread Dispatcher — Mobile

## Evidence status

- `FGStudio.Engine.Utilities.MainThread` naming: **RECONFIRMED** in mobile metadata.
- Previous mobile metadata parse identified members consistent with PC: singleton instance, `Execute(System.Action)`, `Update`, `DoExecuteWorks`, queue-backed dispatch.
- PC KB independently solved the same class/pattern statically.
- Live external guest Action/delegate proof on LD9: **TARGETED-PROOF**.

## Why it matters

Many Unity/Lua/game actions are unsafe when invoked from an arbitrary worker/native hook thread even if a function pointer/signature is correct.

Production flow should be:

```text
host decision
 -> per-session ActionGate
 -> guest constructs valid managed action/delegate
 -> MainThread.Execute(action)
 -> Unity Update processes action
 -> semantic mutation
 -> state proof
```

## Read-only vs mutable

Read-only scanner can use a safer, separately tested path and immutable snapshots.

Mutable calls such as movement/NPC/skill/network producer should not be fired concurrently from scanner threads.

## Common failure modes to investigate before blaming packet logic

- delegate object collected/unrooted;
- wrong managed signature;
- stale target/UI/shop object after map generation change;
- call occurs on non-Unity thread;
- two state-machine timers issue actions simultaneously;
- action result is assumed after fixed delay and a second action races it.

## Required first proof

Use a harmless callback/action with no gameplay mutation. Confirm it executes and the client remains stable across repeated calls and at least one map transition.

Only then promote movement/combat/revive/sell actions to this path.
