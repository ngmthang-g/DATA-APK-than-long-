# Auto Tool Action Catalog — Mobile

Evidence statuses below refer to the frozen mobile APK.

| Action | Exact mobile semantic/action | Status | Result proof |
|---|---|---|---|
| Start Train | `AutoFight_Main:StartAutoFight(C_AutoModel.Train)`, Train=1 | STATIC-SOLVED | target/combat state + clean stop |
| Stop/yield Train | `StartAutoFight(C_AutoModel.None)` / mode None | STATIC-SOLVED | no new train mutation |
| Move | `Game.MoveTo`, `MoveToEx` | STATIC-SOLVED | fresh position |
| Cross-map travel | `Game.GoTo(mapID,x,y)` | STATIC-SOLVED | MapID + IsMapReady + tolerance |
| NPC route | `GetNPCPosition -> GoTo -> ClickNPC` / shipped `GoToNPC` | STATIC-SOLVED | current dialog/shop |
| Select/chase | `SelectTarget`, `ChaseTarget` | STATIC-SOLVED | current target/movement |
| Skill on target | `RequestUsingSkillWithTarget(skillID,RoleID)` | STATIC-SOLVED | cooldown/combat/server state |
| Pick ground item | `PickUpItemFromItemPack(itemPackID,slotIndex,UsingAuto)` | STATIC-SOLVED / RUNTIME-PROOF | bag/item-pack state changes |
| Trigger Sell from bag | `GetFreeBagSpace() <= configured threshold` | STATIC-SOLVED state policy | transition ownership, no pickup conflict |
| Sort Bag | `CMD_BAG_SORT=100006`, payload `10` | STATIC-SOLVED | UpdateItemsList/fresh BagVersion; optional utility only |
| Sort any item site | `100006`, payload `site` | STATIC-SOLVED | corresponding item list refresh |
| Use medicine/item | `100005`, `3:itemInstanceID` | STATIC-SOLVED / RUNTIME-PROOF | HP/MP/item/cooldown change |
| Abandon item | `100005`, `4:itemInstanceID` | STATIC-SOLVED / RUNTIME-PROOF | removal/fresh bag; require `IsItemThrowable` + explicit policy |
| Move item to site | `100005`, `5:itemInstanceID:destinationSite` | STATIC-SOLVED / RUNTIME-PROOF | item site changes |
| Merge items | `100005`, `7:id1;id2;...` | STATIC-SOLVED | fresh quantities/instances |
| Split item | `100005`, `8:itemInstanceID:quantity` | STATIC-SOLVED | new instances/quantities |
| Destroy items | `100005`, `9:id1;id2;...` | STATIC-SOLVED but DANGEROUS | removal proof; explicit narrow opt-in only |
| Normal revive | `Network.SendPacket(200063,"1")` | STATIC-SOLVED / RUNTIME-PROOF | alive/HP/revival/map state |
| Newbie revive | `200063:"2"` | STATIC-SOLVED | revive state |
| Skill revive | `200063:"3"` | STATIC-SOLVED | revive state |
| GameDialog selection | `100007`, `selectionID:SelectedItemID` | STATIC-SOLVED | next dialog/UI/server state |
| Open normal shop | inbound `200034` gives current shop data | STATIC-SOLVED lifecycle | shop exists + IsGuildShop=false |
| Sell one live item | `200036`, `itemInstanceID:NpcShopID:ShopID` | STATIC-SOLVED / RUNTIME-PROOF | RemoveItem/UpdateItemsList/fresh bag |
| Send chat | `100008`, object `{RoleID,Name,Content=Base64(text),Channel}` | STATIC-SOLVED / RUNTIME-PROOF | inbound/visible/server response |
| Send location ping | append `@GOTO_MapID_GridX_GridY`, then normal chat | STATIC-SOLVED / RUNTIME-PROOF | received clickable location |

## Exact weapon keep rule

Recovered mobile Lua uses `Game.GetEquipType(ItemID)` as **equipment position**, comparing it directly to `C_EquipPosition`.

```text
weapon = Game.GetItemType(ItemID) == "Equip"
      && Game.GetEquipType(ItemID) == C_EquipPosition.Weapon[1]
      && C_EquipPosition.Weapon[1] == 0
```

Do not use `GetEquipType < 10` as a weapon rule. See `database/AUTO_SELL_CLASSIFICATION.md`.

## Item/survival guards

```text
use current DBItemData.ID, never stale/template identity
configured HP/MP medicine is reserved from Sell/Drop/Destroy
Abandon requires Game.IsItemThrowable(ItemID) plus user's explicit policy
bulk Destroy is disabled by default
Bag sort is not required for FreeBagSpace or semantic bag scanning
```

## Sell guards

```text
40000000 <= ItemID < 50000000 -> do not sell
Game.IsItemSellable(ItemID) must be true
use DBItemData.ID as live instance identity
current shop must not be guild shop
```

## Mutation contract

```text
fresh snapshot -> guard -> ONE mutable action -> concrete proof -> fresh snapshot
```

Do not invoke response handlers as requests. Do not run Train/pickup actions concurrently with Sell/Heal/Revive ownership. Do not copy shipped AutoDrop/AutoUsing switches as proof that those engines are implemented; see `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md`.
