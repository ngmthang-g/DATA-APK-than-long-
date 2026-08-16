# AI Router — route mobile automation tasks before deep reading

Read `AI_BOOTSTRAP.md` and `AUTO_TOOL_SCOPE.md` first.

| Task | Primary context pack |
|---|---|
| EXE architecture, LD discovery, multi-instance isolation, state machine | `contexts/BUILD_TOOL_CORE.md` |
| Read HP/map/position/death/bag/target state | `contexts/BUILD_RUNTIME_SCANNER.md` |
| Safely execute Unity/IL2CPP mutable calls | `contexts/BUILD_MAINTHREAD_BRIDGE.md` |
| Auto Train / enemy / chase / skill / return spot | `contexts/BUILD_AUTO_TRAIN.md` |
| Death / Đầu thai / return to map | `contexts/BUILD_AUTO_REVIVE.md` |
| Bag full / vendor / item filtering / sell | `contexts/BUILD_AUTO_SELL.md` |
| Train + Revive + Sell + multi-LD arbitration | `contexts/BUILD_ORCHESTRATOR.md` |

## Compact lookup first

Before opening deep analysis:

- `research/VERIFIED_APK_SNAPSHOT.md`
- `AUTO_FEATURE_READINESS.md`
- `database/AUTO_TOOL_API_CATALOG.md`
- `database/AUTO_TOOL_ACTION_CATALOG.md`
- `database/FACTS.jsonl`
- `research/AUTO_RUNTIME_PROOF_QUEUE.md`

## Exact network action question

If asking “packet/hàm nào thực sự gửi hành động”:

1. check `database/AUTO_TOOL_ACTION_CATALOG.md`;
2. if status is TARGETED-PROOF, inspect only `analysis/02_LUA_GAME_UI_NETWORK_API.md` plus related feature doc;
3. trace one manual action at `SendPacketToServer` / `SendPacket` / `TCPGame` boundary on LD9;
4. record exact packet/payload and success proof;
5. update FACTS/catalog/feature status.

Do not reverse all UI bundles first.

## PC comparison question

Use `analysis/08_PC_MOBILE_CROSSWALK.md`.

PC facts may guide exact symbol/packet search, but numeric packet IDs and payloads remain mobile-unverified until recovered from APK producer/runtime trace.

## Inventory question

Use:

- `contexts/BUILD_AUTO_SELL.md`
- `analysis/04_INVENTORY_ITEMS_SHOP.md`
- live semantic item fields / API catalog.

Do not count bag cells by image unless semantic bag APIs fail.

## Map/NPC question

Use `analysis/03_WORLD_ENTITY_MAP_PATH.md` and `database/AUTO_TOOL_API_CATALOG.md`.

Prefer `GetNPCPosition`, `GoTo`, `ClickNPC`, auto-path state. Do not invent pixel coordinates.

## Crash/diss during internal action

Use:

1. `contexts/BUILD_MAINTHREAD_BRIDGE.md`
2. `analysis/07_MAIN_THREAD_DISPATCHER.md`
3. `analysis/05_LD9_HOST_GUEST_ARCHITECTURE.md`
4. per-instance action gate logs.

First suspects: wrong thread, stale managed/native object, overlapping actions, map/UI generation change, invalid payload, unrooted managed delegate.

## Hard routing rule

**Route → lookup → targeted proof. Do not broad-reverse the whole APK because one exact action is still unknown.**
