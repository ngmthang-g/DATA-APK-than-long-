# AI Knowledge Index — Thần Long Mobile APK / LDPlayer 9

Repository: `ngmthang-g/DATA-APK-than-long-`

Canonical purpose: **technical memory for a Windows EXE that manages multiple LDPlayer 9 instances running the Thần Long Mobile APK**. Each LD instance is one independent BotSession.

---

# 1. Start here — do not read the whole repo

Normal route:

```text
AI_BOOTSTRAP.md
 -> AUTO_TOOL_SCOPE.md
 -> AI_ROUTER.md
 -> one matching contexts/BUILD_*.md
 -> compact database/action catalog
 -> deep analysis only for a real gap
```

Best compact references:

- `AUTO_FEATURE_READINESS.md`
- `database/AUTO_TOOL_API_CATALOG.md`
- `database/AUTO_TOOL_ACTION_CATALOG.md`
- `database/AUTO_ACTION_EXACT_FLOWS.md`
- `database/PACKET_IDS_LUA_MOBILE.csv`
- `research/AUTO_RUNTIME_PROOF_QUEUE.md`

---

# 2. Frozen APK snapshot

```text
ThanLongMobile_2024.apk
SHA256 9de719391c29d50816a3762f758abe26896cab4d996706711b30fdc10d9933f0
size   77,318,940 bytes
Unity  6000.3.6f1
arch   ARM64
IL2CPP metadata version 39
```

Core native/runtime files include `libil2cpp.so`, `libunity.so`, `libFGClientTool_Android.so` and `global-metadata.dat`.

Metadata parse currently maps approximately:

```text
15,667 type definitions
130,649 methods
68,049 fields
1,186 original C# source-path strings
```

Canonical: `analysis/01_IL2CPP_RUNTIME_METADATA.md`, `analysis/10_SOURCE_PATH_MANIFEST.md`.

---

# 3. Major Phase-3 breakthrough — decrypted Interface Lua/UI

`Interface.unity3d` was decrypted using the client native crypto implementation, parsed as UnityFS and its TextAssets were extracted.

Recovered automation source includes:

```text
AutoFight_Main.lua
Revival.lua
NPCShop.lua
NPCShop_SellItemTab.lua
GameDialog.lua
ChatBox.lua
Global_Constants.txt
Global_Functions.lua
TCPPacketDefine.txt
TCPCmdHandler.lua
```

This changes many earlier `TARGETED-PROOF` items into exact static mobile facts.

Canonical: `analysis/11_INTERFACE_BUNDLE_DECRYPTION.md`.

---

# 4. Packet model — two command layers

Do not confuse:

1. C#/engine `TCPGameServerCmds` low-level events (`CMD_REVIVE=80`, `CMD_CHAT_DATA=89`, etc.);
2. Lua gameplay `G_TCPPacketDefine` requests (`CMD_REVIVE_DATA=200063`, `CMD_CLIENT_CHAT=100008`, etc.).

Canonical: `analysis/12_PACKET_LAYERS_AND_LUA_PROTOCOL.md` and `database/PACKET_IDS_LUA_MOBILE.csv`.

---

# 5. Auto Train — semantic engine solved

Mobile source independently verifies:

```text
C_AutoModel.Train = 1
AutoFight_Main:StartAutoFight(C_AutoModel.Train)
```

Built-in Train uses:

```text
GetNearbySpritesWithPredicate
IsDeath / RoleData
GetSkillLuaData
HasPath
StopAutoPath
SelectTarget
ChaseTarget
RequestUsingSkillWithTarget
MoveToEx
```

It includes target filtering, skill/range/path guards, loot, return-to-center and death-comeback donor logic.

Canonical: `analysis/13_BUILTIN_AUTO_TRAIN_ENGINE.md`, `features/AUTO_TRAIN.md`.

---

# 6. Death / Revive / return map — request solved

Exact normal revive / Đầu thai request from mobile Lua:

```text
CMD_REVIVE_DATA = 200063
payload = "1"
```

Types:

```text
1 normal
2 newbie
3 skill
```

Built-in AutoFight records death MapID/position and has `AutoComeback` route logic. Map 87 is treated specially as infernal/death map; the stock donor routes through Map 2 before returning to the saved death map/position.

