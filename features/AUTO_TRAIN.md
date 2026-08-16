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

## Loot / bag threshold

The stock Train pickup path uses `Game.GetFreeBagSpace()` directly. It disables pickup when available space is insufficient. Therefore the external orchestrator should observe free bag space continuously from the snapshot and use a configured threshold to yield Train into Auto Sell.

Do not open the bag screen every N minutes just to count cells.

Canonical: `analysis/22_LOOT_BAG_FULL_AND_SELL_TRIGGER.md`.

## Recovery ownership

The shipped Train loop also calls its own HP/MP recovery logic. For an external EXE there are two valid designs:

1. leave stock recovery enabled and only observe/protect configured medicine; or
2. disable stock recovery and let the external `AUTO_RECOVERY` state own semantic item-use actions.

Do **not** run both independently without arbitration because both can issue item mutations while Train is active. If external recovery is chosen, it should preempt Train briefly, issue one item action, prove result, then release ownership.

Stock recovery contains a timestamp weakness documented in `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md`; do not copy that implementation literally.

## Production policy

External EXE should not duplicate a second combat loop while the shipped Train engine is active. The host orchestrator should own transitions such as Revive/Recovery/Sell/Heal/spot switching, then resume Train.

Canonical deep docs:

- `analysis/13_BUILTIN_AUTO_TRAIN_ENGINE.md`
- `analysis/20_ITEM_ACTIONS_AND_AUTO_RECOVERY.md`
- `analysis/22_LOOT_BAG_FULL_AND_SELL_TRIGGER.md`
