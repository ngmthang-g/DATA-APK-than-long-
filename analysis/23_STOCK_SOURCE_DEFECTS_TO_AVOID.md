# Stock mobile source defects / stale paths to avoid copying into the EXE

Status: **VERIFIED SOURCE OBSERVATIONS + one cross-platform identity conflict**

Purpose: the shipped auto is a valuable behavioral donor, but it is not automatically production-quality. This file records concrete source defects/stale settings that an external tool should not clone blindly.

## 1. Auto-drop settings deserialize the wrong slot

`AutoFight_Main` serializes PICKITEM as nine fields, ending with:

```text
...
AutoUsingItem
UsingItemList
IsAutoDropItem
DropItemSettings
```

But deserialization reads:

```text
IsAutoDropItem    = PickItemSettingsPrams[8]
DropItemSettings  = PickItemSettingsPrams[8]
```

The drop-list should logically come from field 9. This is a direct source-level indexing defect/stale path.

Implication: do not reuse the stock serialized auto-drop configuration format as production truth without correcting/migrating it.

## 2. AutoUsingItem / AutoDrop UI exists without an observed execution loop

`PickUp.lua` exposes/configures:

```text
AutoUsingItem
UsingItemList
IsAutoDropItem
DropItemSettings
```

The recovered `AutoFight_Main.lua` loads/saves these values, but a corpus-wide search of the recovered Interface Lua did not find a corresponding general automatic-use/automatic-drop execution path. This is evidence that visible settings may be unfinished, removed or delegated to code/resources not present in this corpus.

Implication: the Windows tool should implement explicit semantic item policies through verified `CMD_ITEM_ACTION` actions rather than assuming the game's switches work.

## 3. Auto HP/MP timestamp guard is ineffective after successful item use

`DoAutoRegen()` checks:

```text
System.NowTicks - LastTrigerHpRegen > 1000
```

However, every successful medicine branch does:

```text
Network.SendPacket(... Use ...)
return true
```

before execution reaches the final:

```text
LastTrigerHpRegen = System.NowTicks
```

Therefore a successful medicine use does not update this inner timestamp guard. Outer coroutine scheduling still reduces call frequency, but this is not a robust action-success limiter.

Implication: external automation should use:

```text
fresh HP/item state -> one Use action -> wait for state/item/cooldown proof -> rescan
```

not a copied timestamp-only loop.

## 4. Loot reserve comparisons are conservative and asymmetric

Stock pickup often requires `GetFreeBagSpace() > 1` for one selected item and `GetFreeBagSpace() > #items` for a pack, rather than `>=`. This intentionally or accidentally leaves a spare slot.

Implication: expose a user/tool policy `Sell when FreeBagSpace <= N`; do not infer that only exact zero means full.

## 5. Nga My symbolic naming conflict — do not trust variable name as skill identity

Mobile `Global_Constants.txt` contains:

```text
C_NMBuff.KIMCHAMDOKIEP = 407
```

But the independently analyzed PC frozen Config data identifies skill 407 as **Xung Hư Dưỡng Khí** and 423 as **Kim Châm Độ Kiếp**.

This is a **cross-source conflict**, not permission to silently rewrite the mobile constant. Possible explanations include stale Lua variable naming, version/content differences, or server/config remapping.

Implication: never identify a real skill solely from the Lua variable label. Resolve actual mobile skill template/name/runtime behavior before production support/buff logic.

## General donor rule

For every shipped-auto behavior distinguish:

```text
semantic idea worth reusing
vs.
exact implementation worth copying
```

Prefer its semantic APIs and packet producers, but replace stale timing, fragile serialization and ambiguous symbolic names with fresh state/action/proof contracts.
