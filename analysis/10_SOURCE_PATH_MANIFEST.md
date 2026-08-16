# Phase 2 — Original C# source-path manifest recovered from mobile IL2CPP metadata

Status: **VERIFIED STATIC EVIDENCE**

## Why this matters

The mobile `global-metadata.dat` does not only expose class/method/field names. It also preserves a large set of original C# source file paths from the build. This gives a much stronger subsystem map than the initial symbol-only pass and makes targeted reverse engineering far more efficient.

Recovered from the uploaded `ThanLongMobile_2024.apk` snapshot:

- **1,186 unique original `.cs` source-path strings**
- **537** under `Assets/Scripts/Game`
- **499** under `Assets/Scripts/Engine`
- **149** under `Assets/Scripts/LuaSystem`
- plus `MainGame.cs`

These paths are evidence about the original source organization. They do **not** mean the C# source code itself is present in plaintext.

## High-value automation source paths

### Auto path / movement

```text
Assets\Scripts\Engine\Logic\AutoPath\AutoPathManager.cs
Assets\Scripts\Engine\Logic\AutoPath\AutoPathManager_API.cs
Assets\Scripts\Engine\Logic\AutoPath\AutoPathManager_Core.cs
Assets\Scripts\Engine\Logic\AutoPath\AutoPathManager_Define.cs
Assets\Scripts\Engine\Logic\AutoPath\AutoPathManager_Load.cs
Assets\Scripts\Engine\Logic\PathFinder\PathFinder.cs
Assets\Scripts\Engine\Logic\PathFinder\PathFinder_API_Check.cs
Assets\Scripts\Engine\Logic\PathFinder\PathFinder_API_Core.cs
Assets\Scripts\Engine\Logic\PathFinder\PathFinder_API_Get.cs
Assets\Scripts\Engine\Logic\PathFinder\PathFinder_Data.cs
Assets\Scripts\Engine\Logic\PathFinder\PathFinder_Define.cs
Assets\Scripts\Engine\Logic\PathFinder\PathFinder_Utilities.cs
```

### Combat / target / skill

```text
Assets\Scripts\Engine\Logic\Manager\Combat\CombatManager.cs
Assets\Scripts\Engine\Logic\Manager\Combat\CombatManager_API.cs
Assets\Scripts\Engine\Logic\Manager\Combat\CombatManager_Cooldown.cs
Assets\Scripts\Engine\Logic\Manager\Combat\CombatManager_Relationship.cs
Assets\Scripts\Engine\Logic\Manager\Combat\CombatManager_UseSkill.cs
Assets\Scripts\Engine\Logic\Manager\Combat\CombatManager_Utilities.cs
Assets\Scripts\Engine\Objects\Scene\GScene_SelectTarget.cs
Assets\Scripts\LuaSystem\Game\LuaSystemAPI_Game_AutoFight_API.cs
Assets\Scripts\LuaSystem\Game\LuaSystemAPI_Game_Action.cs
Assets\Scripts\LuaSystem\Game\LuaSystemAPI_Game_Skill.cs
```

### Inventory / bag / item

```text
Assets\Scripts\Engine\Logic\Manager\Item\ItemManager.cs
Assets\Scripts\Engine\Logic\Manager\Item\ItemManager_API.cs
Assets\Scripts\Engine\Logic\Manager\Item\ItemManager_Data.cs
Assets\Scripts\Engine\Objects\Scene\GScene_ItemPack.cs
Assets\Scripts\Game\Components\ItemPack\ItemPack.cs
Assets\Scripts\Game\Logic\Session\SessionData_ItemPack.cs
Assets\Scripts\Game\Network\Data\ItemPack\GC_ItemPackData.cs
Assets\Scripts\Game\Network\Data\ItemPack\GC_WrapItemPackData.cs
Assets\Scripts\Game\Network\TCPGameEventProcessor_ItemPack.cs
Assets\Scripts\Game\Objects\ItemPack\GItemPack.cs
Assets\Scripts\LuaSystem\Game\LuaSystemAPI_Game_Item.cs
Assets\Scripts\LuaSystem\Game\LuaSystemAPI_Game_ItemPack.cs
Assets\Scripts\LuaSystem\SharedData\Item\LuaItemData.cs
```

### NPC / scene

