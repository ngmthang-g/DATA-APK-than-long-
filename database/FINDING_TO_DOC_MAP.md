# Finding → Canonical Document Map

Use this file to jump from one concrete implementation question to the smallest canonical evidence set.

| Finding/question | Read |
|---|---|
| APK fingerprint / metadata version | `research/VERIFIED_APK_SNAPSHOT.md`, `CLIENT_MANIFEST.md` |
| IL2CPP resolver/export strategy | `analysis/01_IL2CPP_RUNTIME_METADATA.md` |
| recovered original C# source layout | `analysis/10_SOURCE_PATH_MANIFEST.md` |
| decrypted Lua/UI corpus | `analysis/11_INTERFACE_BUNDLE_DECRYPTION.md` |
| packet 80 vs 200063 / command layers | `analysis/12_PACKET_LAYERS_AND_LUA_PROTOCOL.md`, `database/PACKET_IDS_LUA_MOBILE.csv` |
| HP/map/position/death/bag snapshot | `analysis/18_RUNTIME_ROLE_BAG_SNAPSHOT.md`, `contexts/BUILD_RUNTIME_SCANNER.md` |
| transient GameDialog/NPCShop/Revival/bag events | `analysis/26_LUA_PACKET_EVENT_OBSERVER.md` |
| EXE controls LD9 APK / process binding | `analysis/25_ANDROID_MANIFEST_PROCESS_BINDING.md`, `analysis/05_LD9_HOST_GUEST_ARCHITECTURE.md` |
| guest Lua/UI semantic bridge | `analysis/24_LUA_UI_ACTION_BRIDGE_BLUEPRINT.md`, `contexts/BUILD_MAINTHREAD_BRIDGE.md` |
| MainThread / crash from wrong action thread | `analysis/07_MAIN_THREAD_DISPATCHER.md` |
| map/path/cross-map/return/NPC route | `analysis/03_WORLD_ENTITY_MAP_PATH.md` |
| built-in Train exact flow | `analysis/13_BUILTIN_AUTO_TRAIN_ENGINE.md`, `features/AUTO_TRAIN.md` |
| loot/pickup/bag-full Sell trigger | `analysis/22_LOOT_BAG_FULL_AND_SELL_TRIGGER.md` |
| HP/MP medicine / Use item | `analysis/20_ITEM_ACTIONS_AND_AUTO_RECOVERY.md`, `features/AUTO_RECOVERY.md` |
| Abandon/Move/Destroy item | `analysis/20_ITEM_ACTIONS_AND_AUTO_RECOVERY.md`, `database/ITEM_ACTIONS_MOBILE.csv` |
| stock auto bugs/stale toggles | `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md` |
| revive packet + return map | `analysis/14_REVIVE_RETURN_MAP_ENGINE.md`, `features/AUTO_REVIVE.md` |
| shop/sell exact request | `analysis/15_INVENTORY_NPCSHOP_AUTO_SELL.md`, `features/AUTO_SELL.md` |
| classify KEEP vs SELL safely | `database/AUTO_SELL_CLASSIFICATION.md` |
| NPC Trị liệu / dynamic GameDialog | `analysis/16_GAMEDIALOG_NPC_TREATMENT.md`, `features/AUTO_HEAL_NPC.md` |
| Auto Chat / actual channel ID / @GOTO | `analysis/17_CHAT_CHANNEL_AND_GOTO_AUTOMATION.md`, `features/AUTO_CHAT.md` |
| built-in mobile AutoSettings 3.5 | `database/BUILTIN_AUTO_SETTINGS_MOBILE.md` |
| external per-LD profile/settings | `database/EXE_PER_LD_PROFILE_SCHEMA.md` |
| Lua function lookup without opening whole script | `database/AUTO_LUA_FUNCTION_CATALOG.csv` |
| exact action quick lookup | `database/AUTO_ACTION_EXACT_FLOWS.md`, `database/AUTO_TOOL_ACTION_CATALOG.md` |
| state→guard→action→proof orchestration | `analysis/27_AUTO_STATE_ACTION_PROOF_MATRIX.md` |
| multiple LD state arbitration | `analysis/19_LD9_ACTION_ORCHESTRATION.md`, `features/AUTO_ORCHESTRATOR.md` |
| packaged APK lacks full PC-style Config tables | `analysis/21_PACKAGED_RESOURCE_BOUNDARY.md` |
| compare with PC repo | `analysis/08_PC_MOBILE_CROSSWALK.md` |
| emulator/ADB/security observation | `analysis/06_SECURITY_EMULATOR_SURFACE.md` |
| what still needs live test | `research/AUTO_RUNTIME_PROOF_QUEUE.md` |
