# Context Pack — Build Runtime Scanner

## Required

- `research/VERIFIED_APK_SNAPSHOT.md`
- `analysis/01_IL2CPP_RUNTIME_METADATA.md`
- `analysis/18_RUNTIME_ROLE_BAG_SNAPSHOT.md`
- `analysis/26_LUA_PACKET_EVENT_OBSERVER.md`
- `analysis/25_ANDROID_MANIFEST_PROCESS_BINDING.md`
- `database/AUTO_TOOL_API_CATALOG.md`
- `database/RUNTIME_SNAPSHOT_SCHEMA.md`

## First implementation target

Per ADB serial/current Android PID, publish one immutable snapshot containing at minimum:

```text
SessionId / AndroidGamePID
ResolverGeneration / WorldGeneration / SnapshotVersion
RoleID / RoleName
HP / MaxHP / MP / MaxMP / IsDeath
MapID / Position / IsMapReady
CurrentMoveDestination / AutoPathing
SelectedTarget summary
FreeBagSpace
BagVersion + BagItems when needed
DialogGeneration + copied Selections summary
ShopGeneration + current NpcShopID/ShopID/IsGuildShop
RevivalGeneration
Captcha/safety state
```

## Event-driven invalidation

Use core/Lua event observation to invalidate/rescan transient state:

```text
RemoveItem -> site:dbID:position
UpdateItemsList -> bag stale/rescan
CMD_SHOW_GAMEDIALOG -> copy current dialog data
CMD_NPC_SHOP_DATA -> copy current shop data
CMD_REVIVE_DATA -> copy Revival lifecycle
NewCaptcha -> safety pause signal
```

Do not retain GameDialog/NPCShop/Lua script object pointers as Windows-host state.

## Proof

Run repeatedly while:

- moving inside one map;
- crossing maps;
- opening/closing NPC dialogs and shop;
- changing bag contents;
- dying/reviving when safe to test;
- running at least two LD9 instances in different states.

Pass only when no stale/cross-instance data survives the relevant generation changes.

Do not implement mutable combat/sell/revive until this read-only layer and generation invalidation are stable.
