# AI Bootstrap — đọc file này đầu tiên

Repo này được thiết kế để AI không phải reverse lại toàn bộ APK sau mỗi phiên làm việc.

## Primary scope

Mục tiêu là xây **Windows EXE quản lý nhiều LDPlayer 9**, mỗi LD9 chạy một APK game và một BotSession độc lập.

Ưu tiên kiến thức phục vụ:

- đọc tên nhân vật/RoleID/HP/MaxHP/map/tọa độ/death;
- đọc target/quái xung quanh;
- di chuyển/path/đổi map/quay lại bãi;
- bật/tắt/điều phối train;
- quét tay nải và phân loại item;
- mở NPC/shop và bán an toàn;
- hồi sinh/đầu thai;
- điều phối nhiều LD9 mà action không tranh chấp;
- proof hành động thành công để tránh spam/diss/crash.

## Read flow bắt buộc

```text
AI_BOOTSTRAP.md
 -> AUTO_TOOL_SCOPE.md
 -> AI_ROUTER.md
 -> đúng một contexts/BUILD_*.md
 -> compact database/catalog
 -> deep analysis khi thật sự còn gap
```

Mục tiêu bình thường: đọc khoảng 5–10 tài liệu liên quan, không preload toàn repo.

## Evidence levels

- `VERIFIED` — có bằng chứng trực tiếp từ APK/metadata/native export/config hoặc runtime capture cụ thể.
- `RECONFIRMED` — fact từ parse trước đã được kiểm tra lại bằng artifact hiện tại.
- `PROBABLE` — suy luận mạnh nhưng thiếu exact association/payload/runtime proof.
- `PC-DONOR` — fact đã verified trên PC, hữu ích để định hướng mobile nhưng **không tự động đúng trên APK**.
- `TARGETED-PROOF` — đã biết chính xác cần test gì trên LD9 để nâng cấp fact.
- `HYPOTHESIS` — hướng nghiên cứu, chưa dùng làm logic production.

Không được tự nâng `PC-DONOR/PROBABLE` thành `VERIFIED`.

## Rule chống reverse lan man

Nếu exact fact đã tồn tại trong `research/VERIFIED_APK_SNAPSHOT.md`, `database/FACTS.jsonl` hoặc catalog thì dùng fact đó. Chỉ phân tích binary/asset sâu hơn khi task hiện tại thật sự thiếu thông tin.

## Rule chống bê offset PC sang APK

PC và mobile có nhiều dấu hiệu chung codebase, nhưng:

```text
PC = Windows x64 / GameAssembly.dll
Mobile = Android ARM64 / libil2cpp.so
```

Tên namespace/class/method/protocol semantic có thể đối chiếu. **Absolute offset/RVA/pointer layout không được copy 1:1.** Resolver mobile phải dựa IL2CPP metadata/runtime hoặc signature riêng cho snapshot mobile.

## Canonical per-LD architecture

```text
Resolver
 -> Read-only Scanner
 -> Immutable Snapshot
 -> Observer
 -> State Machine
 -> Safety Guard
 -> Action Gate (max 1 mutable action)
 -> MainThread / semantic game API
 -> Result proof
 -> Fresh snapshot
```

Read-only observers có thể chạy song song. Mutable action trong cùng một game instance không được cạnh tranh.

## Multi-LD hard rules

Mỗi instance giữ riêng:

```text
LD index / ADB serial
Windows emulator PID/window
Android game PID
resolver cache
snapshot version/world generation
character identity
train profile
sell profile
feature state
last action/proof
error/retry counters
```

Không share live pointer, action queue, selected target, shop state hay cached item instance ID giữa các LD9.

## MainThread rule

Metadata có `FGStudio.Engine.Utilities.MainThread`; nghiên cứu trước đã thấy mô hình `Execute(Action)`/queue/Unity Update giống client PC. Trước production write-action cần proof live harmless callback trên LD9. Mutable Unity/Lua call không được gọi tùy tiện từ host worker thread.

## Current high-value VERIFIED names

Static metadata hiện tại chứa các tên:

```text
LuaSystemAPI_Game
LuaSystemManager
LuaSystemAPI_Network
MainThread
RoleData / LuaLeaderData
GetFreeBagSpace
GetItemsAtSite / GetItems / GetItemData / GetItemAtSite / GetTotalItems / CountItem
IsItemSellable / GetItemBasePrice / IsItemSellToShopWithBoundMoney
GetNPCPosition / ClickNPC
MoveTo / MoveToEx / GoTo
AutoPathManager / StartAutoPath / StopAutoPath / IsAutoPathing
SendAutoPathRequestChangeMap
GetNearByEnemies / SelectTarget / ChaseTarget
UseSkill / RequestUsingSkill / RequestUsingSkillWithTarget / RequestUsingSkillWithPos
IsMapReady / GetCurrentMoveDestination / GetLocalMapObjects / GetNearbyObjects
IsDeath / ProcessObjectDeath / ProcessObjectRevive / CMD_REVIVE
SendPacketToServer / SendPacket / TCPGame / TCPOutPacket / PacketCmdID
```

Tên tồn tại không tự chứng minh payload hoặc runtime-call safety; đọc feature/context tương ứng.

## Security/emulator caution

`libFGClientTool_Android.so` export các surface như `FG_EmuDetect`, `FG_GetEmuScore`, `FG_IsAdbEnabled`, `FG_IsAdbReallyEnabled`, `FG_GetEnabledAccessibilityServices`, `FG_CanDrawOverlays`, `FG_GetSecurityReport`, input metrics/tap hooks.

Điều này không chứng minh LD9 bị block, nhưng có nghĩa thiết kế không nên giả định emulator/ADB/accessibility/overlay là vô hình.

## Mandatory next step

Đọc `AUTO_TOOL_SCOPE.md`, sau đó `AI_ROUTER.md` và chọn đúng context pack.
