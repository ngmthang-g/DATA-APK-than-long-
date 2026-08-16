# Context Pack — Build Runtime Scanner

## Required

- `research/VERIFIED_APK_SNAPSHOT.md`
- `analysis/01_IL2CPP_RUNTIME_METADATA.md`
- `analysis/03_WORLD_ENTITY_MAP_PATH.md`
- `analysis/04_INVENTORY_ITEMS_SHOP.md`
- `database/AUTO_TOOL_API_CATALOG.md`
- `database/RUNTIME_SNAPSHOT_SCHEMA.md`

## First implementation target

Read-only snapshot containing:

```text
session/process generation
character identity if available
HP/MaxHP/IsDeath
MapID/Position/IsMapReady
selected target summary
nearby enemy summary
GetFreeBagSpace
bag item summary only when requested/needed
```

## Proof

Run repeatedly while moving and changing map. Verify no stale cross-instance data.

Do not implement mutable combat/sell/revive until this layer is stable.
