# Feature — Auto Sell

Status: **EXACT MOBILE SELL MECHANISM SOLVED STATIC; vendor live proof pending**

## Scan

```text
Bag site = 10
Game.GetFreeBagSpace()
Game.GetItemsAtSite(10)
```

Keep `DBItemData.ID` (live instance), `ItemID` (template) and `Position` distinct.

## Stock guards

```text
quest range 40000000 <= ItemID < 50000000 -> reject
Game.IsItemSellable(ItemID) must be true
```

## Shop

Wait current `CMD_NPC_SHOP_DATA=200034` and require `IsGuildShop=false`. Read current `NpcShopID` and `ID` from current shop data.

## Exact sell

```text
CMD_NPC_SHOP_SELL_REQUEST = 200036
payload = itemInstanceID:NpcShopID:ShopID
```

## Correct loop

```text
fresh bag -> choose ONE current instance -> sell -> removal/update proof -> fresh bag
```

Stop when no configured sell candidates remain or configured free-space target is reached.

Canonical: `analysis/15_INVENTORY_NPCSHOP_AUTO_SELL.md`.
