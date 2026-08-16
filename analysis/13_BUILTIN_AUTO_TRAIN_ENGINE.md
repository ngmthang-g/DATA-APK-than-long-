# Built-in Auto Train engine — mobile source recovery

Status: **SEMANTIC FLOW VERIFIED STATIC; live bridge invocation still requires runtime safety proof**

## Exact mode and entry point

Recovered `Global_Constants.txt`:

```text
C_AutoModel.None     = 0
C_AutoModel.Train    = 1
C_AutoModel.PK       = 2
C_AutoModel.Quest    = 3
C_AutoModel.AutoPath = 4
C_AutoModel.Fllow    = 5
```

Recovered `AutoFight_Main.lua` confirms:

```text
AutoFight_Main:StartAutoFight(C_AutoModel.Train)
```

Train start sets current mode, disables AutoF1, clears existing auto-flag state and enters `AutoTrainStart()`. Stop/yield is mode `None` plus `Game.AutoRemoveFlag()`.

## Train loop

`AutoTrainStart()` saves the current position as `StartAutoPostion`, clears ignored/banned targets and item-pick state, then runs a coroutine with an approximately 0.3-second loop.

Important guards include:

```text
RoleData.IsDeath
CurrentAutoMode
IsComeBackTrain
IsMovingWithJoyStick
IsRoleBusy / IsLastActionOver
IsProgress
```

When `IsTrainInRanger=true`, the engine uses `RangerScan`; source initialization shows default `RangerScan=500`.

## Target acquisition

`FindBestTarget()` calls `Game.GetNearbySpritesWithPredicate`. The predicate keeps living `Monster` objects, filters ignored targets, enforces radius and optionally checks monster ResID lists/quest filters.

## Combat action flow

```text
PickTrainSkillReady(target.Type)
 -> Game.GetSkillLuaData(skillID)
 -> derive CastRange
 -> distance/path guard
 -> Game.StopAutoPath()
 -> Game.SelectTarget(target.RoleID)
 -> Game.ChaseTarget(...)
 -> success: Game.RequestUsingSkillWithTarget(skillID,target.RoleID)
```

Unreachable/stale targets are moved into `IgnoredTarget`; the engine reloads and rechecks targets rather than trusting stale state.

## Return to center

The engine periodically compares current position with `StartAutoPostion`. If outside `RangerScan`, it calls `Game.MoveToEx(StartX,StartY,success,cancel)` and pauses normal target selection while returning.

## Recommendation for external EXE

Prefer invoking the shipped semantic Train engine initially. Let the Windows/LD9 orchestrator own cross-feature transitions such as Revive/Sell/Heal/spot switching, while built-in Train owns target/chase/skill/loot. Do not run a second competing combat loop concurrently.
