# Auto Tool API Catalog — Mobile

This catalog now includes exact mobile metadata signatures/source usage where recovered.

| Domain | API/symbol | Mobile status | Tool use |
|---|---|---|---|
| role | `Game.RoleData` / `RoleData` | exact source+metadata | main local snapshot |
| vital | CurrentHP/MaxHP/CurrentMP/MaxMP | exact metadata | survival |
| death | `LuaLeaderData.IsDeath`, `GRole.IsDeath` | exact metadata | death truth candidates |
| map | MapID/PosX/PosY/Position, `IsMapReady` | exact source+metadata | transition proof |
| map | `GetCurrentMoveDestination` | exact metadata | movement progress |
| movement | `MoveTo`, `MoveToEx`, `GoTo` | exact source+metadata | travel |
| path | `AutoPathManager.StartAutoPath/StopAutoPath/IsAutoPathing` | exact metadata | route lifecycle |
| path | `TCPGameEventProcessor.SendAutoPathRequestChangeMap` | exact metadata | producer donor |
| NPC | `GetNPCPosition`, `ClickNPC` | exact source+metadata | service navigation |
| world | `GetNearbySpritesWithPredicate` | exact source+metadata | Train target scan |
| target | `SelectTarget`, `ReloadTarget`, `IsSelectTargetDie`, `ChaseTarget` | exact source+metadata | combat |
| skill | `GetSkillLuaData`, `GetSkillCooldown`, `RequestUsingSkillWithTarget` | exact source+metadata | combat |
| bag | `GetFreeBagSpace`, `GetItemsAtSite` | exact metadata/source usage | full threshold/scan |
| item | `DBItemData.ID/ItemID/Site/Position/Bound/Quantity` | exact metadata | live identity |
| item | `IsItemSellable`, `GetItemType`, `GetEquipType`, `GetItemBasePrice` | exact source+metadata | keep/sell |
| dialog | `GameDialog.Selections` runtime map | exact Lua source | semantic NPC action |
| shop | current `shopData.NpcShopID`, `shopData.ID`, `IsGuildShop` | exact Lua source | sell guard/payload |
| chat | `WorldToGridPosition`, `RoleData.Position`, `C_ChatChannel` | exact Lua source | @GOTO + channels |
| network | `LuaSystemAPI_Network.SendPacket` | exact metadata/source usage | semantic packet bridge |
| network | `TCPGame.SendPacket`, `TCPOutPacket.MakeTCPOutPacket` | exact metadata | lower-level producer |
| thread | `MainThread.Execute(Action)` | exact metadata member map | safe managed dispatch candidate |

## Exact high-level actions

See `database/AUTO_ACTION_EXACT_FLOWS.md` before searching native code.
