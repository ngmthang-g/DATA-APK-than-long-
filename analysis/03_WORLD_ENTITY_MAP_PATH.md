# World / Entity / Map / Path / NPC — mobile deep map

Status: **STATIC PATH/MOVEMENT SURFACE SOLVED; live route acceptance/timing remains runtime proof**

## Runtime world state

Useful exact metadata/source names:

```text
RoleData.MapID / PosX / PosY / Position
LuaLeaderData.IsDeath
Game.IsMapReady()
Game.GetCurrentMoveDestination()
Game.GetLocalMapObjects()
Game.GetNearbyObjects()
Game.SelectedTarget
Game.GetNearByEnemies()
Game.GetNearbySpritesWithPredicate(...)
```

These support an immutable per-instance snapshot without screen OCR.

## Exact high-level movement signatures

`FGStudio.LuaSystem.API.LuaSystemAPI_Game`:

```text
MoveTo(posX,posY)                                      token 0x0600073F
MoveToEx(posX,posY,moveCompleted,cancelCompleted)     token 0x06000740
GoTo(mapID,posX,posY,moveCompleted)                   token 0x06000741
GetCurrentMoveDestination()                           token 0x06000742
IsMapReady()                                           token 0x06000769
GetNPCPosition(npcID,randomSelect)                    token 0x06000773
```

## Built-in cross-map semantic pattern

Recovered `GoToNPC(mapID,npcID)` demonstrates the shipped route contract:

```text
if current MapID != target map:
    Game.GoTo(targetMap,-1,-1, callback)

then:
    GetNPCPosition(npcID)
    Game.GoTo(targetMap,npcX,npcY, callback)
    Game.ClickNPC(npcID)
```

The normal death-comeback donor also uses `GoTo(map,-1,-1)` for map-only routing in its special infernal-map path. This is strong source evidence that `-1,-1` is intentionally used by client Lua when the immediate goal is to reach the target map without requiring one exact destination coordinate at that step.

## AutoPathManager

`FGStudio.Engine.Logic.AutoPathManager` exact members:

```text
StartAutoPath(toMapCode,toPosX,toPosY)  token 0x06002D5A
StopAutoPath()                          token 0x06002D5B
DoLogic()                               token 0x06002D5C
FinishMoveByPaths                       get/set
IsAutoPathing                           token 0x06002D5F
```

State fields include `currentPaths`, `destPos` and `autoPathType`.

Core TCP producer also exposes:

```text
TCPGameEventProcessor.SendAutoPathRequestChangeMap(toMapID,toPosX,toPosY)
```

Do not call the low-level producer just because it exists if `Game.GoTo`/AutoPath can own the route lifecycle semantically.

## PathFinder

`FGStudio.Engine.Logic.PathFinder` has exact static APIs:

```text
HasPath(fromPos,toPos)
HasPathByGrid(fromGridPos,toGridPos)
CanDirectMove(fromPos,toPos)
GetNearestTeleportLocation(mapID,destPos)
GetCommonPath(fromMapID,fromPos,toMapID)
GetPortalOnlyPath(fromMapID,fromPos,toMapID)
GetPathWithTeleportItem(fromMapID,fromPos,toMapID)
FindPathWithPortalOnly(fromMapCode,toMapCode)
FindPathWithTeleportItem(fromMapCode,toMapCode)
FindPathWithoutTeleportItem(fromMapCode,toMapCode)
FindPath(fromPos,toPos)
GetNPCName(mapID,resID)
```

Internal data explicitly separates:

```text
portalOnlyEdges
commonEdges
teleportItemEdges
pathFinderWithOnlyPortal
pathFinderWithoutTeleportItem
pathFinderWithTeleportItem
```

This proves cross-map travel is not a single blind movement command; the client contains graph/path policies for portals and teleport-item routes.

## Recommended routing policy for external EXE

For normal return-to-train/vendor/healer routes, initially delegate route selection to `Game.GoTo` rather than reconstructing portal graphs externally.

Use PathFinder APIs as:

- precondition/debug information;
- fallback route diagnostics;
- later optimization if a specific route policy needs portal-only or teleport-item control.

Do not run a second movement planner against an active `AutoPathManager`.

## Travel completion proof

```text
requested destination
 -> action accepted / movement begins
 -> map transition(s)
 -> fresh IsMapReady == true
 -> fresh MapID matches expected
 -> current Position valid
 -> distance to final destination <= configured tolerance
```

Also observe `IsAutoPathing` / `GetCurrentMoveDestination` where useful. A fixed six-second sleep is not a map proof.

## Failure handling

Examples:

```text
no progress / destination unchanged for timeout
unexpected MapID after transition
map not ready beyond timeout
path cancelled
character died during route
new higher-priority state (captcha/manual pause)
```

On failure, stop/yield current path, reacquire a fresh world snapshot and re-plan from current state. Do not replay a stale portal/position action decided before a map generation change.

## NPC primitives

Preferred service route:

```text
configured map + npc identity
 -> map-only GoTo when needed
 -> GetNPCPosition(current npcID)
 -> GoTo live NPC position
 -> verify proximity/map-ready
 -> ClickNPC
 -> inspect actual GameDialog/NPCShop state
```

Do not make an NPC's current X/Y a permanent hardcoded truth if runtime lookup exists.

## Train spot contract

Per BotSession save semantic configuration:

```text
TrainMapID
TrainX
TrainY
AllowedRadius
ReturnTolerance
```

After Revive/Sell/Heal completion, return with `Game.GoTo` and only resume Train after fresh snapshot proves the correct map/position.

## World generation invalidation

Invalidate cached object references when any of these occur:

- map change/loading;
- player death/revive transition;
- game process restart;
- disconnect/login scene;
- UI/service transaction that destroys/recreates current managed script objects.

IDs and configured semantic coordinates may persist; raw object/script pointers should not.
