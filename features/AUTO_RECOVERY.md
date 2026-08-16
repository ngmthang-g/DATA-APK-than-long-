# Feature — Auto HP/MP recovery

Status: **STATIC CORE SOLVED; runtime item/server timing proof pending**

## Stock semantic flow

Choose template medicine with:

```text
Game.IsHPMedicine(ItemID)
Game.IsMPMedicine(ItemID)
```

At runtime:

```text
HP/MP threshold reached
 -> Game.FindItem in Bag by configured template ItemID
 -> live DBItemData.ID
 -> CMD_ITEM_ACTION 100005
 -> payload 3:itemInstanceID
```

## Recommended external policy

Do not duplicate the stock timestamp weakness. Use one-action/proof semantics and reserve configured medicine in Auto Sell/Drop filters.

Canonical: `analysis/20_ITEM_ACTIONS_AND_AUTO_RECOVERY.md`.
