# Loot, bag-full detection and Auto Sell trigger — mobile

Status: **EXACT STATIC SOURCE RECOVERED**

## Built-in loot path

`AutoFight_Main:DoPickItem()` scans ground loot using:

```text
Game.GetNearbyItemPack(predicate,...)
```

It filters type `ItemPack`, already-attempted RoleIDs, range and reachability, then uses:

```text
Game.MoveToEx(...)
Game.ClickToObject(itemPack.RoleID)
```

When the item-pack payload arrives, `RevicePackItemData(itemPackID, items)` decides what to take.

Exact metadata APIs:

```text
GetNearbyItemPack(...) token 0x060007B5
PickUpItemFromItemPack(itemPackID,slotIndex,UsingAuto) token 0x0600071B
```

## Bag-full state is semantic

The stock auto never needs to open the bag UI to count cells. It repeatedly uses:

```text
Game.GetFreeBagSpace()
```

When the pack will not fit or a filtered pickup reaches the reserve threshold, source displays `Túi đồ đã đầy không thể nhặt`, clears `IsPickItemProsecc` and sets:

```text
AutoFightSettings["PICKITEM"]["IsOn"] = false
```

Therefore an external orchestrator can use **free bag space itself** as the canonical Sell trigger.

## Stock reserve behavior

For unfiltered pickup-all the source requires:

```text
GetFreeBagSpace() > #items
```

not `>= #items`.

For many filtered single-slot pickups it requires:

```text
GetFreeBagSpace() > 1
```

This effectively preserves at least one spare slot. Whether intentional or conservative, it means a production tool should expose a configurable threshold such as:

```text
Sell when FreeBagSpace <= N
```

rather than waiting only for exact zero.

## Built-in loot filters

Recovered filter settings:

```text
IsFilterItem
FilterItemSettings = material/equipment/min-star/quest/gem flags
PickRanger (default 500)
```

Source classifies broad item groups using ItemID range/index plus equipment template data/star information. These filters are useful donors for loot policy, but Auto Sell should still classify **current bag instances** independently.

## Important incomplete/stale stock settings

The PickUp UI also exposes settings named:

```text
AutoUsingItem
UsingItemList
IsAutoDropItem
DropItemSettings
```

However, in the recovered Lua corpus:

- `AutoUsingItem`/`UsingItemList` are configured by `PickUp.lua`, but no corresponding execution loop was found in `AutoFight_Main.lua`;
- `IsAutoDropItem`/`DropItemSettings` are serialized, but no actual auto-drop execution path was found;
- deserialization assigns `DropItemSettings = PickItemSettingsPrams[8]`, the same slot used for the boolean `IsAutoDropItem`, while serialization writes a ninth field. This is a strong source-level defect/stale implementation signal.

Do **not** assume these visible settings actually implement reliable auto-use/auto-drop behavior.

## Correct EXE transition

```text
TRAINING
 -> fresh FreeBagSpace
 -> threshold reached
 -> yield/stop Train + pickup
 -> SELL_ROUTE
 -> current normal shop
 -> sell-one/proof/rescan loop
 -> free-space target reached
 -> return train spot
 -> restore pickup setting/policy
 -> resume Train
```

The tool should own this transition rather than waiting for a periodic screen-based bag check.
