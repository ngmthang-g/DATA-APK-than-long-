# Thần Long Mobile APK — Automation Research Knowledge Base

Repository này là **cơ sở dữ liệu kỹ thuật cho việc xây dựng tool EXE trên Windows điều khiển nhiều cửa sổ LDPlayer 9 chạy Thần Long Mobile APK**.

Mục tiêu của repo **không phải** lưu mọi thứ về APK. Chỉ ưu tiên kiến thức giúp xây Auto Train, chết → hồi sinh → tự chạy lại bãi, quét tay nải → tự bán đồ → quay lại train, và orchestration nhiều LD9 độc lập.

Repo được tổ chức theo cùng triết lý với data PC `ngmthang-g/clinent-game-than-long-DATA-2222`: đọc index/router trước, dùng catalog/database compact trước, chỉ mở deep analysis khi fact chính xác chưa có.

## Điểm xuất phát

Đọc theo thứ tự:

1. `AI_BOOTSTRAP.md`
2. `AUTO_TOOL_SCOPE.md`
3. `AI_ROUTER.md`
4. đúng một `contexts/BUILD_*.md`
5. `database/AUTO_TOOL_API_CATALOG.md` / `database/AUTO_TOOL_ACTION_CATALOG.md`
6. deep `analysis/` chỉ khi cần bằng chứng.

## Snapshot APK hiện tại

- file nghiên cứu: `ThanLongMobile_2024.apk`
- SHA-256: `9de719391c29d50816a3762f758abe26896cab4d996706711b30fdc10d9933f0`
- kích thước upload: `77,318,940` bytes
- Unity Android ARM64 + IL2CPP
- `libil2cpp.so`: 90,658,536 bytes
- `global-metadata.dat`: 14,475,116 bytes
- metadata header version: `39` (`0x27`)
- có `Assembly-CSharp.dll` trong `ScriptingAssemblies.json`
- package được Version.xml trỏ tới: `com.fgstudio.thanlongmobile`

## Kiến trúc mục tiêu

```text
Windows ThanLongAuto.exe
  -> LD Instance Manager
  -> BotSession riêng cho từng LD9
  -> guest/ARM64 bridge trong từng Android instance
  -> IL2CPP resolver
  -> read-only snapshot
  -> observer/state machine/safety guard
  -> Action Gate: tối đa 1 mutable action / instance
  -> MainThread/semantic game API
  -> proof -> rescan
```

Không dùng pixel/OCR làm nguồn chân lý nếu game đã expose state semantic. Không dùng offset PC cho mobile. Không dùng chung live pointer/state giữa các LD9.

## Trạng thái hiện tại

Static APK đã xác nhận nhiều primitive rất có giá trị: bag/item, NPC, movement/path, target/combat, death/revive processing, Lua/network bridge, MainThread và security/emulator surface.

Hai mắt xích quan trọng **chưa được nâng lên VERIFIED trên mobile**:

1. exact outbound revive request packet ID/payload;
2. exact outbound sell request packet ID/payload/current shop requirements.

PC có các fact tương ứng nhưng chỉ được dùng làm **cross-platform hypothesis/donor**, không được mặc định giống mobile.
