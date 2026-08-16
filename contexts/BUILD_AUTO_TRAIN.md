# Context Pack — Build Auto Train

## Required

- `features/AUTO_TRAIN.md`
- `analysis/02_LUA_GAME_UI_NETWORK_API.md`
- `analysis/03_WORLD_ENTITY_MAP_PATH.md`
- `database/AUTO_TOOL_API_CATALOG.md`
- `database/AUTO_TOOL_ACTION_CATALOG.md`
- `contexts/BUILD_MAINTHREAD_BRIDGE.md` once mutable actions begin

## Do not assume

Do not assume PC `AutoFight_Main:StartAutoFight` exists on mobile until recovered.

## Minimum viable semantic Train

Use fresh enemy/target/map/death state and one select/chase/skill action at a time. Yield immediately to revive or return-to-map state.
