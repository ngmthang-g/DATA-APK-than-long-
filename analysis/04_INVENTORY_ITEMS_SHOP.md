# Inventory / Items / Shop — Mobile APK

## Strong static bag surface

Verified metadata names:

```text
GetFreeBagSpace
GetItems
GetItemsAtSite
GetItemData
GetItemAtSite
GetTotalItems
CountItem
```

This makes opening the visible bag and counting empty UI cells unnecessary for primary logic.

## Item identity fields

Names present:

```text
ID
ItemID
Site
Position
Bound
Quantity
Creator
DurationLeft
Durability
Properties
Attributes
```

Interpretation must preserve three separate concepts:

```text
ID       likely live/database instance identity
ItemID   template/config identity
Position current slot/index
```

Exact mobile runtime schema should be confirmed by scanner output before sending item mutations.

## Classification helpers

Verified names:

```text
IsItemSellable
GetItemBasePrice
GetItemType
GetEquipType
IsItemThrowable
IsItemSellToShopWithBoundMoney
BasePrice
SellPrice
```

This gives a better keep/sell policy than icon/OCR heuristics.

## Item sites

Metadata includes `ItemSite` plus names:

```text
Bag
Body
Storage
Stall
Trade
Pet
```

Exact numeric enum values on mobile still need dumping/runtime confirmation. Do not import PC's numeric Bag site without confirmation.

## Safe Auto Sell design

```text
fresh bag snapshot
 -> select one candidate using policy
 -> require current vendor/shop readiness
 -> send exactly one sell action
 -> prove instance removed/quantity changed/money+shop state consistent
 -> fresh bag snapshot
 -> repeat
```

Never prepare 90 stale slot clicks. Slots and IDs can change after every mutation.

## Current missing link

Current APK evidence does not yet establish:

- exact outbound sell command ID;
- exact payload format;
- exact current shop data fields required by request;
- exact success response/update event.

PC KB has a solved sell path and is an excellent donor/search hint, but those numbers remain `PC-DONOR` here.

## Vendor route

Use semantic NPC routing (`GetNPCPosition`/`GoTo`/`ClickNPC`) and inspect actual shop state. A known human-visible vendor name alone is not proof that a current NPC has the normal sell service in this mobile build.
