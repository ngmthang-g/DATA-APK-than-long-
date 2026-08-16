# Item actions + built-in HP/MP recovery — mobile

Status: **EXACT STATIC SOURCE RECOVERED**

## Core item packet

```text
CMD_ITEM_ACTION = 100005
```

Recovered `C_ItemAction` values relevant to automation:

```text
Use     = 3
Abandon = 4
Move    = 5
Merge   = 7
Split   = 8
Destroy = 9
```

## Exact payloads

### Use one live item

```text
3:<DBItemData.ID>
```

Used by `RoleInfo_BagTab`, `QuickItemsBar` and the stock Auto HP/MP engine.

### Abandon one live item

```text
4:<DBItemData.ID>
```

Stock bag UI only exposes this action when:

```text
Game.IsItemThrowable(dbItemData.ItemID) == true
```

The stock UI displays a destructive confirmation first. An external tool must preserve at least the same throwability guard plus the user's explicit keep/drop policy.

### Move item between sites

```text
5:<DBItemData.ID>:<destinationSite>
```

Examples in Storage/FashionBag source. This is a semantic item move, not a screen drag.

### Destroy multiple live items

```text
9:<id1;id2;id3;...>
```

Recovered from `DestroyItems.lua`. The stock frame allows up to 35 staged items and displays an irreversible confirmation before sending. Because this operation is destructive and its stock frame does not provide the same simple `IsItemThrowable` guard as ordinary Abandon, production automation should **not** use bulk Destroy unless the user explicitly enables a narrowly defined destroy policy and the exact candidate list is freshly rescanned.

## Built-in Auto HP/MP recovery

`AutoHp.lua` lets the user select bag items only when:

```text
Game.IsHPMedicine(ItemID)
Game.IsMPMedicine(ItemID)
```

Settings include:

```text
AutoRegenHP
AutoRegenHPPercent
HPItemRegen
AutoRegenMP
AutoRegenMPPercent
MPItemRegen
AutoRevival
AutoComeback
```

`AutoFight_Main:DoAutoRegen()` checks current `RoleData.HPPercent` / `MPPercent`, finds a matching live bag instance by template ItemID and sends `100005` with `Use=3` and the **live DBItemData.ID**.

The Train loop calls recovery approximately every third 0.3-second tick path (comments describe ~1.5 s in some modes; actual scheduling varies by mode/other yields). Do not depend on the stock timing as a production contract.

## Important implementation note

Inside `DoAutoRegen`, successful item-use branches return before the final assignment to `LastTrigerHpRegen`. Therefore that inner timestamp guard is not a reliable success-rate limiter on those paths. Outer loop scheduling still limits how often the function is reached, but an external tool should use stronger semantics:

```text
fresh HP/item snapshot
 -> one Use request
 -> wait HP/item/cooldown/action-state change
 -> fresh snapshot
 -> decide again
```

Do not fire repeated Use packets merely because HP is still below threshold immediately after the request.

## Relation to Auto Sell

If Auto Sell and Auto HP use the same item type, survival policy wins. Build a keep rule for configured HP/MP medicine before selecting sell/drop/destroy candidates.
