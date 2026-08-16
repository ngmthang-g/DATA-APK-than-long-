# AI Bootstrap — đọc file này đầu tiên

Repo này được thiết kế để AI không phải reverse lại toàn bộ APK sau mỗi phiên làm việc.

## Primary scope

Mục tiêu là xây **Windows EXE quản lý nhiều LDPlayer 9**, mỗi LD9 chạy một APK game và một BotSession độc lập.

Ưu tiên kiến thức phục vụ:

- đọc tên nhân vật/RoleID/HP/MaxHP/map/tọa độ/death;
- đọc target/quái xung quanh;
- di chuyển/path/đổi map/quay lại bãi;
- bật/tắt/điều phối Train;
- Auto HP/MP bằng item semantic;
- quét tay nải, phân loại, bán/vứt/xóa/chuyển item an toàn;
- mở NPC/shop/GameDialog, bán và trị liệu;
- Auto Chat và ping tọa độ;
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

- `VERIFIED` / `VERIFIED_STATIC` — bằng chứng trực tiếp từ APK/metadata/native/recovered Lua/UI.
- `RUNTIME-PROOF` — exact static action đã biết nhưng cần chứng minh an toàn/acceptance trên LD9.
- `PC-DONOR` — fact PC hữu ích để tìm mobile nhưng không tự động là mobile fact.
- `HYPOTHESIS` — hướng nghiên cứu chưa dùng làm production truth.

Không được tự nâng `PC-DONOR/HYPOTHESIS` thành VERIFIED.

## Rule chống reverse lan man

Nếu exact fact đã tồn tại trong `database/FACTS.jsonl`, action/API catalog hoặc canonical analysis thì dùng fact đó. Chỉ phân tích binary/asset sâu hơn khi task hiện tại thật sự thiếu thông tin.

## Rule chống bê offset PC sang APK

```text
PC = Windows x64 / GameAssembly.dll
Mobile = Android ARM64 / libil2cpp.so
```

Tên namespace/class/method/protocol semantic có thể đối chiếu. **Absolute offset/RVA/pointer layout không được copy 1:1.**

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

Không share live pointer, action queue, selected target, current shop/dialog hoặc cached item instance ID giữa các LD9.

## High-value exact mobile facts

Recovered Interface Lua + IL2CPP metadata independently verify:

```text
C_AutoModel.Train = 1
AutoFight_Main:StartAutoFight(C_AutoModel.Train)

CMD_REVIVE_DATA = 200063
normal revive payload = "1"

C_ItemSite.Bag = 10
CMD_NPC_SHOP_SELL_REQUEST = 200036
sell payload = DBItemData.ID:NpcShopID:ShopID

CMD_SHOW_GAMEDIALOG = 100007
payload = selectionID:SelectedItemID

CMD_CLIENT_CHAT = 100008
Content = Base64(text)
location token = @GOTO_<MapID>_<GridX>_<GridY>

CMD_ITEM_ACTION = 100005
Use=3
Abandon=4
Move=5
Destroy=9
```

Useful semantic APIs include:

```text
GetFreeBagSpace / GetItemsAtSite / IsItemSellable / IsItemThrowable
GetNPCPosition / ClickNPC
MoveTo / MoveToEx / GoTo
GetNearbySpritesWithPredicate
SelectTarget / ChaseTarget
GetSkillLuaData / GetSkillCooldown / RequestUsingSkillWithTarget
IsMapReady / GetCurrentMoveDestination
RoleData / LuaLeaderData.IsDeath
LuaSystemAPI_Network.SendPacket
MainThread.Execute(System.Action)
```

## MainThread rule

Exact `MainThread` member map is statically recovered. Before production mutations, prove one harmless managed Action callback on a live LD9 with correct lifetime/thread. Do not invoke Unity/Lua mutable action from arbitrary guest worker threads.

## Security/emulator caution

`libFGClientTool_Android.so` exposes emulator/ADB/accessibility/overlay/security-report surfaces. This proves observability, not a block condition. Research may document behavior and stability but must not attempt security/anti-cheat bypass.

## Resource boundary

`Interface.unity3d` contains the valuable Lua/UI corpus. Decrypted Shared/LoadingResources/Logo are mainly graphical resources; base `data.unity3d` exposes runtime types/resources but not the full PC-style Config database. Use PC Config only as donor identity until mobile downloaded/runtime data confirms it.

## Mandatory next step

Đọc `AUTO_TOOL_SCOPE.md`, sau đó `AI_ROUTER.md` và chọn đúng context pack.
