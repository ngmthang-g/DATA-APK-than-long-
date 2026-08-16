# Bag scanner data contract — mobile EXE

Status: **QUERY SURFACE / LIVE INSTANCE SCHEMA VERIFIED STATIC; live repeated scan remains runtime proof**

Purpose: define the data for a useful Windows bag scanner / KEEP-SELL policy editor without opening the in-game bag UI or relying on icons/OCR.

---

# 1. Enumerate current bag instances

Canonical current source:

```text
Game.GetItemsAtSite(C_ItemSite.Bag)
C_ItemSite.Bag = 10
```

Each `DBItemData` supplies live instance state:

```text
ID
ItemID
RoleID
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

Important identity:

```text
ID       = live instance/database ID used by item/sell requests
ItemID   = template ID used for semantic template queries
Position = current container slot
```

Never display or label ItemID as “instance ID”.

---

# 2. Exact semantic template queries

`LuaSystemAPI_Game` exposes:

```text
GetItemTemplateData(itemID)  token 0x060006FC
GetItemIcon(itemID)          token 0x060006FF
GetItemName(itemID)          token 0x06000700
GetEquipVisualID(itemID)     token 0x06000701
GetEquipStar(itemID)         token 0x06000702
GetEquipLevel(itemID)        token 0x06000703
GetItemType(itemID)          token 0x06000704
GetItemData(dbID)            token 0x06000705
CountItem(itemID,site)       token 0x06000706
GetItemAtSite(site,pos)      token 0x06000707
IsItemThrowable(itemID)      token 0x0600070B
IsItemSellable(itemID)       token 0x0600070C
GetEquipBoundRule(itemID)    token 0x0600070D
GetEquipType(itemID)         token 0x0600070E
GetEquipSet(itemID)          token 0x06000714
GetItemBasePrice(itemID)     token 0x06000715
GetItemBuyPrice(itemID)      token 0x06000716
IsItemSellToShopWithBoundMoney(itemID) token 0x06000717
GetItemMaxStack(itemID)      token 0x06000718
GetItemExtraHint(itemID)     token 0x0600071A
```

This allows the scanner to resolve names/types/prices dynamically even though the full static PC-style Config tables are not packaged in the base APK.

---

# 3. Recommended scanner row

For every current Bag instance return:

```text
InstanceID
TemplateItemID
SlotPosition
Name
ItemType
Quantity
Bound
Creator
Durability

EquipPosition?       # Game.GetEquipType when ItemType==Equip
EquipPositionName?   # tool lookup from EQUIP_POSITIONS_MOBILE.csv
IsWeapon             # ItemType==Equip && EquipPosition==0
TemplateEquipStar?
InstanceEquipStar?   # Properties[118] if present
EquipLevel?

BasePrice
EstimatedBaseValue = BasePrice * Quantity
BuyPrice?
SellToBoundMoney?
SellableByGame
ThrowableByGame
MaxStack

RecoveryType = HP_MEDICINE / MP_MEDICINE / NONE
PolicyDecision
PolicyReason
BagVersion
```

The `?` fields are conditional; do not force an equipment query onto non-equipment item types.

## Price caution

Recovered NPCShop buy-back UI computes:

```text
basePrice = Game.GetItemBasePrice(ItemID) * Quantity
```

This proves the base price is useful for UI/diagnostics. Do not promise it equals the final amount actually received from every sell transaction until runtime money/shop behavior confirms the relevant rules (bound money, server modifiers, etc.).

Use names such as:

```text
BaseValue
EstimatedBaseValue
```

rather than `GuaranteedSellGold`.

---

# 4. Weapon and equipment UI

Exact mobile weapon rule:

```text
ItemType == "Equip"
AND
Game.GetEquipType(ItemID) == C_EquipPosition.Weapon[1] == 0
```

Display:

```text
Equip position: Vũ khí
IsWeapon: true
```

For other equipment, use `database/EQUIP_POSITIONS_MOBILE.csv`.

Do not use `<10` as a weapon rule.

---

# 5. Recovery protection

For every template evaluate:

```text
Game.IsHPMedicine(ItemID)
Game.IsMPMedicine(ItemID)
```

If the profile uses the template for Auto Recovery, mark:

```text
PolicyDecision = RESERVED
PolicyReason = AUTO_RECOVERY_HP / AUTO_RECOVERY_MP
```

before applying Sell/Drop/Destroy rules.

---

# 6. KEEP / SELL policy editor

Recommended Windows UI can show two logical rule panels rather than moving live instance IDs permanently:

```text
KEEP rules                 SELL rules
------------------------------------------------
Weapon position            CommonItem type
Template ItemID X          Template ItemID Y
Medicine reserve           selected non-weapon equips
Gem                         etc.
```

When the user selects a current item and chooses “Giữ loại này” or “Bán loại này”, decide whether the rule should target:

```text
exact TemplateItemID
ItemType
EquipPosition
star/level condition
other explicit semantic condition
```

Do not persist current `InstanceID` as a long-term category rule because that live instance can disappear and another instance will have a different ID.

A temporary `NeverSellInstanceIDs` reservation may exist only inside the current session/transaction.

---

# 7. Sorting and bag positions

`Position` is the current slot and can change after sort/move/update. It is useful for display/debug but is not stable action identity.

After:

```text
bag sort
item move
sell
use/drop/destroy
UpdateItemsList
```

increment BagVersion and rebuild rows from current `GetItemsAtSite(10)`.

Do not retain a `slot -> item` map as truth across mutations.

---

# 8. Scanner timing

For UI display, periodic lightweight refresh can be throttled. For a destructive action decision, always use a **fresh scan tied to current BagVersion**.

Event-driven invalidation from `RemoveItem` / `UpdateItemsList` is preferred over very high-frequency full list polling.

Bag-full monitoring itself does not require enumerating every item; `GetFreeBagSpace()` can be sampled independently and cheaply for the Train→Sell trigger.

---

# 9. Multi-LD boundary

Each row must carry SessionId/BagVersion or be nested under one session. The same template ItemID can exist in multiple LDs with different live instance IDs, quantities and policies.

Never let selecting an instance row in LD0 cause an action against LD1 just because the template ItemID/slot looks the same.
