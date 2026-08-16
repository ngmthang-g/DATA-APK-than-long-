# World / Entity / Map / Path / NPC

## Runtime world state

Useful metadata names:

```text
RoleData
MapID
PosX
PosY
Position
IsDeath
IsMapReady
GetCurrentMoveDestination
GetLocalMapObjects
GetNearbyObjects
SelectedTarget
GetNearByEnemies
GetNearbySpritesWithPredicate
```

These support an immutable per-instance snapshot without screen OCR.

## Movement primitives

Verified names:

```text
MoveTo
MoveToEx
GoTo
AutoPathManager
StartAutoPath
StopAutoPath
IsAutoPathing
SendAutoPathRequestChangeMap
```

Expected production proof for travel:

```text
requested destination
 -> action accepted
 -> map transition if needed
 -> IsMapReady == true
 -> fresh MapID matches
 -> fresh Position valid
 -> distance to destination <= tolerance
```

A six-second sleep is not a map proof.

## NPC primitives

Verified names:

```text
GetNPCPosition
ClickNPC
```

Preferred service routing:

```text
configured map + npc identity
 -> resolve current NPC position
 -> GoTo/path
 -> verify proximity/map-ready
 -> ClickNPC
 -> inspect actual dialog/shop state
```

Do not make an NPC's current X/Y a permanent hardcoded truth if runtime lookup exists.

## Train spot contract

Per BotSession save:

```text
TrainMapID
TrainX
TrainY
AllowedRadius
ReturnTolerance
```

On recovery/sell completion, return using semantic pathing and only resume combat after fresh snapshot proves the correct map/position.

## World generation invalidation

Invalidate cached object references when any of these occur:

- map change/loading;
- player death/revive transition;
- game process restart;
- disconnect/login scene;
- major UI/service transaction that replaces current managed objects.

IDs and configured semantic coordinates may persist; raw object pointers should not.
