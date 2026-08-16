# Auto Sell classification contract — mobile

Status: **SEMANTIC MOBILE API/TYPE STRINGS VERIFIED; final user sell policy is configuration**

Purpose: separate three different questions:

1. **Can the game sell this item?**
2. **What kind of item is it?**
3. **Does this tool profile want to sell it?**

Never collapse them into one boolean.

## Mandatory live identity

Every decision starts from a fresh Bag instance:

```text
DBItemData.ID       = live instance/database ID
DBItemData.ItemID   = template ID
DBItemData.Site     = current container
DBItemData.Position = current slot
DBItemData.Bound
DBItemData.Quantity
DBItemData.Creator
DBItemData.Properties
DBItemData.Attributes
```

Require `Site == Bag (10)` for normal Auto Sell candidates.

## Stock game sell guards

Recovered `NPCShop_SellItemTab` blocks:

```text
40000000 <= ItemID < 50000000
Game.IsItemSellable(ItemID) == false
```

Preserve both guards.

`IsItemSellable=true` means only **the client allows a normal shop sale**. It does not mean the user wants the tool to sell it.

## Observed semantic item-type strings

Recovered Lua source compares `Game.GetItemType(ItemID)` against values including:

```text
CommonItem
ScriptItem
Medicine
Gem
PetEquip
Equip
Iron
Rice
Salt
Undefined
```

The first six appear broadly in item UI/filter logic; the material-like values also appear in specialized logic. Treat unknown/new strings as `Unknown` and default KEEP.

## Medicine protection

If either:

```text
Game.IsHPMedicine(ItemID)
Game.IsMPMedicine(ItemID)
```

or the template ID is configured as recovery medicine, classify it as `PROTECTED_RECOVERY` before any sell/drop rule.

This prevents Auto Sell from consuming the item pool needed by Auto Recovery.

## Equipment caution

For `GetItemType(ItemID) == "Equip"` or `"PetEquip"`:

- default KEEP unless the profile explicitly opts into equipment selling;
- inspect fresh semantic/static fields needed by the rule (subtype, level, star, creator, bound, etc.);
- `DBItemData.Properties[C_ItemProperty.EquipStar]` uses property key `118` when present;
- non-empty `Creator` is used by shipped source to recognize crafted equipment in several systems.

### Do not infer weapon identity from a numeric shortcut

The mobile source contains both `C_EquipType` and `C_EquipPosition` concepts with overlapping numeric domains, and `Game.GetEquipType` is used in multiple feature contexts. Until one exact production rule is validated against mobile runtime/template data, do **not** use guesses such as:

```text
GetEquipType < 10 => weapon
```

or icon/color/OCR.

The PC static database fact `Equips.EquipPoint == 0` is a useful donor for PC weapon classification, not automatically a mobile runtime contract.

## High-value protection signals

Recommended default KEEP candidates include:

```text
Unknown/Undefined classification
Medicine / configured recovery item
Gem unless explicitly sold
Equip/PetEquip unless explicit equipment policy
quest ItemID range 40000000..49999999
IsItemSellable == false
explicit template keep list
explicit live-instance keep reservation
items involved in a current transaction/action
```

Additional profile rules may protect:

- bound/unbound state;
- minimum equipment star;
- creator/crafted equipment;
- item quantity reserve;
- specific ItemIDs or ranges;
- item types/subtypes proven by runtime/template data.

## Rule order

Recommended deterministic order:

```text
1 current fresh Bag instance?
2 hard game guard / quest range?
3 active transaction reservation?
4 recovery/survival protection?
5 explicit KEEP ItemID/rule?
6 Unknown or sensitive category default KEEP?
7 explicit SELL rule match?
8 otherwise KEEP
```

This means **no broad default SELL**.

## Candidate record for EXE UI/logging

For each bag row expose at least:

```text
InstanceID
ItemID
Name if semantic template data available
Type
EquipSubtype/Position if validated
Star if present
Bound
Quantity
SellableByGame
MatchedPolicyRule
Decision = KEEP / SELL / DROP / DESTROY / RESERVED
Reason
```

This makes destructive automation auditable and lets the user move exact categories/templates between keep/sell lists without relying on screen cells.

## Sell execution boundary

Classification chooses a candidate. Execution still requires:

```text
current normal NPCShop
fresh candidate still present
current DBItemData.ID
one 200036 request
removal/update proof
fresh rescan
```

Never classify once and then blindly sell a cached 90-item list.
