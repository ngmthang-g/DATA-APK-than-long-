# Lua / Game / UI / Network API Surface

## High-value bridge classes

Current metadata contains:

```text
FGStudio.LuaSystem.API.LuaSystemAPI_Game
LuaSystemAPI_Network
LuaSystemManager
```

These mirror the architecture seen in the PC client and are the preferred semantic research surface before raw native disassembly.

## Game/query/action names found

Current metadata contains high-value names such as:

```text
GetNearByPeacePlayers
GetNearbySpritesWithPredicate
SelectedTarget
GetNearByEnemies
GetFreeBagSpace
GetItemsAtSite
GetNPCPosition
ClickNPC
MoveTo
MoveToEx
GoTo
SelectTarget
ChaseTarget
UseSkill
RequestUsingSkill
RequestUsingSkillWithTarget
RequestUsingSkillWithPos
GetSkillCooldown
GetBuffs
GetSkillLuaData
IsMapReady
```

A prior metadata parse grouped many of these under the LuaSystem Game API. Current static string recheck confirms the names exist in the frozen APK; exact signature/ownership should be emitted by the mobile resolver/dumper into later catalogs.

## Network boundary

Names present:

```text
SendPacketToServer
SendPacket
TCPGame
TCPOutPacket
PacketCmdID
CMD_REVIVE
```

This is critical because unknown UI actions can often be solved with a narrow producer trace:

```text
manual UI action
 -> semantic Lua/Game producer
 -> SendPacketToServer / TCPGame.SendPacket
 -> command ID + payload
 -> inbound/update state proof
```

That is preferred over blind click automation.

## Revive research

`ProcessObjectDeath`, `ProcessObjectRevive`, `IsDeath` and `CMD_REVIVE` are present. This proves a native/managed revive flow exists.

Missing exact fact: outbound command number/payload for the user pressing normal revive/Đầu thai on this mobile snapshot.

## Sell research

Static inventory APIs prove item selection can be semantic. The missing step is the exact shop/sell producer path and required current shop identifiers.

Preferred proof:

```text
open vendor
 -> capture current shop/dialog state
 -> sell one item manually
 -> trace one outbound command
 -> verify live item mutation
```

## High-level Train donor

PC source research found a high-level built-in `AutoFight_Main` Train flow. Current mobile string scan in this pass did not find exact standalone names `AutoFight_Main` or `StartAutoFight`.

Therefore do not claim the same high-level UI/Lua path exists in this APK yet. Low-level target/chase/skill primitives are already enough to continue architecture research.
