# Runtime snapshot sources for the LD9 tool

Status: **STATIC DATA CONTRACT STRONG; live object/event acquisition still needs proof**

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
Revival inbound lifecycle through ReceivePacket(200063,data)
```

Production snapshot should use one canonical local death boolean plus HP and record server Revival generation/lifecycle separately rather than infer death from UI visibility.

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

State APIs:

```text
Game.GetFreeBagSpace()
Game.GetItemsAtSite(10)
```

Lifecycle proof/invalidation signals from Lua event layer include:

```text
AddItem event = 1
RemoveItem = 2, data site:dbID:position
UpdateItemsList = 3
```

Never rely on one cached bag instance list across a mutation or UpdateItemsList event.

## Dynamic server/UI transaction state

Current GameDialog/NPCShop/Revival data is transient and should be captured at inbound packet time where possible. Canonical observer map: `analysis/26_LUA_PACKET_EVENT_OBSERVER.md`.

Recommended copies:

```text
DialogGeneration
Dialog.Title
Dialog.Message
Dialog.Selections[] { ID, Text }

ShopGeneration
Shop.NpcShopID
Shop.ShopID
Shop.CategoryName
Shop.IsGuildShop

RevivalGeneration
Revival.Action / relevant enable/time fields available at runtime

LastBagEvent
LastChatEvent
CaptchaActive / CaptchaGeneration
```

Do not send UI/server object references to the Windows host.

## Movement / Train state

Useful semantic sources:

```text
Game.IsMapReady()
Game.GetCurrentMoveDestination()
AutoPathManager.IsAutoPathing
selected target summary
current Auto mode when accessible through the stock service
```

The host needs enough state to distinguish `moving`, `map loading`, `at train spot`, `current transaction owns character` and `stalled` without screen OCR.

## Recommended immutable per-LD snapshot

```text
SessionId
AndroidGamePID
ResolverGeneration
WorldGeneration
SnapshotVersion
Timestamp

RoleID / RoleName
FactionID / Level
MapID
PositionX / PositionY
HP / MaxHP
MP / MaxMP
IsDeath
IsMapReady
IsRiding
CurrentMoveDestination
AutoPathing
CurrentAutoMode?
SelectedTarget summary

FreeBagSpace
BagVersion
BagItems[]

DialogGeneration + CurrentDialog summary
ShopGeneration + CurrentShop summary
RevivalGeneration + Revival summary
Captcha/safety state

LastInboundPacket summary
LastCoreEvent summary
LastActionId / LastActionState / LastProof
```

## Generation rules

### ResolverGeneration

Increment/rebuild when Android game PID or core runtime is recreated.

### WorldGeneration

Increment/invalidate world/UI object references on events such as:

- map/scene transition;
- reconnect/login world recreation;
- major death/revive scene transition where objects are rebuilt.

### Dialog/Shop transaction generation

Increment whenever server replaces/closes/reopens the corresponding transient transaction. A `selectionID`, `NpcShopID`, `ShopID` or Lua UI object from an older generation is stale.

### BagVersion

Increment/invalidate after bag item events/mutations; current `DBItemData.ID` list must come from a fresh semantic scan before the next destructive item action.

## Windows-host boundary

Never share managed/native pointers between LD sessions. The Windows host should receive serialized semantic values and generation IDs, not guest object addresses.

A host decision is valid only against the snapshot version/generation it was computed from. Before execution, the guest action gate should reject the action if its required generation no longer matches current state.
