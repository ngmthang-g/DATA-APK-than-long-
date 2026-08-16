# Thần Long Mobile APK — Automation Research Knowledge Base

Repository này là **cơ sở dữ liệu kỹ thuật cho việc xây dựng tool EXE trên Windows điều khiển nhiều cửa sổ LDPlayer 9 chạy Thần Long Mobile APK**.

Mục tiêu không phải lưu mọi thứ về APK. Repo ưu tiên kiến thức trực tiếp giúp xây:

- Auto Train / đánh quái;
- chết -> Đầu thai -> quay lại map/bãi;
- Auto HP/MP và recovery;
- quét tay nải, giữ/bán/vứt/xóa theo policy;
- đi NPC, mở GameDialog/NPCShop, trị liệu/bán đồ;
- Auto Chat và ping tọa độ `@GOTO`;
- orchestration nhiều LD9 độc lập.

Repo được tổ chức theo cùng triết lý với data PC `ngmthang-g/clinent-game-than-long-DATA-2222`: đọc index/router trước, dùng catalog/database compact trước, chỉ mở deep analysis khi exact fact chưa có.

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
- kích thước: `77,318,940` bytes
- Unity `6000.3.6f1`, Android ARM64 + IL2CPP
- metadata v39
- ~15,667 type / 130,649 method / 68,049 field
- 1,186 original C# source-path strings recovered
- encrypted `Interface.unity3d` đã được giải mã, cho 631 TextAssets Lua/XML.

## Exact mobile facts đã giải được

```text
Train mode = 1
Start Train = AutoFight_Main:StartAutoFight(C_AutoModel.Train)

Normal Đầu thai = CMD_REVIVE_DATA 200063, payload "1"

Bag = ItemSite 10
Sell = CMD_NPC_SHOP_SELL_REQUEST 200036
Sell payload = itemInstanceID:NpcShopID:ShopID

GameDialog = 100007, payload selectionID:SelectedItemID

Chat = 100008
Chat content = Base64(text)
Location ping = @GOTO_<MapID>_<GridX>_<GridY>

Item action = 100005
Use=3, Abandon=4, Move=5, Destroy=9
```

Các fact trên là **mobile static evidence**, không còn là giả định lấy từ PC. Việc còn lại là runtime proof trên LD9: bridge safety, server acceptance, shop/healer cụ thể và multi-instance isolation.

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
