# Chat, channel selection and coordinate ping automation

Status: **OUTBOUND CHAT + @GOTO FORMAT VERIFIED STATIC**

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

## Exact coordinate ping format

`ChatBox:ButtonSendLocationClicked()` reads current MapID and converts current world position to grid position, then appends exactly:

```text
@GOTO_<MapID>_<GridX>_<GridY>
```

The text then goes through ordinary chat as Base64 content.

## Clickable navigation on receiver

Chat rendering recognizes a `GoTo_<map>_<x>_<y>` link, converts grid coordinates back to world coordinates and registers a click callback that calls `Game.GoTo(mapID,worldX,worldY)`.

This proves location ping is a semantic client feature, not a screenshot/mouse trick.

## EXE design

Per LD9 session expose:

```text
SendChat(channel,text,privateRoleID?,privateName?)
SendLocationPing(channel,optionalPrefixText)
```

Use a rate limiter and one action gate. Server-side cooldown/permission failures are runtime outcomes; do not attempt to bypass them.
