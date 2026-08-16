# Auto Tool Scope — Mobile APK / LDPlayer 9

## Primary product

A single Windows `ThanLongAuto.exe` manages multiple LDPlayer 9 instances. Each emulator runs one game client and is treated as an independent automation session.

## In scope

### Core scanner

- LD instance / ADB serial / game PID mapping;
- player identity;
- HP/MaxHP/death;
- map ID / position / map-ready;
- selected/nearby targets;
- bag free space and live item list;
- current action/proof state.

### Auto Train

- prefer shipped semantic Train engine where practical;
- target scan/select/chase/skill/loot;
- train origin/radius/return-center;
- yield for higher-priority Revive/Heal/Sell states;
- resume after recovery.

### Death recovery / return map

- semantic death detection;
- one exact normal revive request;
- alive/map-ready proof;
- return to saved train spot;
- resume Train.

### Auto HP/MP / survival

- identify HP/MP medicine semantically;
- use live bag instance ID through `CMD_ITEM_ACTION`;
- one action -> state/item proof -> rescan;
- reserve configured recovery items from sell/drop filters.

### Inventory / Auto Sell / item policy

- `GetFreeBagSpace` and live Bag scan rather than cell OCR;
- keep/sell/drop/destroy/move policy using semantic item fields;
- preserve `IsItemSellable` / `IsItemThrowable` and quest-item guards;
- destructive `Destroy` only when explicitly configured;
- route to current vendor, require normal NPCShop;
- sell one current item instance at a time and prove removal;
- return to train spot.

### NPC / Trị liệu

- route by NPC ID with `GetNPCPosition -> GoTo -> ClickNPC`;
- inspect current `GameDialog.Selections`;
- match current visible treatment/service text;
- use actual live selection ID;
- prove HP/money/dialog result.

### Auto Chat / ping tọa độ

- semantic channel send through `CMD_CLIENT_CHAT`;
- Base64 message content;
- `@GOTO_<MapID>_<GridX>_<GridY>` location ping;
- preserve client practical length limit and implement conservative rate limiting;
- no Windows focus/keyboard dependency when semantic send works.

### Multi-LD orchestration

- independent settings/profile per instance;
- one mutable action gate per game PID;
- no live pointer/state sharing;
- per-instance logs and error isolation;
- global UI may start/pause selected sessions without merging their state machines.

## Out of scope by default

Do not spend research budget on:

- cosmetics/rendering/voice chat;
- full quest encyclopedia;
- every NPC/item/monster unless required by a configured feature;
- bypassing Captcha/security/anti-cheat;
- generic screen macro engines when semantic game state/action exists.

## Architectural non-goals

Do not build production control around:

```text
OCR HP
pixel template matching for normal state
hardcoded LD screen coordinates
fixed sleep as action success proof
PC RVA copied into ARM64 mobile
one global mutable queue shared by all emulators
unbounded packet/action spam
```

Fallback input may exist only for a genuinely unexposed UI edge and must be isolated from semantic state truth.

## Success criteria

A feature is production-ready only when it has:

```text
STATE source
 + GUARD
 + ONE semantic action
 + RESULT proof
 + timeout/failure path
 + fresh rescan
 + multi-instance isolation
```
