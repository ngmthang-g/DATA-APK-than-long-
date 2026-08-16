# Mobile packet layers — low-level core commands vs Lua gameplay protocol

Status: **VERIFIED STATIC EVIDENCE**

## Critical distinction

The mobile client contains at least two command-number spaces that must not be mixed.

### Layer A — C#/engine `TCPGameServerCmds`

Recovered from IL2CPP metadata default enum values. Examples:

```text
CMD_LOGIN               1
CMD_ENTER_MAP           6
CMD_PING_CHECK          7
CMD_CHANGE_MAP         16
CMD_REMOVE_ITEM        25
CMD_UPDATE_ITEMS_LIST  26
CMD_CLICK_OBJECT       27
CMD_USE_SKILL          36
CMD_OBJECT_DEATH       41
CMD_ITEM_PACK          48
CMD_AUTO_PATH          49
CMD_REVIVE             80
CMD_MOVE_TO_LOCATION   83
CMD_CHAT_DATA          89
CMD_CAPTCHA             96
CMD_CLIENT_LUA         100
```

These IDs describe the engine/core TCP event surface and processors such as `ProcessObjectDeath`, `ProcessObjectRevive`, `ProcessChatData`, `ProcessRemoveItem` and `ProcessUpdateItemsList`.

### Layer B — Lua gameplay/server packet table `G_TCPPacketDefine`

Recovered directly from decrypted `TCPPacketDefine.txt`:

```text
CMD_ITEM_ACTION              100005
CMD_BAG_SORT                 100006
CMD_SHOW_GAMEDIALOG          100007
CMD_CLIENT_CHAT              100008
CMD_NPC_SHOP_DATA            200034
CMD_NPC_SHOP_BUY_REQUEST     200035
CMD_NPC_SHOP_SELL_REQUEST    200036
CMD_OTHER_ROLE_COMMAND       200051
CMD_OPEN_STORAGE             200052
CMD_TEAM_ACTION              200057
CMD_REVIVE_DATA              200063
```

This independently proves that many high-level gameplay packet IDs previously known from PC are also present in the mobile Lua layer.

## Why both `CMD_REVIVE=80` and `CMD_REVIVE_DATA=200063` exist

They belong to different command layers. The Lua UI sends `CMD_REVIVE_DATA=200063`; the native/engine network layer separately exposes a low-level `CMD_REVIVE=80` event and `ProcessObjectRevive` lifecycle.

Tool logs/catalogs should preserve:

```text
Action request: Lua gameplay packet
Server/core lifecycle: engine TCP command/event
```

## Exact generic send bridge

C#/IL2CPP exposes:

```text
FGStudio.LuaSystem.API.LuaSystemAPI_Network.SendPacket(packetID,data)
FGStudio.Engine.Network.TCPGame.SendPacket(packetID,data)
TCPOutPacket.MakeTCPOutPacket(...)
```

Recovered Lua calls `Network.SendPacket(...)` for Revive, Shop, GameDialog and Chat.

## Mutation safety

Never invoke client response handlers (`ProcessRemoveItem`, `ProcessObjectRevive`, etc.) as if they were requests. They are proof/lifecycle consumers, not action producers.
