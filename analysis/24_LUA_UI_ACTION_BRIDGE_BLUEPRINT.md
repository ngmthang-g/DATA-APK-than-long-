# Lua/UI semantic action bridge blueprint — Android LD9

Status: **STATIC BRIDGE SURFACE VERIFIED; external live invocation/lifetime remains RUNTIME-PROOF**

## Why this matters

Many high-value mobile actions are implemented in recovered Lua (`AutoFight_Main`, `NPCShop_SellItemTab`, `GameDialog`, `ChatBox`) while the host tool is a Windows EXE and the game process is Android ARM64. The guest bridge needs a stable way to call the game-owned Lua/UI layer without screen clicks.

## Official C# Lua manager surface

`FGStudio.LuaSystem.LuaSystemManager` exact metadata members:

```text
CreateTable(luaString)             token 0x060000F1
CreateTable()                      token 0x060000F2
OnReceiveEvent(...)                0x060000F3..F5
OnReceivePacket(data)              0x060000F6
SendPacketToServer(packetID,data)  0x060000F7
HasScript(uiName)                  0x060000F8
GetScript(uiName)                  0x060000F9
LoadFromAssetBundle(assetBundle)   0x060000FA
LoadFromFolder(directory)          0x060000FB
ExecuteFunction(closure,args)      0x060000FC
ExecuteFunction(functionName,args) 0x060000FD
get_LuaEnv()                       0x060000FE
Reload()                           0x06000100
Initialize()                       0x06000101
```

Original source-path strings independently identify:

```text
Assets\Scripts\LuaSystem\LuaSystemManager_API.cs
Assets\Scripts\LuaSystem\LuaSystemManager_Network.cs
Assets\Scripts\LuaSystem\LuaSystemManager_Script.cs
```

This is evidence that script lookup/execution/networking are first-class client facilities, not reverse-engineered hacks.

## GUI bridge surface

`FGStudio.LuaSystem.API.LuaSystemAPI_GUI`:

```text
MainCallUI(uiName,alwaysOnTopIfNoParent,args) 0x06000851
MainFindUI(uiName)                           0x06000852
MainFindAllUIs(uiName)                       0x06000853
CallUI(uiName,args)                          0x06000854
CallUIAlwaysOnTop(uiName,args)               0x06000855
FindUI(uiName)                               0x06000856
FindAllUIs(uiName)                           0x06000857
```

Recovered Lua uses `GUI.FindUI("AutoFight_Main")`, `GUI.FindUI("GameDialog")`, `GUI.FindUI("NPCShop")`, etc. throughout normal gameplay.

## Script object model

Metadata type `LuaScriptData` stores:

```text
ID
UIName
UIRoot
Script
InitArgs
```

with get/set accessors. This confirms a live UI has a separate script object associated with its UI root and initialization arguments.

## Network bridge

For packet-oriented actions:

```text
FGStudio.LuaSystem.API.LuaSystemAPI_Network.SendPacket(packetID,data)
 token 0x0600087A
```

and lower layers:

```text
LuaSystemManager.SendPacketToServer
TCPGame.SendPacket
TCPOutPacket.MakeTCPOutPacket
```

For exact packet actions already source-solved (Revive, Sell, GameDialog, Chat, ItemAction), using the game-owned semantic network bridge may be simpler than invoking the original button handler, provided the guest bridge dispatches it on the correct managed/main-thread path and preserves the same guards/state proof.

## Recommended action adapter split

### Direct C# Game API

Use exact static Game APIs for state/movement/combat/item queries where the action is already exposed semantically:

```text
GetFreeBagSpace
GetItemsAtSite
GetNPCPosition
GoTo
ClickNPC
SelectTarget
ChaseTarget
RequestUsingSkillWithTarget
```

### Exact packet producer

For actions whose Lua source is just validation + `Network.SendPacket`, replicate the verified semantic request **after preserving the original guards**:

```text
Revive
Sell
GameDialog selection
Chat
Item Use/Abandon/Move/Destroy
```

### Lua script method

Use game-owned Lua script invocation when the desired behavior is a substantial stateful coroutine/engine rather than one packet, especially:

```text
AutoFight_Main:StartAutoFight(Train)
```

The built-in Train engine owns coroutines, target state, ignored lists, pickup state, radius and comeback state. Reimplementing it as a packet call is incorrect.

## Runtime proof still required

Static metadata proves the APIs exist, but not the exact external call ABI/lifetime. Before production:

1. resolve `LuaSystemManager`/GUI/network types in the guest process;
2. prove a harmless managed/main-thread callback;
3. acquire one live script/UI object and inspect lifetime across UI/map transitions;
4. prove one harmless method/query or reversible action;
5. never retain script/UI object pointers after `WorldGeneration` changes;
6. compare direct `Network.SendPacket` versus Lua-method invocation behavior for required guards/coroutines.

## Avoid

- injecting arbitrary Lua text when a loaded game script/action already exists;
- holding a `GameDialog`, `NPCShop` or AutoFight script object across scene/reconnect generations;
- invoking stateful Train by sending only its downstream combat packets;
- calling Unity/Lua mutation from an arbitrary guest worker thread;
- assuming `ExecuteFunction(functionName,args)` can invoke `object:method` syntax without a live runtime proof of its accepted naming/closure semantics.

The last point is intentionally left `RUNTIME-PROOF`: method-name presence alone does not establish the exact external invocation contract.
