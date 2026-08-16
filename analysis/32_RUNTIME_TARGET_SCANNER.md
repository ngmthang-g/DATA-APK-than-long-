# Runtime target / nearby sprite scanner — mobile

Status: **RETURN DATA MODEL + SHIPPED USAGE VERIFIED STATIC; live repeated scanner remains runtime proof**

Purpose: support Train debug/status, target selection and future nearby-player features without CE/OCR.

---

# 1. Base map object

Metadata:

```text
FGStudio.LuaSystem.SharedData.LuaMapObjectData
```

fields/accessors:

```text
RoleID
Name
Type
Position
```

This is the object used by map/minimap scans.

---

# 2. Sprite data extends the map-object model

Metadata:

```text
FGStudio.LuaSystem.SharedData.LuaMapSpriteData
```

adds/exposes:

```text
IsDeath
Distance(...)
ResID
AvartaID
FactionID
Level
HP
MaxHP
CreateTime
GuildName
TeamRank
```

The class only declares four additional backing fields (`ResID`, `FactionID`, `Level`, `CreateTime`) while exposing inherited map-object properties plus calculated/runtime getters such as HP/MaxHP/IsDeath. Operationally the returned sprite carries the combined object/sprite schema needed by Lua.

Recommended copied row:

```text
RoleID
ResID
Name
Type
WorldPosition
GridPosition
IsDeath
HP
MaxHP
HPPercent? = tool derived when MaxHP>0
Level
FactionID
GuildName
TeamRank
AvartaID
CreateTime
DistanceFromPlayer / DistanceFromTrainCenter
SnapshotVersion
```

---

# 3. Nearby peaceful players — exact shipped UI proof

`MainUI_NearByPlayers_PlayersTab.lua` calls:

```text
Game.GetNearByPeacePlayers(MaxPlayers)
```

and directly uses:

```text
playerData.Name
playerData.Level
playerData.FactionID
playerData.MaxHP
playerData.HP
playerData.GuildName
playerData.AvartaID
playerData.TeamRank
playerData.RoleID
```

Selecting a row calls:

```text
Game.SelectTarget(playerData.RoleID)
```

Therefore the mobile client independently proves nearby peaceful-player HP/MaxHP and identity are semantic API data, not something requiring party membership, OCR or a CE pointer scan.

For any future buff feature, do not rediscover these fields through screen/memory heuristics first.

---

# 4. Train monster scan — exact stock predicate

`AutoFight_Main:FindBestTarget()` calls:

```text
Game.GetNearbySpritesWithPredicate(predicate, RolePosition)
```

The predicate reads:

```text
MapObject.Type
MapObject.IsDeath
MapObject.RoleID
MapObject.ResID
MapObject.Position
```

and applies:

```text
Type == "Monster"
not dead
not in IgnoredTarget
optional CurrentMonsterNeedKill.ResID match
within RangerScan around train center/current position
optional whitelist AttackMonsterList
optional lure/banned-target state
```

This is the canonical minimum Train target schema.

---

# 5. Monster HP is also available

After `FindBestTarget()` returns the chosen sprite, stock Train immediately reads:

```text
CurrentTarget.HP
```

and stores it as `LastHpTarget`. It treats:

```text
HP == -1
```

as a stale/dead/unusable target condition and pushes the RoleID into the ignored set.

When retaining a target between loop iterations it calls:

```text
Game.ReloadTarget(CurrentTarget.RoleID)
```

to refresh target state rather than trusting the old object indefinitely.

This proves the tool can expose current monster HP/MaxHP for debug/policy if desired, but production Train does not need to poll every monster's absolute HP just to pick the next target.

---

# 6. Selected target schema

Metadata:

```text
FGStudio.LuaSystem.SharedData.SelectedTargetData
```

exposes:

```text
RoleID
Type
MonsterBelongState
Avarta
Name
Level
TeamID
GroupID
GuildID
AlliesID
IsAlliesHost
GuildRank
TeamLeaderRoleID
FactionID
HPPercent
MPPercent
RagePercent
EnergyPercent
```

Use selected-target data for UI/status and interactions that depend specifically on the currently selected entity. Do not substitute it for the full nearby-sprite list.

---

# 7. Recommended EXE Train debug panel

Per LD session:

```text
Current target:
  RoleID
  ResID
  Name
  HP / MaxHP
  Type
  Distance
  Position grid

Nearby monsters (debug optional):
  ResID / Name / HP / distance / IsDeath

Train center:
  MapID / grid X,Y / RangerScan

Ignored target count / current reason
```

This makes Train failures diagnosable without screen recording.

---

# 8. Scanner freshness

Nearby entities are inherently volatile. Do not persist RoleID/object pointers across map/world generations.

For a selected target:

```text
RoleID from current world generation
 -> ReloadTarget when needed
 -> target gone/dead/stale -> clear/ignore and rescan
```

For the Windows host, copy values only. The guest owns all live object resolution.

---

# 9. What NOT to overbuild

The stock Train already proves it can operate with:

```text
Type
IsDeath
RoleID
ResID
Position
HP for current-target progress/stale detection
```

Do not make full HP/MaxHP scans of every monster a prerequisite for first Train implementation unless a concrete feature policy needs them. Keep the read-only observer rich for diagnostics but the action loop minimal.
