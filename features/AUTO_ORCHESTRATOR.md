# Feature — Auto Orchestrator for each LD9

Status: **DESIGN, backed by solved mobile semantic actions**

Each LD9 has an independent state machine and action gate.

Suggested priority:

```text
Manual/Safety
 > Revive
 > map transition
 > critical Heal
 > current Sell/NPC transaction
 > Chat/Ping when requested
 > Train
```

Core cycle:

```text
fresh immutable snapshot
 -> decide one owner feature
 -> guard
 -> one mutable semantic action
 -> observe result
 -> fresh snapshot
```

Example integrated flow:

```text
TRAINING
  dead -> REVIVE -> RETURN_SPOT -> TRAINING
  bag threshold -> SELL_ROUTE -> SELL_LOOP -> RETURN_SPOT -> TRAINING
  heal condition -> HEAL_ROUTE -> DIALOG_SERVICE -> RETURN_SPOT -> TRAINING
  user/schedule chat -> SEND_CHAT -> return current state
```

Do not share guest pointers, current target, shop data or dialog state across LD instances.

Canonical: `analysis/19_LD9_ACTION_ORCHESTRATION.md`.
