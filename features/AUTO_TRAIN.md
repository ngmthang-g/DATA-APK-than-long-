# Feature Specification — Auto Train (Mobile / LD9)

Status: **STATIC-STRONG primitives; high-level built-in Train entrypoint not yet verified on mobile**.

## Required state

Per BotSession:

```text
Alive / IsDeath
MapID
Position
IsMapReady
SelectedTarget
nearby enemy candidates
train origin/map/radius
skill/cooldown state if using low-level combat
current higher-priority feature state
```

Static metadata provides names supporting these requirements, including `GetNearByEnemies`, `GetNearbySpritesWithPredicate`, `SelectedTarget`, `SelectTarget`, `ChaseTarget`, `UseSkill`, `RequestUsingSkill*`, `GetSkillCooldown`, movement/path APIs and map-ready state.

## Preferred architecture

Do not make Windows screen clicks the normal combat engine.

If a verified high-level built-in Train entrypoint is recovered, prefer it when it safely encapsulates target/chase/skill/loot behavior. Otherwise build a narrow semantic loop from exposed Game APIs.

## Low-level semantic loop

```text
fresh snapshot
 -> if dead: yield to REVIVE
 -> if wrong map/outside return tolerance: yield to RETURN_TO_SPOT
 -> if current Sell transaction: yield
 -> choose current reachable enemy
 -> one SelectTarget/Chase/Skill action
 -> proof from fresh target/player state
 -> rescan
```

## Target policy

Do not cache object pointers across map/world generation changes. Keep stable IDs/ResIDs only when their semantics are confirmed.

## Skill policy

Use semantic skill request methods and cooldown state. Do not spam skill requests on a fixed timer when target/death/map/busy state says the previous action is not complete.

## PC donor

PC KB has solved `AutoFight_Main:StartAutoFight(C_AutoModel.Train)`. Current mobile exact-string pass did not confirm standalone `AutoFight_Main`/`StartAutoFight`.

Status remains `PC-DONOR / TARGETED-PROOF`, not mobile fact.

## Completion/health proof

Train is considered active only from semantic state/action evidence, not because the visible Auto button looks enabled.

## Failure handoff

Priority:

```text
manual/security pause
 > death/revive
 > map return
 > current sell transaction
 > normal train
```
