# Feature — Auto Chat / location ping

Status: **STATIC CORE SOLVED; runtime server acceptance/rate limits remain**

## Exact send

```text
CMD_CLIENT_CHAT = 100008
packet object:
  RoleID
  Name
  Content = Base64(text)
  Channel
```

## Location ping

Build from the session's own snapshot:

```text
mapID = RoleData.MapID
grid = WorldToGridPosition(RoleData.Position)
text += "@GOTO_" + mapID + "_" + gridX + "_" + gridY
```

Then send through ordinary chat.

## Tool UX recommendation

Per LD9 profile:

- channel dropdown;
- text template;
- button `Gửi chat`;
- button `Ping vị trí`;
- optional scheduled/triggered messages with conservative cooldown;
- no keyboard/window-focus dependency.

Do not attempt to evade server chat limits or moderation.
