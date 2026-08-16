# Mobile Auto Database — compact lookup

This directory is for implementation lookup, not long narrative reverse-engineering.

## Start with these

- `AUTO_TOOL_API_CATALOG.md` — semantic state/query/API surface.
- `AUTO_TOOL_ACTION_CATALOG.md` — mutable actions + guards/proof expectations.
- `AUTO_ACTION_EXACT_FLOWS.md` — shortest exact packet/API flow cheat sheet.
- `FACTS.jsonl` — atomic machine-readable facts.
- `FINDING_TO_DOC_MAP.md` — finding/question → canonical deep document.

## Packet / event lookup

- `PACKET_IDS_LUA_MOBILE.csv` — exact Lua gameplay + core command IDs relevant to auto.
- `RUNTIME_EVENTS_MOBILE.csv` — inbound packet/event IDs useful for proof/invalidation.
- `ITEM_ACTIONS_MOBILE.csv` — Use/Abandon/Move/Merge/Split/Destroy exact item actions.

## Inventory / Sell

- `AUTO_SELL_CLASSIFICATION.md` — conservative KEEP/SELL/DROP/DESTROY classification contract.
- `AUTO_TOOL_ACTION_CATALOG.md` — exact Sell packet and mutation contract.

## Chat

- `CHAT_CHANNELS_MOBILE.csv` — actual server/client channel IDs. Do not confuse with ChatBox dropdown index.

## Settings / profiles

- `BUILTIN_AUTO_SETTINGS_MOBILE.md` — shipped mobile AutoSettings schema, version 3.5.
- `EXE_PER_LD_PROFILE_SCHEMA.md` — recommended tool-owned independent profile per LD9.

Do not place external vendor/shop/dialog transaction IDs or live item instance IDs into persistent config.

## Source-evidence navigation

- `RECOVERED_LUA_EVIDENCE.csv` — hashes of key recovered Lua/TextAsset evidence.
- `AUTO_LUA_FUNCTION_CATALOG.csv` — key Lua function → source asset/domain lookup without opening whole scripts.
- `APK_SYMBOLS.csv` — frozen APK symbol-name evidence.
- `PC_MOBILE_SYMBOL_CROSSWALK.csv` — PC donor ↔ mobile evidence status.

## Runtime state

- `RUNTIME_SNAPSHOT_SCHEMA.md` — original compact snapshot draft.
- canonical expanded state/generation contract: `../analysis/18_RUNTIME_ROLE_BAG_SNAPSHOT.md`.
- state/action/proof orchestration: `../analysis/27_AUTO_STATE_ACTION_PROOF_MATRIX.md`.

## Evidence rule

Do not treat `PC-DONOR` as mobile production truth. Do not confuse the engine/core command space with Lua gameplay packet IDs. If an exact mobile fact exists here, do not broad-reverse it again unless the APK version changes or evidence conflicts.
