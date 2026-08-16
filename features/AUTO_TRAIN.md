# Feature — Auto Train / Đánh quái

Status: **MOBILE SEMANTIC ENGINE SOLVED STATIC; live action safety proof pending**

## Canonical start/stop

```text
C_AutoModel.Train = 1
GUI.FindUI("AutoFight_Main"):StartAutoFight(C_AutoModel.Train)
```

Yield/stop through mode None rather than clicking the visible settings tab.

## Built-in engine owns

- train center and `RangerScan`;
- monster scan with `GetNearbySpritesWithPredicate`;
- death/ignored target filters;
- optional monster whitelist;
- path/range checks;
- target select/chase;
- skill selection/cast;
- loot integration;
- return-to-center;
- recovery from stale/unreachable targets;
- optional death comeback donor.

## Production policy

External EXE should not duplicate a second combat loop while the shipped Train engine is active. The host orchestrator should own transitions such as Revive/Sell/Heal/spot switching, then resume Train.

Canonical deep doc: `analysis/13_BUILTIN_AUTO_TRAIN_ENGINE.md`.
