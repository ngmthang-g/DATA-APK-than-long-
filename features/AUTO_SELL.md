# Feature — Auto Sell

Status: **EXACT MOBILE SELL MECHANISM SOLVED STATIC; vendor live proof pending**

## Canonical trigger

Do **not** periodically open the bag UI. Use semantic state:

```text
FreeBagSpace = Game.GetFreeBagSpace()
if FreeBagSpace <= configured SellThreshold:
    transition TRAINING -> SELL
```

Stock loot code itself uses `GetFreeBagSpace()` and disables pickup when space is insufficient. Many stock pickup paths preserve at least one free slot, so the tool should expose a configurable threshold rather than wait for exact zero.

Canonical trigger evidence: `analysis/22_LOOT_BAG_FULL_AND_SELL_TRIGGER.md`.

## Scan

```text
Bag site = 10
Game.GetFreeBagSpace()
Game.GetItemsAtSite(10)
```

Keep `DBItemData.ID` (live instance), `ItemID` (template) and `Position` distinct.

Before classifying sell candidates, reserve:

- configured HP/MP recovery medicine;
- explicit user keep list;
- quest range protected by stock client;
- any additional protected categories required by profile.

## Stock guards

```text
quest range 40000000 <= ItemID < 50000000 -> reject
Game.IsItemSellable(ItemID) must be true
```

## Transaction ownership

Before leaving the train spot:

```text
save TrainMap/TrainPosition
stop/yield Train
suspend pickup ownership
invalidate any current combat target/action
```

Do not let stock pickup/combat compete with vendor navigation or sell actions.

## Shop

Route semantically with configured NPC identity:

```text
GetNPCPosition -> GoTo -> ClickNPC
```

If a GameDialog appears, select the current shop/service option semantically. Wait current `CMD_NPC_SHOP_DATA=200034` and require `IsGuildShop=false`. Read current `NpcShopID` and `ID` from current shop data.

## Exact sell

```text
CMD_NPC_SHOP_SELL_REQUEST = 200036
payload = itemInstanceID:NpcShopID:ShopID
```

`itemInstanceID` is `DBItemData.ID`.

## Correct inner loop

```text
fresh bag
 -> choose ONE current sell candidate
 -> current normal shop guard
 -> one sell request
 -> RemoveItem / UpdateItemsList / fresh bag proof
 -> rescan
```

Stop when either:

```text
no configured sell candidate remains
OR
FreeBagSpace >= configured ReturnFreeSpaceTarget
```

Do not cache a list of 90 slots/items and send against stale IDs.

## Return to Train

```text
close/leave current shop as appropriate
 -> GoTo(saved TrainMap,saved TrainPosition)
 -> fresh IsMapReady + MapID + position tolerance proof
 -> restore pickup policy
 -> StartAutoFight(C_AutoModel.Train)
```

If death or a higher-priority survival condition occurs during the transaction, hand ownership to Revive/Recovery and resume the interrupted transaction only after fresh state reacquisition.

Canonical: `analysis/15_INVENTORY_NPCSHOP_AUTO_SELL.md`, `analysis/22_LOOT_BAG_FULL_AND_SELL_TRIGGER.md`.
