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
- `ITEM_SITES_MOBILE.csv` — Bag/Body/Mount/Storage/Fashion/Gem/etc. site IDs.
- `EQUIP_POSITIONS_MOBILE.csv` — equipment positions; mobile Weapon position is exactly 0.
- `AUTO_TOOL_ACTION_CATALOG.md` — exact Sell packet and mutation contract.
- canonical bag scanner row contract: `../analysis/31_BAG_SCANNER_DATA_CONTRACT.md`.

Important mobile weapon rule:

```text
Game.GetItemType(ItemID) == "Equip"
AND Game.GetEquipType(ItemID) == 0
```

Recovered Lua proves `Game.GetEquipType()` is operationally compared against `C_EquipPosition`, not the misleadingly named `C_EquipType` family enum. Do not use `<10 => weapon`.

## NPC / transactions

- canonical stable-vs-live NPC identity strategy: `../analysis/30_NPC_DISCOVERY_IDENTITY_RUNTIME.md`.
- dynamic dialog/shop/revival copy contract: `../analysis/28_DYNAMIC_TRANSACTION_SNAPSHOT_CONTRACT.md`.

Do not persist live NPC RoleID, current GameDialog selectionID, current ShopID/NpcShopID or Lua UI pointers as profile identity.

## Coordinates

Canonical grid/world contract: `../analysis/29_COORDINATE_DOMAINS_GRID_WORLD.md`.

Visible map coordinates and movement world coordinates are different domains. Use the game's `WorldToGridPosition` / `GridToWorldPosition` rather than host-side guessed multiplication.

## Target / nearby-player scanner

Canonical schema: `../analysis/32_RUNTIME_TARGET_SCANNER.md`.

Mobile `LuaMapSpriteData` exposes/inherits semantic RoleID/ResID/Name/Type/Position/IsDeath/HP/MaxHP/Level/Faction/Guild/Team data. The stock nearby-player UI independently uses HP/MaxHP directly.

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
- central inbound observer: `../analysis/26_LUA_PACKET_EVENT_OBSERVER.md`.
- dynamic transaction generations: `../analysis/28_DYNAMIC_TRANSACTION_SNAPSHOT_CONTRACT.md`.
- state/action/proof orchestration: `../analysis/27_AUTO_STATE_ACTION_PROOF_MATRIX.md`.

## Evidence rule

Do not treat `PC-DONOR` as mobile production truth. Do not confuse the engine/core command space with Lua gameplay packet IDs. If an exact mobile fact exists here, do not broad-reverse it again unless the APK version changes or evidence conflicts.
