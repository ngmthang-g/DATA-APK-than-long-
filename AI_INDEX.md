# AI Knowledge Index — Thần Long Mobile APK / LD9

Repository: `ngmthang-g/DATA-APK-than-long-`

Đây là knowledge base cho **tool EXE Windows điều khiển nhiều LDPlayer 9 chạy Thần Long Mobile**.

---

# 1. Start here — không đọc toàn repo

```text
AI_BOOTSTRAP.md
 -> AUTO_TOOL_SCOPE.md
 -> AI_ROUTER.md
 -> contexts/BUILD_*.md phù hợp
 -> database catalog
 -> deep analysis nếu còn gap
```

Compact references:

- `AUTO_FEATURE_READINESS.md`
- `research/VERIFIED_APK_SNAPSHOT.md`
- `research/AUTO_RUNTIME_PROOF_QUEUE.md`
- `database/AUTO_TOOL_API_CATALOG.md`
- `database/AUTO_TOOL_ACTION_CATALOG.md`
- `database/FACTS.jsonl`
- `analysis/08_PC_MOBILE_CROSSWALK.md`

---

# 2. Frozen APK snapshot

Current artifact:

```text
ThanLongMobile_2024.apk
SHA256 9de719391c29d50816a3762f758abe26896cab4d996706711b30fdc10d9933f0
size   77,318,940 bytes
```

Core files:

```text
lib/arm64-v8a/libil2cpp.so                       90,658,536
lib/arm64-v8a/libunity.so                        22,212,304
lib/arm64-v8a/libFGClientTool_Android.so            876,144
assets/bin/Data/Managed/Metadata/global-metadata.dat 14,475,116
assets/bin/Data/data.unity3d                     25,382,730
assets/Interface*.unity3d                        multiple bundles
```

Metadata sanity `FA B1 1B AF` + version `0x27` = **39**.

`ScriptingAssemblies.json` includes `Assembly-CSharp.dll`, `Assembly-CSharp-firstpass.dll`, Unity modules, Google.Protobuf, protobuf-net, Newtonsoft.Json and other runtime assemblies.

---

# 3. PC ↔ Mobile relationship

Evidence strongly supports a shared/closely-related FGStudio Unity codebase: same high-value namespaces/class/method vocabulary appears on mobile and PC (`LuaSystemAPI_Game`, `MainThread`, semantic Game APIs, TCP layer, item/map/NPC concepts).

Use PC repo as **semantic donor/index accelerator**. Do not reuse PC RVA/offsets.

Canonical crosswalk: `analysis/08_PC_MOBILE_CROSSWALK.md`.

---

# 4. Runtime state surface

High-value mobile metadata names include:

```text
RoleData
LuaLeaderData
CurrentHP / MaxHP
MapID / PosX / PosY / Position
IsDeath
IsMapReady
GetCurrentMoveDestination
GetLocalMapObjects
GetNearbyObjects
```

Target/combat surface:

```text
GetNearByEnemies
SelectTarget
ChaseTarget
UseSkill
RequestUsingSkill
RequestUsingSkillWithTarget
RequestUsingSkillWithPos
```

These provide a strong route to semantic scanner/action logic instead of OCR/image matching.

---

# 5. Map / movement / NPC

Verified names:

```text
MoveTo
MoveToEx
GoTo
GetNPCPosition
ClickNPC
AutoPathManager
StartAutoPath
StopAutoPath
IsAutoPathing
SendAutoPathRequestChangeMap
```

Target design:

```text
current snapshot
 -> choose semantic destination
 -> GoTo/AutoPath
 -> verify MapID + IsMapReady + fresh Position within tolerance
```

Do not hardcode screen coordinates when semantic position/path APIs are available.

---

# 6. Inventory / Auto Sell

Verified metadata names:

```text
GetFreeBagSpace
GetItems
GetItemsAtSite
GetItemData
GetItemAtSite
GetTotalItems
CountItem
IsItemSellable
GetItemBasePrice
IsItemSellToShopWithBoundMoney
```

Item-related field names present:

