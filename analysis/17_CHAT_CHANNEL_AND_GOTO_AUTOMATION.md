# Chat, channel selection and coordinate ping automation

Status: **OUTBOUND CHAT + @GOTO FORMAT VERIFIED STATIC; external scheduling/orchestration required**

## Exact chat packet

Recovered Lua packet table:

```text
CMD_CLIENT_CHAT = 100008
```

`ChatBox:ButtonSendMessageClicked()` builds:

```text
RoleID  = private target RoleID (unused for normal channels)
Name    = private target name
Content = String.ToBase64(content)
Channel = actual channelID
```

and sends it through `Network.SendPacket(100008, packetData)`.

An internal tool therefore does not need to focus the LD window or synthesize keyboard input for ordinary chat.

## UI dropdown index is NOT the server channel ID

Stock source first reads:

```text
selectedChannelIndex = SelectChatChannelDropdown.SelectedID
```

Then it iterates `C_ChatChannels` entries where `CanChat=true` and translates that UI index to:

```text
channelID = channelData.Channels[1]
```

Only this final `channelID` goes into the packet.

Therefore an external EXE should store/use **actual `C_ChatChannel` IDs**, not a ChatBox dropdown index or the legacy AutoSettings `ChatSelect` value unless that UI preference has been explicitly translated.

## No stock Auto-Chat sender found

Corpus-wide search of the 631 recovered Interface TextAssets finds `CMD_CLIENT_CHAT` only in:

```text
TCPPacketDefine.txt
ChatBox.lua
```

`AutoFight_Main` stores utility fields such as:

```text
ChatSelect
ChatCostumeChannel
ChatSelectSend
```

but those functions are channel/custom-channel/last-selection persistence helpers. No periodic or event-driven `CMD_CLIENT_CHAT` send loop was found in the recovered stock AutoFight source.

Therefore **Auto Chat is an external EXE orchestration feature**, not a reliable built-in auto toggle. The EXE can still reuse the exact semantic packet producer.

## Actual channel IDs

```text
Default          -1
System            0
System_Broad_Chat 1
Guild             2
Allies            3
Team              4
Near              5
Faction           6
Private           7
Global            8
Special           9
CrossServer      10
```

`C_ChatChannels` separately marks which UI channel groups are actually sendable (`CanChat=true`). Normal sendable groups include Special, Near, Private, Team, Global, Guild, Allies, Faction and CrossServer. Aggregate display groups such as All/System/Custom are not direct send channels.

## Exact coordinate ping format

`ChatBox:ButtonSendLocationClicked()` reads current MapID and converts current world position to grid position, then appends exactly:

```text
@GOTO_<MapID>_<GridX>_<GridY>
```

The text then goes through ordinary chat as Base64 content.

The shipped ChatBox input has:

```text
CharacterLimit = 200
```

The location button refuses to append the `@GOTO_...` token when current text + token would exceed this client UI limit. An external semantic sender should preserve the same practical 200-character envelope unless runtime evidence shows the server safely accepts another limit.

## Other stock semantic attachment tokens

Before Base64 conversion, stock source can append:

```text
@ITEM_<live item DBID>
@PET_<pet DBID>
```

These are useful protocol evidence but are not needed for basic Auto Chat/Ping. Do not attach stale item/pet IDs across session/bag generations.

## Clickable navigation on receiver

Chat rendering recognizes generated rich-text links matching:

```text
<link="GoTo_<mapID>_<gridX>_<gridY>">
```

`GF_SetChatAction` parses the values, converts grid coordinates with:

```text
Game.GridToWorldPosition(gridX,gridY)
```

and registers a click callback:

```text
Game.GoTo(mapID,worldX,worldY)
```

The raw sender token is `@GOTO_...`; the richer `<link="GoTo_...">` form is produced before/display-side handling. This proves location ping is a semantic client feature, not a screenshot/mouse trick.

## Private chat guard

For actual channel `Private=7`, stock source requires `G_PrivateChatData.RoleID != -1`. An external private-message action must supply the intended live RoleID/name and should not reuse a stale private target from another session.

## Local cooldown finding

The recovered `ButtonSendMessageClicked()` performs channel/content/private-target validation, optional item/pet attachment, Base64 conversion and immediately calls `Network.SendPacket`. No explicit client-side time/cooldown limiter is present in that send function.

This **does not prove there is no server-side anti-spam/cooldown**. Treat server acceptance, throttling and permission errors as runtime behavior. The EXE should implement its own conservative per-session rate limiter rather than sending as fast as the bridge permits.

## EXE design

Per LD9 session expose:

```text
SendChat(actualChannelID,text,privateRoleID?,privateName?)
SendLocationPing(actualChannelID,optionalPrefixText)
```

The Windows host may schedule periodic messages or trigger messages from state-machine events (death, vendor trip, spot switch, user hotkey), but every send should still pass through the per-session action/rate gate.

No Windows focus/keyboard dependency is needed. Do not attempt to bypass server chat limits or moderation.
