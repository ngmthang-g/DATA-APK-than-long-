# Auto Feature Readiness — Mobile APK

Legend:

- `STATIC-STRONG` — strong static semantic basis exists.
- `TARGETED-PROOF` — exact remaining runtime/static proof is narrow and known.
- `DESIGN` — architecture defined, implementation/runtime proof pending.
- `PC-DONOR` — PC has a solved equivalent; useful to guide mobile work only.

| Feature | Status | What is already known | Remaining proof |
|---|---|---|---|
| IL2CPP resolver | STATIC-STRONG | ARM64 `libil2cpp.so`, metadata v39, standard `il2cpp_*` exports such as domain/class/method lookup and runtime invoke | live resolver test inside LD9 guest process |
| Local state scanner | STATIC-STRONG | RoleData/LuaLeaderData, HP/map/pos/death names present | exact object acquisition chain + repeated stability |
| Map readiness/position | STATIC-STRONG | `IsMapReady`, map/position fields, move destination/local/nearby objects | runtime snapshot contract |
| NPC navigation | STATIC-STRONG | `GetNPCPosition`, `GoTo`, `ClickNPC` | actual configured vendor/NPC runtime flow |
| Auto path | STATIC-STRONG | `AutoPathManager`, `StartAutoPath`, `StopAutoPath`, `IsAutoPathing`, change-map request name | live behavior across portal/map change |
| Combat primitives | STATIC-STRONG | nearby enemy, select, chase, skill request names | recover preferred high-level train donor / live harmless test |
| Bag scanner | STATIC-STRONG | free space, item list/data/site APIs and item identity fields | exact object/schema mapping in runtime snapshot |
| Item sellability | STATIC-STRONG | `IsItemSellable`, base/sell price related names | keep/sell policy + exact sell request |
| MainThread | TARGETED-PROOF | class and dispatch pattern reconfirmed; PC equivalent fully solved | one harmless live guest callback/action proof |
| Revive | TARGETED-PROOF | death/revive handlers and `CMD_REVIVE` name present | exact outbound packet ID/payload + acceptance/completion proof |
| Auto Sell request | TARGETED-PROOF | bag/item/NPC/network primitives present | exact shop-open state + outbound sell packet/payload + removal proof |
| Return-to-train | DESIGN | GoTo/path/map-ready primitives present | runtime transition policy and tolerances |
| Multi-LD EXE | DESIGN | host/guest architecture defined | LD discovery, channel, isolation, soak test |
| Emulator/security compatibility | TARGETED-PROOF | native detection/reporting surfaces present | observe real behavior without bypassing security controls |

## PC donor facts that must not be silently copied

PC KB has exact examples including revive packet `200063` and sell packet `200036`. These are high-value search hints. They are **not mobile VERIFIED** in this repo until a mobile producer/trace confirms them.

## Production readiness rule

No mutable feature becomes `READY` solely because a method/packet name appears in metadata. It needs live call safety plus server/client state proof.
