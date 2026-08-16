# NPC discovery and stable identity — mobile runtime strategy

Status: **RETURN SCHEMAS/APIS VERIFIED STATIC; stable per-NPC identities require runtime/static data verification**

Purpose: support tool setup for Auto Sell / Trị liệu without confusing live scene RoleID with persistent NPC resource ID.

---

# 1. `GetLocalMapObjects()` return schema is exact

Metadata type:

```text
FGStudio.LuaSystem.SharedData.LuaMapObjectData
```

contains exactly:

```text
RoleID
Name
Type
Position
```

with getters/setters and no `ResID`/`NPCID` field.

Recovered `LocalMap_LocalMapTab.lua` uses:

```text
localMapObjects = Game.GetLocalMapObjects()
for _, objData in ipairs(localMapObjects):
    objData.Type
    objData.Name
    objData.Position
```

Known `Type` strings include:

```text
NPC
Monster
GrowPoint
Zone
Portal
```

This is enough to build a useful **scan current map objects** UI.

## What it does NOT prove

`LuaMapObjectData.RoleID` is a live scene/runtime role identity. Do not persist it as the stable NPC resource ID expected by:

```text
Game.GetNPCPosition(npcID)
Game.GetMapNPCName(mapID,npcResID)
Game.GetNearestNPC(npcID)
```

There is no static evidence that RoleID == npcID.

---

# 2. Nearby/live NPC schema

`Game.GetNearestNPC()` is used by stock UI and returns at least:

```text
RoleID
Name
Position
```

`MainUI_BackgroundWork` shows the talk button only when the nearest NPC is within distance <= 100 and stores:

```text
ButtonTalkToNPC.Tag = nearestNPCData.RoleID
```

So `RoleID` is appropriate for immediate live scene target interaction, not stable profile identity.

`Game.GetNearestNPC(npcID)` accepts a configured/stable NPC ID and returns the corresponding live nearby NPC object when present. Stock Auto Quest then uses its live RoleID for `SelectTarget(...,true)`.

---

# 3. Two shipped NPC interaction donors

There are two useful stock patterns.

## A. Generic task description helper — direct `ClickNPC`

Recovered `Global_Functions.lua`:

```text
if wrong map:
    Game.GoTo(mapID,-1,-1, callback)

npcPos = Game.GetNPCPosition(npcID)
Game.GoTo(mapID,npcPos.X,npcPos.Y, callback)
Game.ClickNPC(npcID)
```

This is the cleanest semantic donor for a tool that already knows stable `npcID`.

## B. Auto Quest helper — live RoleID selection

`AutoFight_Main:GoToNPC(mapID,npcID)` adds stock-auto policies:

```text
optional RiderUp
GetNPCPosition(npcID)
GoTo NPC position
nearest = Game.GetNearestNPC(npcID)
if nearest exists and dialog timing guard allows:
    Game.SelectTarget(nearest.RoleID,true)
```

This demonstrates a second interaction path where selecting the live NPC target with the boolean flag triggers the quest/dialog interaction behavior.

For external Sell/Heal, prefer direct semantic `ClickNPC(npcID)` initially because it avoids unnecessary target-state coupling, unless runtime proof shows a particular service requires the live-select path.

---

# 4. Runtime NPC scanner UI

Recommended EXE setup button:

```text
[Quét NPC map hiện tại]
```

Guest calls:

```text
Game.GetLocalMapObjects()
```

and returns only `Type == "NPC"` rows:

```text
LiveRoleID
Name
WorldPosition
GridPosition = WorldToGridPosition(Position)
DistanceFromPlayer
```

This is useful for the user to identify visible NPCs such as a vendor/healer by name and coordinates.

### Important UI label

Call the first identifier:

```text
Live RoleID
```

not `NPC ID`, unless a stable resource ID has separately been verified.

---

# 5. Promoting a PC donor stable NPC ID to mobile runtime-verified

The PC repository has valuable static Config candidate IDs, but base APK resource inspection does not contain the same full current table. Use candidates as hypotheses, then verify through mobile runtime.

For one candidate:

```text
candidateMapID
candidateNpcID
expectedName from PC donor/user knowledge
```

On the running mobile client call:

```text
name = Game.GetMapNPCName(candidateMapID,candidateNpcID)
pos  = Game.GetNPCPosition(candidateNpcID,...)
```

If the current mobile runtime returns a consistent name and usable position for that candidate, record evidence such as:

```text
MOBILE_RUNTIME_VERIFIED
MapID
NpcID
returned Name
returned Position/GridPosition
client/resource version
observed date/session
```

For stronger proof, route near it and require:

```text
GetNearestNPC(candidateNpcID) != nil
returned live Name matches
then open expected current GameDialog/NPCShop service
```

Only after this should the profile persist `NpcID` as a stable action identity for that current client/resource revision.

---

# 6. Vendor verification

For Auto Sell, stable NPC identity alone is not enough. Verify service:

```text
NpcID identity proof
 -> route/interact
 -> optional GameDialog shop selection
 -> inbound CMD_NPC_SHOP_DATA 200034
 -> current IsGuildShop == false
 -> current ShopID/NpcShopID captured
```

Then the NPC can be classified as a runtime-verified normal Sell vendor.

PC donor candidates may be tested in this way without blindly hardcoding their coordinates.

---

# 7. Healer verification

For Trị liệu:

```text
NpcID identity proof
 -> route/interact
 -> current GameDialog copied
 -> treatment text matcher uniquely resolves a selection
 -> selection action
 -> HP/result proof
```

The NPC is not a verified healer merely because its name/archetype looks medical.

---

# 8. Profile fields

Persist only verified semantic identity:

```text
Vendor.MapID
Vendor.NpcID
Vendor.ExpectedName   # proof/display, not action key

HealNPC.MapID
HealNPC.NpcID
HealNPC.ExpectedName
```

Do not persist:

```text
live RoleID
current scene object pointer
current world NPC position as permanent truth
current ShopID/NpcShopID
current GameDialog selectionID
```

These are session/transaction data.

---

# 9. Practical setup workflow

Recommended UI:

```text
Current map: <MapName> (<MapID>)

[Quét NPC hiện tại]
Name              Grid X,Y     Live RoleID
...

Stable NPC candidate:
NpcID [____]
[Kiểm tra ID]
 -> GetMapNPCName
 -> GetNPCPosition
 -> show returned name/grid position

[Thử mở NPC]
 -> semantic route/interact
 -> report GameDialog/NPCShop transaction

[Lưu làm NPC bán đồ]
[Lưu làm NPC trị liệu]
```

The last two buttons should only become available after the intended service transaction has been observed in development/testing mode.

This workflow gives the user the convenience of runtime scanning while preserving the distinction between transient scene identity and stable NPC resource identity.
