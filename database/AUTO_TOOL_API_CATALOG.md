# Auto Tool API Catalog — Mobile

Evidence here is primarily exact metadata-name presence; signature/runtime ownership is refined during resolver tests.

| Domain | Symbol/API | Static status | Intended use |
|---|---|---|---|
| player | `RoleData` | VERIFIED name | local player state root candidate |
| player | `LuaLeaderData` | VERIFIED name | leader/player shared data candidate |
| vital | `CurrentHP`, `MaxHP`, `IsDeath` | VERIFIED names | survival/death state |
| map | `MapID`, `PosX`, `PosY`, `Position` | VERIFIED names | location snapshot |
| map | `IsMapReady` | VERIFIED name | transition guard/proof |
| map | `GetCurrentMoveDestination` | VERIFIED name | path progress |
| world | `GetLocalMapObjects`, `GetNearbyObjects` | VERIFIED names | environment scan |
| target | `SelectedTarget` | VERIFIED name | target snapshot |
| target | `GetNearByEnemies` | VERIFIED name | train candidate scan |
| target | `GetNearbySpritesWithPredicate` | VERIFIED name | filtered sprite scan donor |
| player scan | `GetNearByPeacePlayers` | VERIFIED name | nearby player scan donor |
| combat | `SelectTarget` | VERIFIED name | select semantic target |
| combat | `ChaseTarget` | VERIFIED name | move/chase target |
| combat | `UseSkill`, `RequestUsingSkill*` | VERIFIED names | semantic skill request |
| combat | `GetSkillCooldown`, `GetBuffs`, `GetSkillLuaData` | VERIFIED names | cooldown/buff/skill metadata |
| movement | `MoveTo`, `MoveToEx`, `GoTo` | VERIFIED names | semantic travel |
| path | `AutoPathManager`, `StartAutoPath`, `StopAutoPath`, `IsAutoPathing` | VERIFIED names | auto-route control/status |
| NPC | `GetNPCPosition`, `ClickNPC` | VERIFIED names | vendor/service route |
| bag | `GetFreeBagSpace` | VERIFIED name | bag threshold |
| bag | `GetItemsAtSite`, `GetItems`, `GetItemData`, `GetItemAtSite`, `GetTotalItems`, `CountItem` | VERIFIED names | live inventory scan |
| item | `IsItemSellable`, `GetItemBasePrice`, `GetItemType`, `GetEquipType`, `IsItemThrowable` | VERIFIED names | keep/sell policy |
| network | `SendPacketToServer`, `SendPacket`, `TCPGame`, `TCPOutPacket` | VERIFIED names | exact action producer trace |
| revive | `CMD_REVIVE`, `ProcessObjectDeath`, `ProcessObjectRevive` | VERIFIED names | revive research/state |
| thread | `MainThread` | RECONFIRMED | safe semantic action dispatch candidate |

Never infer exact numeric enum/packet values from name presence alone.
