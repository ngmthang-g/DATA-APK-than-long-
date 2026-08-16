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
| Normal revive | `Network.SendPacket(200063,"1")` | STATIC-SOLVED / RUNTIME-PROOF | alive/HP/revival/map state |
| Newbie revive | `200063:"2"` | STATIC-SOLVED | revive state |
| Skill revive | `200063:"3"` | STATIC-SOLVED | revive state |
| GameDialog selection | `100007`, `selectionID:SelectedItemID` | STATIC-SOLVED | next dialog/UI/server state |
| Open normal shop | inbound `200034` gives current shop data | STATIC-SOLVED lifecycle | shop exists + IsGuildShop=false |
| Sell one live item | `200036`, `itemInstanceID:NpcShopID:ShopID` | STATIC-SOLVED / RUNTIME-PROOF | RemoveItem/UpdateItemsList/fresh bag |
| Send chat | `100008`, object `{RoleID,Name,Content=Base64(text),Channel}` | STATIC-SOLVED / RUNTIME-PROOF | inbound/visible/server response |
| Send location ping | append `@GOTO_MapID_GridX_GridY`, then normal chat | STATIC-SOLVED / RUNTIME-PROOF | received clickable location |

## Item guards for Sell

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

Do not invoke response handlers as requests and do not run Train combat actions concurrently with Sell/Heal/Revive ownership.