```text
Assets\Scripts\Engine\Objects\Scene\GScene_NPC.cs
Assets\Scripts\Game\Components\NPC\NPC.cs
Assets\Scripts\Game\Components\NPC\NPC_UI.cs
Assets\Scripts\Game\Entities\NPC\NPCTemplateData.cs
Assets\Scripts\Game\Logic\Session\SessionData_NPC.cs
```

### Network / packets

```text
Assets\Scripts\Engine\Network\TCPGame\TCPGame.cs
Assets\Scripts\Engine\Network\TCPGame\TCPGame_API.cs
Assets\Scripts\Engine\Network\TCPGame\TCPGame_API_Packet.cs
Assets\Scripts\Engine\Network\TCPGame\TCPGame_Define.cs
Assets\Scripts\Engine\Network\TCPGame\TCPGame_Event.cs
Assets\Scripts\Engine\Network\TCPOutPacket\TCPOutPacket.cs
Assets\Scripts\LuaSystem\LuaSystemManager_Network.cs
Assets\Scripts\LuaSystem\Network\LuaSystemAPI_Network.cs
```

### Main-thread execution

```text
Assets\Scripts\Engine\Utilities\Threading\MainThread\MainThread.cs
Assets\Scripts\Engine\Utilities\Threading\MainThread\MainThread_API.cs
Assets\Scripts\Engine\Utilities\Threading\MainThread\MainThread_Core.cs
Assets\Scripts\Engine\Utilities\Threading\MainThread\MainThread_Define.cs
Assets\Scripts\Game\Logic\PCSysnc\UnityMainThreadDispatcher.cs
```

### Captcha safety

```text
Assets\Scripts\Game\Network\Data\Captcha\GC_CaptchaData.cs
Assets\Scripts\Game\Network\TCPGameEventProcessor_Captcha.cs
Assets\Scripts\LuaSystem\Game\LuaSystemAPI_Game_Captcha.cs
```

## LuaSystem game API segmentation

The original project split the Lua game bridge into many partial source files, including:

```text
LuaSystemAPI_Game.cs
LuaSystemAPI_Game_Ability.cs
LuaSystemAPI_Game_Action.cs
LuaSystemAPI_Game_Allies.cs
LuaSystemAPI_Game_AutoFight_API.cs
LuaSystemAPI_Game_Buff.cs
LuaSystemAPI_Game_Captcha.cs
LuaSystemAPI_Game_Item.cs
LuaSystemAPI_Game_ItemPack.cs
LuaSystemAPI_Game_Moving.cs
LuaSystemAPI_Game_PlayerData.cs
LuaSystemAPI_Game_Scene.cs
LuaSystemAPI_Game_Skill.cs
LuaSystemAPI_Game_Task.cs
LuaSystemAPI_Game_Vision.cs
```

This strongly supports using `LuaSystemAPI_Game` as a high-level semantic bridge for automation research rather than treating it as one opaque class.

## Packet command-name surface recovered

At least **116 unique `CMD_*` command-name strings** were recovered from metadata. Auto-relevant names include:

```text
CMD_AUTO_PATH
CMD_CHANGE_MAP
CMD_CLICK_OBJECT
CMD_CLIENT_LUA
CMD_ENTER_MAP
CMD_ITEM_PACK
CMD_MOVE_TO_LOCATION
CMD_OBJECT_DEATH
CMD_REMOVE_ITEM
CMD_REVIVE
CMD_SWAP_ITEMS
CMD_UPDATE_ITEM
CMD_UPDATE_ITEMS_LIST
CMD_USE_SKILL
CMD_CAPTCHA
```

Important boundary: command **names** are VERIFIED here. Exact numeric IDs and request payloads must be proven independently for this mobile snapshot; do not inherit PC numbers automatically.

## New research consequence

The mobile snapshot is substantially richer than the first-pass documentation implied. Future work should route by original source file family first, then resolve matching IL2CPP types/methods, then trace exact network/action behavior only where needed.

Recommended next narrow reverse targets:

1. `LuaSystemAPI_Game_AutoFight_API.cs` method inventory and call targets;
2. ItemPack event processor + Lua ItemPack bridge;
3. `TCPGame_API_Packet.cs` and packet enum/value recovery;
4. revive request producer around `CMD_REVIVE`;
5. exact shop/sell flow, including Lua/client packet construction;
6. MainThread producer patterns suitable for the ARM64 bridge.
