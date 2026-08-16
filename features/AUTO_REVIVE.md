# Feature — Auto Revive / Đầu thai / quay lại bãi

Status: **EXACT MOBILE REQUEST + BUILT-IN COMEBACK DONOR SOLVED STATIC; runtime proof pending**

## Normal revive

```text
CMD_REVIVE_DATA = 200063
C_RevivalType.Normal = 1
payload = "1"
```

Newbie=`"2"`, skill revive=`"3"`.

## Stock donor

`AutoFight_Main:DeathActive()` records death map/position, optionally sends normal revive and enables `IsComeBackTrain`. `ComeBackTrainData()` returns via `Game.GoTo`, with special handling for infernal Map 87.

## Recommended external state proof

```text
DEAD
 -> one revive request
 -> alive/HP/Revival proof
 -> IsMapReady
 -> return saved TrainMap/TrainPos
 -> position tolerance
 -> resume Train
```

Do not clone the stock fixed waits as success proof.

Canonical: `analysis/14_REVIVE_RETURN_MAP_ENGINE.md`.
