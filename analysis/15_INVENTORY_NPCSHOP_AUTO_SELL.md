# Inventory, NPCShop and Auto Sell — mobile exact request flow

Status: **CORE SELL MECHANISM VERIFIED STATIC; vendor service still runtime-specific**

## Bag truth

`ItemSite.Bag = 10` is independently recovered from mobile IL2CPP metadata.

Useful APIs:

```text
Game.GetFreeBagSpace()
Game.GetItemsAtSite(10)
Game.GetItemData(dbID)
Game.GetItemAtSite(...)
Game.GetTotalItems()
Game.CountItem(...)
Game.IsItemSellable(itemID)
Game.GetItemType(itemID)
Game.GetEquipType(itemID)
Game.GetItemBasePrice(itemID)
Game.IsItemSellToShopWithBoundMoney(itemID)
```

`DBItemData` separates live instance and template identity:

```text
ID = live DB/item instance ID
ItemID = template ID
Site
Position
Bound
Quantity
Creator
CreateTime
DurationHour
Durability
Properties
Attributes
```

A sell request must use `ID`, not `ItemID` and not bag `Position`.

## Shop readiness

Inbound Lua packet `CMD_NPC_SHOP_DATA = 200034` opens or refreshes `NPCShop` with current `shopData`.

`NPCShop:RefreshData` disables the Sell tab when `shopData.IsGuildShop == true`; otherwise it enables Sell and passes current shop data to the sell tab. A normal Auto Sell must therefore require current non-guild shop data.

## Exact stock validation

Stock sell UI rejects:

```text
40000000 <= ItemID < 50000000
Game.IsItemSellable(ItemID) == false
```

Quick Sell and context-menu Sell converge on the same semantic request.

## Exact sell request

Recovered `NPCShop_SellItemTab:RequestSellItem(dbItemData)` sends:

```text
CMD_NPC_SHOP_SELL_REQUEST = 200036
payload = itemInstanceID:NpcShopID:ShopID
```

Fields are:

```text
dbItemData.ID
CurrentShopData.NpcShopID
CurrentShopData.ID
```

This exact payload is now mobile VERIFIED STATIC.

## Mutation proof

Core event processors include `ProcessRemoveItem`, `ProcessUpdateItem`, `ProcessSwapItems`, `ProcessUpdateItemsList`.

Correct loop:

```text
fresh bag scan
 -> select ONE sell candidate
 -> require current normal shop data
 -> send ONE 200036 request
 -> wait removal/update + fresh bag difference
 -> rescan
```

Do not cache 90 screen slots or send a burst against stale instances.

## Vendor route

Built-in `GoToNPC(mapID,npcID)` performs cross-map travel if needed, calls `GetNPCPosition`, goes to the live NPC position, then calls `ClickNPC`.

PC static Config is a useful donor for candidate identities (for example Lâu Lan Map 5: Ba Nhĩ 328, Mã Kiêu Minh 373), but this APK Interface snapshot does not contain the same full current NPC config dataset. Keep vendor identity/service as PC-DONOR or USER-REPORTED until mobile runtime confirmation.
