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

- identify nearby enemies;
- select/chase/use skills or invoke built-in semantic train flow if recovered;
- keep train origin/radius;
- stop/yield for higher-priority recovery/sell states;
- resume after recovery.

### Death recovery

- detect death from semantic state;
- wait for valid revive availability/state if required;
- issue exactly one revive request;
- prove alive/map-ready;
- route back to saved train spot;
- resume train.

### Auto Sell

- use `GetFreeBagSpace` rather than counting UI cells;
- scan live Bag items;
- apply keep/sell policy using semantic item data;
- route to configured/current vendor NPC semantically;
- establish current shop state;
- sell one live item instance at a time;
- prove removal/update, rescan;
- return to train spot.

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