```text
ID
ItemID
Site
Position
Bound
Quantity
Creator
DurationLeft
Durability
Properties
Attributes
SellPrice
BasePrice
```

`ItemSite` and names such as `Bag`, `Body`, `Storage`, `Stall`, `Trade`, `Pet` are present.

Important unresolved boundary: exact mobile sell request producer/packet payload/current-shop state has not yet been runtime-proven. PC's `200036` is `PC-DONOR`, not mobile VERIFIED.

Canonical: `features/AUTO_SELL.md`, `analysis/04_INVENTORY_ITEMS_SHOP.md`.

---

# 7. Death / Revive

Verified names:

```text
IsDeath
ProcessObjectDeath
ProcessObjectRevive
CMD_REVIVE
```

This proves death/revive concepts are present in client metadata/network processing.

It does **not** yet prove exact outbound mobile revive packet number/payload. PC's `CMD_REVIVE_DATA=200063` remains donor evidence until mobile trace/producer recovery.

Canonical: `features/AUTO_REVIVE.md`.

---

# 8. Lua / network bridge

Verified names:

```text
LuaSystemManager
LuaSystemAPI_Game
LuaSystemAPI_Network
SendPacketToServer
SendPacket
TCPGame
TCPOutPacket
PacketCmdID
```

This gives two narrow routes for exact action research:

1. static producer call-chain recovery;
2. runtime trace of one manual action on LD9, logging command/payload at the semantic network boundary.

Prefer these over reverse-engineering all UI bundles.

---

# 9. MainThread

`FGStudio.Engine.Utilities.MainThread` exists in metadata. Previous parse found members consistent with PC: `Instance`, `Execute(Action)`, `Update`, `DoExecuteWorks`, queue-backed dispatch.

Status: class/pattern **RECONFIRMED**, external live callback from bridge still `TARGETED-PROOF`.

Canonical: `analysis/07_MAIN_THREAD_DISPATCHER.md`, `contexts/BUILD_MAINTHREAD_BRIDGE.md`.

---

# 10. LD9 host/guest architecture

Windows EXE cannot directly invoke ARM64 methods as though the Android game were a Windows DLL.

Canonical split:

```text
ThanLongAuto.exe
 -> LD discovery + ADB/instance mapping
 -> per-instance command/state channel
 -> ARM64 guest bridge attached to game process
 -> IL2CPP resolver + semantic state/actions
```

Each LD9 is a separate BotSession.

Canonical: `analysis/05_LD9_HOST_GUEST_ARCHITECTURE.md`.

---

# 11. Emulator/security surface

Native exports in `libFGClientTool_Android.so` include:

```text
FG_EmuDetect
FG_GetEmuScore
FG_GetEmuReason
FG_IsAdbEnabled
FG_IsAdbReallyEnabled
FG_GetEnabledAccessibilityServices
FG_CanDrawOverlays
FG_GetSecurityReport
FG_Input_OnTap
FG_Input_GetMetrics
FG_Decrypt
FG_Encrypt
```

Metadata also contains `SecurityReport` and `LoginData` names.

Interpretation: emulator/ADB/accessibility/overlay/input/security state is observable by client code. This is a stability/risk consideration, not proof of a ban/block policy.

---

# 12. Feature readiness

See `AUTO_FEATURE_READINESS.md`.

Current summary:

```text
Read-only scanner primitives      strong static basis
Map/path/NPC primitives           strong static basis
Combat primitives                strong static basis
Bag/item filtering               strong static basis
MainThread dispatcher             static basis; live bridge proof needed
Revive exact outbound request     targeted proof needed
Sell exact outbound request       targeted proof needed
Multi-LD orchestration            architecture/design, runtime implementation needed
```

---

# 13. Current real gaps

Do not broad reverse the APK again. Highest-value gaps are narrow:

1. prove per-LD bridge can resolve/read semantic state repeatedly without destabilizing game;
2. prove harmless MainThread callback/action path;
3. capture/recover exact outbound revive producer and payload;
4. capture/recover exact shop-open + sell request producer and payload;
5. verify return-to-train sequence across map transitions;
6. test multi-instance isolation and no shared stale state.
