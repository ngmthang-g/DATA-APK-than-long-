# Runtime snapshot sources for the LD9 tool

Status: **STATIC DATA CONTRACT STRONG; live object acquisition still needs proof**

## RoleData

`FGStudio.Game.Network.Data.RoleData` has 199 methods / 99 fields in this mobile snapshot. High-value getters include:

```text
RoleID
RoleName
FactionID
Level
MapID
PosX
PosY
Money
BoundMoney
CurrentHP
MaxHP
CurrentMP
MaxMP
TeamID
TeamLeaderRoleID
MoveSpeed
IsRiding
QuickSkills
AutoSettings
LastF1SkillID
```

`Game.RoleData` is the high-level Lua bridge path used by shipped Lua.

## Death truth

Useful independent sources:

```text
LuaLeaderData.IsDeath
GRole.IsDeath
RoleData.CurrentHP
ProcessObjectDeath / ProcessObjectRevive lifecycle
```

Production snapshot should use one canonical local death boolean plus HP rather than infer death from UI visibility.

## Bag truth

`DBItemData` gives live instance records and `ItemSite.Bag=10` identifies the bag. Useful fields:

```text
ID
ItemID
Site
Position
Bound
Quantity
Durability
Properties
Attributes
```

## Recommended immutable per-LD snapshot

```text
SnapshotVersion
WorldGeneration
Timestamp
RoleID / RoleName
MapID
PositionX / PositionY
HP / MaxHP
MP / MaxMP
IsDeath
IsMapReady
IsRiding
CurrentMoveDestination
SelectedTarget
AutoPathing
FreeBagSpace
BagItems[]
CurrentDialog summary
CurrentShop summary
Captcha/safety state
```

Never share managed/native pointers between LD sessions. The Windows host should receive serialized values, not guest object addresses.
