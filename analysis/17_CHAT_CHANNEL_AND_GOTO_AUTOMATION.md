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
Channel = channelID
```

and sends it through `Network.SendPacket(100008, packetData)`.

An internal tool therefore does not need to focus the LD window or synthesize keyboard input for ordinary chat.

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

## Channel IDs

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

## Clickable navigation on receiver

Chat rendering recognizes a `GoTo_<map>_<x>_<y>` link, converts grid coordinates back to world coordinates and registers a click callback that calls `Game.GoTo(mapID,worldX,worldY)`.

This proves location ping is a semantic client feature, not a screenshot/mouse trick.

## Local cooldown finding

The recovered `ButtonSendMessageClicked()` performs channel/content/private-target validation, optional item/pet attachment, Base64 conversion and immediately calls `Network.SendPacket`. No explicit client-side time/cooldown limiter is present in that send function.

This **does not prove there is no server-side anti-spam/cooldown**. Treat server acceptance, throttling and permission errors as runtime behavior. The EXE should implement its own conservative per-session rate limiter rather than sending as fast as the bridge permits.

## EXE design

Per LD9 session expose:

```text
SendChat(channel,text,privateRoleID?,privateName?)
SendLocationPing(channel,optionalPrefixText)
```

The Windows host may schedule periodic messages or trigger messages from state-machine events (death, vendor trip, spot switch, user hotkey), but every send should still pass through the per-session action/rate gate.

No Windows focus/keyboard dependency is needed. Do not attempt to bypass server chat limits or moderation.
