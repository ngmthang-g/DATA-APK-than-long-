# AI Router — mobile automation tool

Read `AI_BOOTSTRAP.md` and `AUTO_TOOL_SCOPE.md` first.

| Task | Primary context pack |
|---|---|
| EXE/LD9 architecture, session isolation, orchestration | `contexts/BUILD_TOOL_CORE.md` |
| HP/map/position/death/bag/target scanner | `contexts/BUILD_RUNTIME_SCANNER.md` |
| Unity/IL2CPP action bridge / MainThread | `contexts/BUILD_MAINTHREAD_BRIDGE.md` |
| Auto Train / target / chase / skill / return center | `contexts/BUILD_AUTO_TRAIN.md` |
| HP/MP item recovery / survival medicine | `contexts/BUILD_AUTO_RECOVERY.md` |
| Death / Đầu thai / return map | `contexts/BUILD_AUTO_REVIVE.md` |
| Bag threshold / loot / vendor / shop / sell | `contexts/BUILD_AUTO_SELL.md` |
| NPC Trị liệu / GameDialog | `contexts/BUILD_AUTO_HEAL.md` |
| Auto chat / channel / @GOTO ping | `contexts/BUILD_AUTO_CHAT.md` |
| Cross-feature state machine / many LD9 | `contexts/BUILD_ORCHESTRATOR.md` |

## Exact-action questions

Start with:

- `database/AUTO_ACTION_EXACT_FLOWS.md`
- `database/AUTO_TOOL_ACTION_CATALOG.md`
- `database/PACKET_IDS_LUA_MOBILE.csv`
- `database/ITEM_ACTIONS_MOBILE.csv` for item use/drop/move/destroy.

Do not reopen binary reverse for facts already solved by recovered Lua.

## Auto Train / loot / bag full

Read `analysis/13_BUILTIN_AUTO_TRAIN_ENGINE.md` and `analysis/22_LOOT_BAG_FULL_AND_SELL_TRIGGER.md`.

Mobile `StartAutoFight(C_AutoModel.Train)` is independently source-verified. Stock pickup already uses `Game.GetFreeBagSpace()` and disables pickup when space is insufficient. Therefore prefer a semantic transition such as `FreeBagSpace <= configured threshold -> Sell`, not a periodic bag-window scan.

## Auto Recovery

Read `analysis/20_ITEM_ACTIONS_AND_AUTO_RECOVERY.md` and `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md`.

Exact medicine action is `CMD_ITEM_ACTION=100005` with `Use=3` and payload `3:<live DBItemData.ID>`. Preserve configured medicine from Sell/Drop/Destroy policy. Do not copy the stock successful-use timestamp bug.

## Revive

Read `analysis/14_REVIVE_RETURN_MAP_ENGINE.md`. Exact normal request is mobile-verified `200063:"1"`. Remaining work is runtime bridge/server completion proof, not payload discovery.

## Auto Sell

Read `analysis/15_INVENTORY_NPCSHOP_AUTO_SELL.md` plus `analysis/22_LOOT_BAG_FULL_AND_SELL_TRIGGER.md`. Exact mobile request is `200036` with `itemInstanceID:NpcShopID:ShopID`. Do not rediscover the packet. Runtime work should verify vendor/current shop and removal lifecycle.

## Item drop/destroy/move

Read `analysis/20_ITEM_ACTIONS_AND_AUTO_RECOVERY.md` and `database/ITEM_ACTIONS_MOBILE.csv`.

- Abandon = `100005` / `4:itemInstanceID`, preserve `IsItemThrowable` guard.
- Move = `5:itemInstanceID:destinationSite`.
- Destroy = `9:id1;id2;...` and is destructive; keep disabled unless user explicitly enables a narrow policy.

Do not trust the stock AutoDrop toggle as a working engine; source shows stale/defective settings paths.

## Treatment

Read `analysis/16_GAMEDIALOG_NPC_TREATMENT.md`. Match runtime selection text and use actual `selectionID:-1`; do not hardcode screen coordinates or a guessed selection ID. The word `Trị liệu` found in packaged Lua also names the Auto HP tab and is not proof of one NPC server selection.

## Chat / ping

Read `analysis/17_CHAT_CHANNEL_AND_GOTO_AUTOMATION.md`. Use `CMD_CLIENT_CHAT=100008` and `@GOTO_MapID_GridX_GridY`; do not automate Windows keyboard focus when semantic send exists. No stock periodic Auto-Chat sender was found; scheduling/event triggers belong to the EXE.

## PC comparison

Use PC data as a Config/static identity donor where the APK snapshot does not embed current config tables. Never reuse Windows RVA/offsets. Promote a PC donor fact to mobile VERIFIED only with mobile static/runtime evidence.

## Packaged resource question

Read `analysis/21_PACKAGED_RESOURCE_BOUNDARY.md`. Do not repeatedly search the solved Shared/LoadingResources/Logo graphics bundles for full NPC/item/map Config tables.

## Crash/diss

First inspect MainThread/action ownership/session generation and `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md`. Do not solve instability by adding more arbitrary sleeps or concurrent worker actions.

**Hard rule: route -> compact exact fact -> targeted runtime proof.**
