# Auto Sell classification contract — mobile

Status: **SEMANTIC MOBILE API/TYPE/POSITION RULES VERIFIED; final user sell policy is configuration**

Purpose: separate three different questions:

1. **Can the game sell this item?**
2. **What kind/position of item is it?**
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

Treat unknown/new strings as `Unknown` and default KEEP.

## Medicine protection

If either:

```text
Game.IsHPMedicine(ItemID)
Game.IsMPMedicine(ItemID)
```

or the template ID is configured as recovery medicine, classify it as `PROTECTED_RECOVERY` before any sell/drop rule.

This prevents Auto Sell from consuming the item pool needed by Auto Recovery.

## `Game.GetEquipType()` is actually used as equipment POSITION

This is an important source-level clarification.

The Lua constants contain both:

```text
C_EquipType
C_EquipPosition
```

Their names make it tempting to assume `Game.GetEquipType()` returns `C_EquipType`. Recovered client source proves otherwise in the automation-relevant context: `GetEquipType(ItemID)` is repeatedly compared directly to `C_EquipPosition.*[1]` and is used as the position index for `GetItemAtSite(Body,equipPos)`.

Examples:

```text
ItemTooltip.lua:
  equipPos = Game.GetEquipType(ItemID)
  equipped = Game.GetItemAtSite(C_ItemSite.Body, equipPos)

ExchangeSuperWeaponPiece.lua:
  Game.GetEquipType(ItemID) == C_EquipPosition.Weapon[1]

StarUpSuperWeapon.lua:
  Game.GetEquipType(ItemID) == C_EquipPosition.Weapon[1]

DragonTattoo...:
  Game.GetEquipType(ItemID) == C_EquipPosition.DragonTattoo[1]
```

Therefore for this mobile source, treat the value operationally as **equip position**.

## Exact mobile weapon classification

Recovered constant:

```text
C_EquipPosition.Weapon = {0, "Vũ khí"}
```

So the exact semantic mobile rule for a normal equipment item occupying the weapon slot is:

```text
Game.GetItemType(ItemID) == "Equip"
AND
Game.GetEquipType(ItemID) == 0
```

This is now independently mobile source-backed and aligns conceptually with the PC static `EquipPoint == 0` rule, but the proof here is from the mobile Lua constants/usages themselves.

### Do NOT use `< 10 => weapon`

`C_EquipType` separately contains weapon-family-looking values 0..8 plus `Preserved=9`, but **that is not the enum the recovered Lua is comparing `Game.GetEquipType()` against** for equipment position decisions.

So this is wrong:

```text
Game.GetEquipType(ItemID) < 10 => weapon
```

Correct for the desired “giữ lại vũ khí” policy:

```text
item type == Equip
AND equip position == Weapon(0)
```

This distinction prevents hats/shoes/other positions from being accidentally grouped by a misleading enum-name inference.

## Other exact equipment positions

Recovered `C_EquipPosition` includes:

```text
0  Weapon
1  Hat
2  Cloth
3  Gloves
4  Shoes
5  Belt
6  Ring
7  Necklace
8  Mount
11 Ring_2
12 Amulet
13 Amulet_2
14 Cuff
15 Shoulderpads
16 Fashion
17 Dart
18 Soul
19 DragonTattoo
20 HeroicOrder
21 Signet
22 WeaponVisual
```

These values can support explicit profile rules such as:

```text
KEEP all Weapon
SELL selected non-weapon equipment meeting star/level conditions
KEEP Dart/Soul/DragonTattoo/other special positions
```

but default policy should still remain conservative.

## Equipment protection / richer rules

For `GetItemType(ItemID) == "Equip"` or `"PetEquip"`:

- default KEEP unless the profile explicitly opts into equipment selling;
- `DBItemData.Properties[C_ItemProperty.EquipStar]` uses property key `118` when present;
- non-empty `Creator` is used by shipped source to recognize crafted equipment in several systems;
- `Game.GetEquipLevel(ItemID)` and other semantic APIs can enrich rules when needed;
- do not use icon/color/OCR as classification truth.

## High-value protection signals

Recommended default KEEP candidates include:

```text
Unknown/Undefined classification
Medicine / configured recovery item
Gem unless explicitly sold
Weapon equipment (Equip + position 0)
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
- specific ItemIDs/ranges;
- explicit equipment positions;
- item types/subtypes proven by runtime/template data.

## Example policy for the user's earlier “keep weapons, sell other eligible trash” idea

Conservative version:

```text
IF not current Bag instance -> ignore
IF quest/protected/unsellable -> KEEP
IF recovery medicine -> KEEP
IF itemType == Equip AND equipPosition == 0 -> KEEP_WEAPON
IF itemType == Equip -> KEEP by default until explicit non-weapon equipment sell rule is enabled
IF explicit Sell rule matches -> SELL
ELSE -> KEEP
```

After testing, a profile can intentionally broaden the non-weapon equipment Sell rule without changing weapon identity logic.

## Rule order

Recommended deterministic order:

```text
1 current fresh Bag instance?
2 hard game guard / quest range?
3 active transaction reservation?
4 recovery/survival protection?
5 exact Weapon/other protected equip position?
6 explicit KEEP ItemID/rule?
7 Unknown or sensitive category default KEEP?
8 explicit SELL rule match?
9 otherwise KEEP
```

This means **no broad default SELL**.

## Candidate record for EXE UI/logging

For each bag row expose at least:

```text
InstanceID
ItemID
Name if semantic template data available
ItemType
EquipPosition if ItemType==Equip
IsWeapon = (ItemType==Equip && EquipPosition==0)
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
