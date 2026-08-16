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

`TopIcon:AutoTrainClick()` calls exactly that service method. `TopIcon:AutoStopClick()` calls:

```text
AutoFight_Main:StartAutoFight(C_AutoModel.None)
```

This is a clean semantic donor for start/stop; the visible settings tab is not the start action.

## Start/stop side effects and mode guard

When Train is requested while current mode is Quest, stock source refuses to start and shows a message telling the user to stop Auto Quest first. It does **not** silently replace the Quest state.

Accepted Train start:

```text
CurrentAutoMode = Train
Game.EnableAutoF1 = false
Game.AutoRemoveFlag()
AutoTrainStart()
```

Stop/None:

```text
CurrentAutoMode = None
Game.AutoRemoveFlag()
```

The running coroutine observes `CurrentAutoMode` and exits/yields according to mode. An external orchestrator should read/own mode state and avoid repeatedly issuing Train start while another mutually exclusive stock mode is active.

## `StopAllCurrentTask()` caveat

Stock `StopAllCurrentTask()` is not an immediate low-level cancellation primitive. It clears quest target state, sets `EnableAutoF1=true`, waits 0.5 s, finds `TopIcon` and calls `AutoStopClick()`. The external tool should prefer the direct semantic `StartAutoFight(None)`/action-gate transition rather than copy this delayed UI-route wrapper.

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

## Loot and survival integration

The same coroutine family also calls pickup and recovery routines. This means external Sell/Recovery ownership must be coordinated with the stock Train service rather than layered as independent mutation loops.

See:

- `analysis/20_ITEM_ACTIONS_AND_AUTO_RECOVERY.md`
- `analysis/22_LOOT_BAG_FULL_AND_SELL_TRIGGER.md`
- `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md`

## Recommendation for external EXE

Prefer invoking the shipped semantic Train engine initially. Let the Windows/LD9 orchestrator own cross-feature transitions such as Revive/Recovery/Sell/Heal/spot switching, while built-in Train owns target/chase/skill/loot when active.

Do not run a second competing combat loop concurrently, and do not use `StartTrain` itself as a polling action; it is a mode transition.
