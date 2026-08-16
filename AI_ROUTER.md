# AI Router — mobile automation tool

Read `AI_BOOTSTRAP.md` and `AUTO_TOOL_SCOPE.md` first.

| Task | Primary context pack |
|---|---|
| EXE/LD9 architecture, session isolation, orchestration | `contexts/BUILD_TOOL_CORE.md` |
| HP/map/position/death/bag/target scanner | `contexts/BUILD_RUNTIME_SCANNER.md` |
| Unity/IL2CPP action bridge / MainThread | `contexts/BUILD_MAINTHREAD_BRIDGE.md` |
| Auto Train / target / chase / skill / return center | `contexts/BUILD_AUTO_TRAIN.md` |
| Death / Đầu thai / return map | `contexts/BUILD_AUTO_REVIVE.md` |
| Bag full / vendor / shop / sell | `contexts/BUILD_AUTO_SELL.md` |
| NPC Trị liệu / GameDialog | `contexts/BUILD_AUTO_HEAL.md` |
| Auto chat / channel / @GOTO ping | `contexts/BUILD_AUTO_CHAT.md` |
| Cross-feature state machine / many LD9 | `contexts/BUILD_ORCHESTRATOR.md` |

## Exact-action questions

Start with:

- `database/AUTO_ACTION_EXACT_FLOWS.md`
- `database/AUTO_TOOL_ACTION_CATALOG.md`
- `database/PACKET_IDS_LUA_MOBILE.csv`

Do not reopen binary reverse for facts already solved by recovered Lua.

## Auto Train

Read `analysis/13_BUILTIN_AUTO_TRAIN_ENGINE.md`. Mobile `StartAutoFight(C_AutoModel.Train)` is now independently source-verified; it is no longer PC-donor-only.

## Revive

Read `analysis/14_REVIVE_RETURN_MAP_ENGINE.md`. Exact normal request is now mobile-verified `200063:"1"`. Remaining work is runtime bridge/server completion proof, not payload discovery.

## Auto Sell

Read `analysis/15_INVENTORY_NPCSHOP_AUTO_SELL.md`. Exact mobile request is now `200036` with `itemInstanceID:NpcShopID:ShopID`. Do not rediscover the packet. Runtime work should verify vendor/current shop and removal lifecycle.

## Treatment

Read `analysis/16_GAMEDIALOG_NPC_TREATMENT.md`. Match runtime selection text and use actual `selectionID:-1`; do not hardcode screen coordinates or a guessed selection ID.

## Chat / ping

Read `analysis/17_CHAT_CHANNEL_AND_GOTO_AUTOMATION.md`. Use `CMD_CLIENT_CHAT=100008` and `@GOTO_MapID_GridX_GridY`; do not automate Windows keyboard focus unless semantic action is unavailable.

## PC comparison

Use PC data as a Config/static identity donor where the APK snapshot does not embed current config tables. Never reuse Windows RVA/offsets. Promote a PC donor fact to mobile VERIFIED only with mobile static/runtime evidence.

## Crash/diss

First inspect MainThread/action ownership/session generation. Do not solve instability by adding more arbitrary sleeps or concurrent worker actions.

**Hard rule: route -> compact exact fact -> targeted runtime proof.**
