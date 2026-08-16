# Feature — Auto Orchestrator for each LD9

Status: **DESIGN, backed by solved mobile semantic actions**

Each LD9 has an independent state machine and action gate.

Suggested priority:

```text
Manual/Safety
 > Revive
 > map-transition completion
 > critical HP/MP Recovery
 > NPC Treatment
 > current Sell/NPC transaction
 > Chat/Ping when requested
 > Train/Pickup
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

## Integrated transitions

```text
TRAINING
  dead
    -> REVIVE
    -> RETURN_SPOT
    -> TRAINING

  HP/MP below configured threshold
    -> RECOVERY
    -> one live-item Use action
    -> proof/rescan
    -> TRAINING

  FreeBagSpace <= SellThreshold
    -> stop/yield Train + pickup
    -> SELL_ROUTE
    -> OPEN_SHOP
    -> SELL_ONE / PROOF / RESCAN loop
    -> RETURN_SPOT
    -> restore pickup policy
    -> TRAINING

  treatment condition
    -> stop/yield Train
    -> HEAL_ROUTE
    -> DIALOG_MATCH
    -> service/result proof
    -> RETURN_SPOT
    -> TRAINING

  scheduled/user/event chat
    -> SEND_CHAT
    -> return current stable owner state
```

## Ownership conflict rules

- Train target/chase/skill must not compete with Sell/Heal/Revive navigation.
- Pickup must be suspended before vendor/treatment routing.
- If external Recovery is enabled, do not let a separate stock recovery loop independently use the same medicine.
- Chat is short but should not be injected during a fragile map/shop/dialog transition unless the message is explicitly higher priority.
- Bulk Destroy is never a background housekeeping action; it requires an explicit policy and fresh candidate list.

## World generation

After death/revive, map transition, reconnect, shop/dialog destruction/recreation, invalidate guest object references and reacquire semantic roots. Use `WorldGeneration`/snapshot version so an action decided from the previous world cannot run in the new one.

Do not share guest pointers, current target, bag instance IDs, shop data or dialog state across LD instances.

Canonical: `analysis/19_LD9_ACTION_ORCHESTRATION.md`.
