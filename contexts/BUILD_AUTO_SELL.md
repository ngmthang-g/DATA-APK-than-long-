# Context Pack — Build Auto Sell

## Required

- `features/AUTO_SELL.md`
- `analysis/04_INVENTORY_ITEMS_SHOP.md`
- `analysis/03_WORLD_ENTITY_MAP_PATH.md`
- `analysis/02_LUA_GAME_UI_NETWORK_API.md`
- `database/AUTO_TOOL_API_CATALOG.md`
- `database/AUTO_TOOL_ACTION_CATALOG.md`
- `research/AUTO_RUNTIME_PROOF_QUEUE.md`

## Implementation order

1. read-only bag scanner;
2. keep/sell policy preview only;
3. NPC semantic route;
4. current shop readiness capture;
5. one disposable-item manual sell trace;
6. one internal sell request + proof;
7. safe rescan loop;
8. return-to-train.

Do not start with a 90-click loop.