Canonical: `analysis/14_REVIVE_RETURN_MAP_ENGINE.md`, `features/AUTO_REVIVE.md`.

---

# 7. Auto Sell — exact request solved

Bag site is mobile-verified:

```text
ItemSite.Bag = 10
```

Exact sell request:

```text
CMD_NPC_SHOP_SELL_REQUEST = 200036
payload = DBItemData.ID : CurrentShopData.NpcShopID : CurrentShopData.ID
```

The first value is the **live item instance ID**, not template ItemID or bag Position.

Stock Lua protects quest-item range `40000000 <= ItemID < 50000000` and requires `Game.IsItemSellable(ItemID)`.

Current normal shop must have `IsGuildShop == false`.

Canonical: `analysis/15_INVENTORY_NPCSHOP_AUTO_SELL.md`, `features/AUTO_SELL.md`.

---

# 8. NPC / Trị liệu

Built-in semantic route:

```text
GetNPCPosition -> GoTo -> ClickNPC
```

`GameDialog` dynamically exposes `Selections[selectionID] = visibleText`. Exact selection request:

```text
CMD_SHOW_GAMEDIALOG = 100007
payload = selectionID:SelectedItemID
```

Default non-award selection uses `SelectedItemID=-1`.

Therefore Trị liệu should match current visible text, not hardcode a button coordinate or guessed selection ID.

PC Config gives NPC 339 Đỗ Thanh Đằng on Lâu Lan Map 5 as a strong donor candidate; mobile runtime must still confirm current service.

Canonical: `analysis/16_GAMEDIALOG_NPC_TREATMENT.md`, `features/AUTO_HEAL_NPC.md`.

---

# 9. Chat and coordinate ping — solved static

Exact chat request:

```text
CMD_CLIENT_CHAT = 100008
packet object: RoleID, Name, Content=Base64(text), Channel
```

Exact coordinate attachment:

```text
@GOTO_<MapID>_<GridX>_<GridY>
```

The client produces it from `RoleData.MapID + WorldToGridPosition(RoleData.Position)`. Receiver-side navigation resolves the link and calls `Game.GoTo`.

Canonical: `analysis/17_CHAT_CHANNEL_AND_GOTO_AUTOMATION.md`, `features/AUTO_CHAT.md`, `database/CHAT_CHANNELS_MOBILE.csv`.

---

# 10. Runtime snapshot basis

High-value mobile state roots now have exact class/method mapping, including `RoleData` (199 methods / 99 fields) and `DBItemData`.

Snapshot should include role identity, map/position, HP/MP/death, map readiness, selected target, auto path state, free bag space/bag instances, current dialog/shop and safety state.

Canonical: `analysis/18_RUNTIME_ROLE_BAG_SNAPSHOT.md`.

---

# 11. Multi-LD architecture

```text
ThanLongAuto.exe
 -> LD instance manager
 -> one independent BotSession per LD9
 -> guest command/state channel
 -> ARM64 guest bridge
 -> IL2CPP resolver / semantic Lua+Game adapter
```

Do not share guest pointers or mutable state between instances. One mutable action gate per BotSession.

Canonical: `analysis/05_LD9_HOST_GUEST_ARCHITECTURE.md`, `analysis/19_LD9_ACTION_ORCHESTRATION.md`.

---

# 12. MainThread

`FGStudio.Engine.Utilities.MainThread` exact member map is recovered (`Instance`, `Execute(Action)`, `Update`, `DoExecuteWorks`, queue field). A separate `UnityMainThreadDispatcher` is also present.

External guest construction/rooting of a valid managed Action and one harmless live callback remain runtime proof requirements before production mutations.

---

# 13. Current real gaps

Broad APK reverse is no longer the highest-value work for Train/Revive/Sell/Chat.

Remaining important runtime proofs:

1. stable per-LD read-only resolver/snapshot;
2. harmless MainThread action bridge;
3. one normal revive acceptance/completion using mobile-proven `200063:"1"`;
4. one mobile vendor path -> current non-guild NPCShop -> one safe `200036` sell -> removal proof;
5. chosen healer runtime dialog text/result;
6. chat acceptance/rate-limit behavior for desired channels;
7. cross-map return-to-train and multi-LD isolation soak test.
