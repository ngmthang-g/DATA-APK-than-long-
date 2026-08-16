# Master Research Map — Mobile APK

## Research priority

```text
P0 runtime state + safe bridge
P1 revive / sell / map return
P1 multi-LD isolation
P2 built-in Train donor
P2 asset/Lua extraction only for unresolved semantic gaps
```

## Subsystem map

| Area | Canonical document | Automation value |
|---|---|---|
| IL2CPP/runtime/metadata | `01_IL2CPP_RUNTIME_METADATA.md` | resolver foundation |
| Lua/Game/Network bridge | `02_LUA_GAME_UI_NETWORK_API.md` | semantic actions + packet tracing |
| world/map/path/NPC | `03_WORLD_ENTITY_MAP_PATH.md` | return map/vendor/navigation |
| inventory/items/shop | `04_INVENTORY_ITEMS_SHOP.md` | bag full + safe sell |
| Windows EXE ↔ LD9 guest | `05_LD9_HOST_GUEST_ARCHITECTURE.md` | actual product architecture |
| security/emulator surface | `06_SECURITY_EMULATOR_SURFACE.md` | stability/risk boundaries |
| MainThread | `07_MAIN_THREAD_DISPATCHER.md` | safe mutable action execution |
| PC/mobile relationship | `08_PC_MOBILE_CROSSWALK.md` | reuse semantic research safely |
| encrypted assets | `09_ASSETS_ENCRYPTION_BUNDLES.md` | fallback semantic extraction |

## Research anti-patterns

Do not:

- scan the whole 90 MB `libil2cpp.so` for every feature;
- copy PC offsets;
- treat metadata name presence as proof of exact object ownership/payload;
- reverse voice/rendering/cosmetics without a concrete auto dependency;
- make screen-coordinate macro the primary engine;
- treat manual visible success after a sleep as internal success proof.

## Preferred question decomposition

Every feature question should be reduced to:

```text
What state do I need?
Where is semantic state exposed?
What is the smallest action that changes it?
What thread/context owns that action?
What observable result proves success?
What invalidates cached state?
```
