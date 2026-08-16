# Lua packet/event observer — dynamic dialog/shop/item proof layer

Status: **CENTRAL LUA DISPATCH FLOW VERIFIED STATIC; external read-only observation remains RUNTIME-PROOF**

## Central inbound Lua packet function

Recovered `TCPCmdHandler.lua` explicitly documents and defines:

```text
function ReceivePacket(packetID, data)
```

Comment meaning: **called from Core whenever a packet is sent from the server to Lua**.

This function dispatches the high-level gameplay/UI packet space and is therefore a much better source for transient server state than retaining UI pointers.

## High-value packet branches

### Revival

For:

```text
packetID == CMD_REVIVE_DATA (200063)
```

it treats `data` as `revivalData`, opens/updates/destroys the `Revival` UI according to its Action, and on initial Open calls:

```text
AutoFight_Main:DeathActive()
```

This is the server/UI lifecycle complement to the outbound normal revive request `200063:"1"`.

### NPCShop

For:

```text
packetID == CMD_NPC_SHOP_DATA (200034)
```

it treats `data` as current `shopData` and either:

```text
GUI.CallUI("NPCShop", shopData)
```

or:

```text
uiNPCShop:RefreshData(shopData)
```

This is the canonical moment to snapshot current `NpcShopID`, shop `ID`, category/guild state and transaction generation.

### GameDialog

For:

```text
packetID == CMD_SHOW_GAMEDIALOG (100007)
```

flow is:

```text
destroy prior GameDialog if present
if data == "NULL": close/no new dialog
otherwise:
    AutoFight_Main:PutGameDialog(data)
    GUI.CallUI("GameDialog", data)
```

Important: `AutoFight_Main:PutGameDialog` retains/auto-processes the dialog only in Quest mode. `GameDialog.lua` itself uses `gameDialogData` during initialization to create buttons and does not expose the original Selections as a durable public field.

Therefore an external treatment/shop observer should copy relevant GameDialog data **at inbound packet time** rather than assume `GUI.FindUI("GameDialog")` can always return the server Selection map later.

Recommended snapshot fields:

```text
DialogGeneration
Title
Message (decoded/normalized as appropriate)
Selections[] { selectionID, visibleText }
Awards summary when relevant
ReceivedAt
```

Do not retain the server object pointer after close/world/UI generation changes.

## Central core-to-Lua event function

Recovered `TCPCmdEventHandler.lua` defines:

```text
function ReceiveEvent(eventType, data)
```

and `TCPPacketDefine.txt` independently defines `G_TCPEventType` values.

Automation-relevant event IDs:

```text
UpdateMoney                 0
AddItem                     1
RemoveItem                  2
UpdateItemsList             3
BeginProgress              12
InteruptProgress           13
UpdateProgressTime         14
AddBuff                    15
UpdateBuff                 16
RemoveBuff                 17
OpenPickUpItemFromItemPack 18
ClosePickUpItemFromItemPack19
UpdateTeamData             47
ChatEvent                  50
PKWarning                  51
NewCaptcha                 57
```

## Exact RemoveItem proof

For `G_TCPEventType.RemoveItem`, stock Lua parses:

```text
site:dbID:position
```

as:

```text
site     = fields[1]
dbID     = fields[2]
position = fields[3]
```

For Bag it updates the bag grid using the live `dbID`.

This is an excellent proof for Sell/Abandon/Use/Destroy mutations when combined with a fresh `GetItemsAtSite(Bag)` rescan.

## UpdateItemsList proof

`G_TCPEventType.UpdateItemsList` forces all bag/storage grids to refresh. It is a useful invalidation signal: after receiving it, mark cached bag snapshot stale and rebuild from semantic APIs.

## Chat event

`G_TCPEventType.ChatEvent = 50` identifies the core-to-Lua chat lifecycle. This can be observed for user-facing confirmation/debugging, but outbound Auto Chat should not rely on seeing an echo if a channel/server policy does not echo the sender identically.

## Captcha safety

`NewCaptcha = 57` is a high-priority safety event. External orchestration should pause mutable auto behavior and surface manual handling. This observer is for detection/safety only; no captcha bypass belongs in the tool.

## Recommended guest observer architecture

Read-only event mirroring:

```text
Core/Lua inbound packet or event
 -> copy only semantic fields needed by tool
 -> assign transaction/event generation
 -> publish immutable guest snapshot/event record to Windows host
 -> allow original client handler to continue normally
```

Do not replace/fake client response handlers to make actions appear successful.

## Runtime proof boundary

Static source establishes central dispatcher semantics, but production instrumentation must prove a non-invasive observation point that preserves original execution/order. Candidate surfaces include game-owned `LuaSystemManager.OnReceivePacket` / `OnReceiveEvent` and associated Lua dispatch, but exact external hook/callback ABI must be validated on LD9 before use.
