# Coordinate domains — displayed grid coordinates vs world/API coordinates

Status: **SEMANTIC CONVERSION API VERIFIED STATIC; exact manual arithmetic intentionally not assumed**

This is critical for user-entered training/NPC coordinates. The game displays familiar map coordinates such as `284,188`, while movement APIs and `RoleData.Position` operate in world coordinates.

## Source evidence

Recovered UI consistently converts world positions before displaying them:

```text
Minimap.lua:
  leaderPos = Game.RoleData.Position
  gridPos = Game.WorldToGridPosition(leaderPos)
  display gridPos.X, gridPos.Y

LocalMap_LocalMapTab.lua:
  gridPos = Game.WorldToGridPosition(leaderPos)

BrowsePlayerInfo.lua / FindPlayers.lua:
  gridPos = Game.WorldToGridPosition(playerData.PosX, playerData.PosY)

ChatBox.lua:
  gridPos = Game.WorldToGridPosition(Game.RoleData.Position)
  @GOTO_MapID_GridX_GridY
```

Receiver-side chat navigation performs the inverse:

```text
worldPos = Game.GridToWorldPosition(gridX,gridY)
Game.GoTo(mapID,worldPos.X,worldPos.Y)
```

This proves two coordinate domains exist and the client has canonical conversion functions.

## 32-unit evidence

Stock comeback notification prints saved world death position as:

```text
DieMapLocaltion.X / 32
DieMapLocaltion.Y / 32
```

This is strong evidence that the displayed grid has a 32-world-unit relationship in this client.

However, do **not** promote a guessed manual formula such as:

```text
world = grid * 32
```

into the production contract without exact rounding/center semantics. The canonical conversion APIs already exist and avoid that ambiguity.

## Recommended profile representation

For coordinates entered by the user from the visible map UI, store them explicitly as grid coordinates:

```text
Train.MapID
Train.GridX
Train.GridY
CoordinateKind = Grid
```

At action time inside the guest:

```text
world = Game.GridToWorldPosition(GridX,GridY)
Game.GoTo(MapID,world.X,world.Y,...)
```

For coordinates captured directly from current `RoleData.Position`, store/label them as world coordinates or convert to grid for user-facing persistence/display.

Do not silently mix the two.

## Recommended snapshot

Expose both forms when useful:

```text
PositionWorldX
PositionWorldY
PositionGridX
PositionGridY
```

where grid values are produced through `Game.WorldToGridPosition`, not host arithmetic.

## NPC coordinates

`Game.GetNPCPosition(npcID)` returns the position expected by movement APIs in the semantic client flow. Do not convert it through grid coordinates before `Game.GoTo` unless a UI display needs grid coordinates.

So:

```text
NPC runtime position -> use directly for GoTo
NPC display/log -> WorldToGridPosition for user readability
```

## Chat ping

`@GOTO` specifically carries **grid** coordinates:

```text
@GOTO_<MapID>_<GridX>_<GridY>
```

Never put raw RoleData world X/Y into that token.

## Return tolerance

Distance checks for route completion should operate in the same coordinate domain as `Game.GetDistance`/world positions. A profile tolerance expressed in visible map/grid cells should be converted intentionally rather than compared directly to world-unit distance.

## Tool UX consequence

Recommended labels:

```text
Bãi train: Map / X / Y (tọa độ bản đồ)
Current: 284,188 (grid)
Debug world: 9088,6016 (example only; do not hardcode formula)
```

The debug world value should come from actual runtime conversion/state, not an inferred multiplication shown to the user.

This explicit coordinate typing prevents one of the easiest-to-make bugs in a PC-host/Android-guest automation system: passing visible grid coordinates directly into world-coordinate movement APIs.
