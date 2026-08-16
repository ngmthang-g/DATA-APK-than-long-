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
- `database/FACTS.jsonl`
- `database/FINDING_TO_DOC_MAP.md`
- `analysis/27_AUTO_STATE_ACTION_PROOF_MATRIX.md`
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
package com.fgstudio.thanlongmobile
main activity com.unity3d.player.UnityPlayerActivity
```

Current static map:

```text
15,667 type definitions
130,649 methods
68,049 fields
1,186 original C# source-path strings
631 recovered Interface TextAssets
```

Canonical:

- `analysis/01_IL2CPP_RUNTIME_METADATA.md`
- `analysis/10_SOURCE_PATH_MANIFEST.md`
- `analysis/25_ANDROID_MANIFEST_PROCESS_BINDING.md`

---

# 3. Major reverse breakthrough — game Lua/UI recovered

`Interface.unity3d` was decrypted through the client native crypto path and parsed as UnityFS.

High-value recovered source:

```text
AutoFight_Main.lua
AutoHp.lua
PickUp.lua
Revival.lua
NPCShop.lua
NPCShop_SellItemTab.lua
GameDialog.lua
ChatBox.lua
Global_Constants.txt
Global_Functions.lua
TCPPacketDefine.txt
TCPCmdHandler.lua
TCPCmdEventHandler.lua
```

This means Train/Revive/Sell/GameDialog/Chat/item actions are no longer based only on metadata names or PC guesses.

Canonical:

- `analysis/11_INTERFACE_BUNDLE_DECRYPTION.md`
- `database/RECOVERED_LUA_EVIDENCE.csv`
- `database/AUTO_LUA_FUNCTION_CATALOG.csv`

---

# 4. Two packet layers — never mix them

### Engine/core examples

```text
CMD_CHANGE_MAP        16
CMD_REMOVE_ITEM       25
CMD_UPDATE_ITEMS_LIST 26
CMD_USE_SKILL         36
CMD_OBJECT_DEATH      41
CMD_AUTO_PATH         49
CMD_REVIVE            80
CMD_CHAT_DATA         89
CMD_CAPTCHA            96
CMD_CLIENT_LUA        100
```

### Lua gameplay examples

```text
CMD_ITEM_ACTION             100005
CMD_BAG_SORT                100006
CMD_SHOW_GAMEDIALOG         100007
CMD_CLIENT_CHAT             100008
CMD_SHARED_PARAMETER        200024
CMD_NPC_SHOP_DATA           200034
CMD_NPC_SHOP_SELL_REQUEST   200036
CMD_REVIVE_DATA             200063
```

So `CMD_REVIVE=80` and normal UI request `CMD_REVIVE_DATA=200063` can both be correct because they belong to different layers.

Canonical:

- `analysis/12_PACKET_LAYERS_AND_LUA_PROTOCOL.md`
- `database/PACKET_IDS_LUA_MOBILE.csv`

---

# 5. Built-in Auto Train — semantic engine solved

Exact mode/start:

```text
C_AutoModel.Train = 1
AutoFight_Main:StartAutoFight(C_AutoModel.Train)
```

Stop/yield:

```text
StartAutoFight(C_AutoModel.None)
```

Source confirms target scan/filter, range/path guards, select/chase, skill use, loot, return-to-center and death-comeback donor logic.

Important implementation facts:

- default `RangerScan=500`;
- Train refuses to silently replace active Quest mode;
- accepted Train start disables `EnableAutoF1`, clears flag state and enters `AutoTrainStart()`;
- `StopAllCurrentTask()` is a delayed UI wrapper, not the preferred direct stop primitive;
- do not run a second external combat loop while the stock Train engine owns combat.

Canonical:

- `analysis/13_BUILTIN_AUTO_TRAIN_ENGINE.md`
- `features/AUTO_TRAIN.md`
- `contexts/BUILD_AUTO_TRAIN.md`

---

# 6. Loot / bag-full trigger — no periodic bag-window scan needed

Stock pickup uses:

```text
Game.GetFreeBagSpace()
```

and disables pickup when space is insufficient. Source often preserves at least one spare slot (`>1`, `>#items`).

Recommended EXE trigger:

```text
FreeBagSpace <= configured SellThreshold
 -> yield Train/pickup
 -> Auto Sell transaction
```

Canonical:

- `analysis/22_LOOT_BAG_FULL_AND_SELL_TRIGGER.md`
- `features/AUTO_SELL.md`

---

# 7. HP/MP recovery and item actions — exact

Lua gameplay item packet:

```text
CMD_ITEM_ACTION = 100005
```

Key actions:

```text
Use     = 3  -> 3:itemInstanceID
Abandon = 4  -> 4:itemInstanceID
Move    = 5  -> 5:itemInstanceID:destinationSite
Merge   = 7  -> 7:id1;id2;...
Split   = 8  -> 8:itemInstanceID:quantity
Destroy = 9  -> 9:id1;id2;...
```

Recovery classification:

```text
Game.IsHPMedicine(ItemID)
Game.IsMPMedicine(ItemID)
```

Stock `DoAutoRegen()` has a timestamp weakness: successful Use branches return before updating `LastTrigerHpRegen`. External recovery should use **one action → HP/item/cooldown proof → rescan**.

Abandon preserves `IsItemThrowable`; Destroy is destructive and disabled by default in the recommended tool design.

Canonical:

- `analysis/20_ITEM_ACTIONS_AND_AUTO_RECOVERY.md`
- `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md`
- `features/AUTO_RECOVERY.md`
- `database/ITEM_ACTIONS_MOBILE.csv`

---

# 8. Death / Revive / return map — exact request solved

Exact mobile normal revive / Đầu thai:

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

Stock AutoFight records death MapID/position and has `AutoComeback`. Map 87 receives special route handling.

Recommended external proof:

```text
dead generation
 -> one 200063:"1"
 -> fresh alive/HP/Revival proof
 -> IsMapReady
 -> Game.GoTo(saved TrainMap,savedPos)
 -> MapID + position tolerance
 -> resume Train
```

Canonical:

- `analysis/14_REVIVE_RETURN_MAP_ENGINE.md`
- `features/AUTO_REVIVE.md`

---

# 9. Movement / AutoPath / cross-map routing

High-level APIs:

```text
MoveTo(x,y)
MoveToEx(x,y,completed,cancelled)
GoTo(mapID,x,y,completed)
GetCurrentMoveDestination()
IsMapReady()
```

`GoToNPC` shows stock cross-map pattern:

```text
GoTo(targetMap,-1,-1)
 -> GetNPCPosition(npcID)
 -> GoTo(targetMap,npcX,npcY)
 -> ClickNPC(npcID)
```

PathFinder contains portal-only, teleport-item and no-teleport route models. Initial EXE should delegate route planning to `Game.GoTo`, not reimplement the graph externally.

Canonical: `analysis/03_WORLD_ENTITY_MAP_PATH.md`.

---

# 10. Auto Sell — exact request solved

Bag:

```text
ItemSite.Bag = 10
```

Current normal shop arrives via:

```text
CMD_NPC_SHOP_DATA = 200034
```

Require:

```text
current shopData
IsGuildShop == false
current NpcShopID + current ShopID
```

Exact sell:

```text
CMD_NPC_SHOP_SELL_REQUEST = 200036
payload = DBItemData.ID:NpcShopID:ShopID
```

The first value is the **live instance ID**.

Stock game guards:

```text
40000000 <= ItemID < 50000000 -> do not sell
Game.IsItemSellable(ItemID) must be true
```

Correct transaction:

```text
fresh Bag
 -> one candidate
 -> one sell
 -> RemoveItem/UpdateItemsList proof
 -> fresh Bag
```

No cached “sell 90 slots” loop.

Canonical:

- `analysis/15_INVENTORY_NPCSHOP_AUTO_SELL.md`
- `features/AUTO_SELL.md`
- `database/AUTO_SELL_CLASSIFICATION.md`

---

# 11. Sell classification — game permission ≠ user policy

Observed semantic types include:

```text
CommonItem
ScriptItem
Medicine
Gem
PetEquip
Equip
Iron
Rice
Salt
Undefined
```

Recommended default is conservative:

- protect configured recovery medicine;
- protect quest range and `IsItemSellable=false`;
- KEEP unknown/undefined;
- KEEP Equip/PetEquip unless an explicit profile rule permits sale;
- no numeric shortcut such as `GetEquipType < 10 => weapon`;
- no icon/OCR classification;
- PC `EquipPoint==0` weapon rule stays PC-DONOR until mobile static/runtime template evidence supports it.

Canonical: `database/AUTO_SELL_CLASSIFICATION.md`.

---

# 12. NPC / Trị liệu — generic mechanism solved, healer runtime still open

Route:

```text
GetNPCPosition -> GoTo -> ClickNPC
```

Dynamic server dialog:

```text
Selections[selectionID] = visibleText
```

Selection request:

```text
CMD_SHOW_GAMEDIALOG = 100007
payload = actualSelectionID:SelectedItemID
ordinary service default SelectedItemID = -1
```

Important: the packaged string `Trị liệu` also names the Auto HP tab; it is **not static proof of one healer dialog option**.

PC Lâu Lan NPC 339 Đỗ Thanh Đằng is a strong donor candidate, but mobile runtime must confirm current identity/text/result.

Canonical:

- `analysis/16_GAMEDIALOG_NPC_TREATMENT.md`
- `features/AUTO_HEAL_NPC.md`

---

# 13. Auto Chat / coordinate ping — exact semantic send solved

Exact packet:

```text
CMD_CLIENT_CHAT = 100008
```

Object:

```text
RoleID
Name
Content = Base64(text)
Channel = actual channel ID
```

Actual channel IDs include:

```text
Guild=2
Team=4
Near=5
Faction=6
Private=7
Global=8
Special=9
CrossServer=10
```

Do **not** send ChatBox dropdown `SelectedID`; it is only a UI index translated by stock source to an actual channel ID.

Location token:

```text
@GOTO_<MapID>_<GridX>_<GridY>
```

Stock UI practical length limit = 200 chars. No explicit local send timer is present; server may throttle, so the EXE implements conservative per-LD rate limiting.

No stock periodic Auto-Chat sender was found. Scheduling/event-triggered chat belongs to the EXE while still using the semantic packet.

Canonical:

- `analysis/17_CHAT_CHANNEL_AND_GOTO_AUTOMATION.md`
- `features/AUTO_CHAT.md`
- `database/CHAT_CHANNELS_MOBILE.csv`

---

# 14. Central inbound observer — copy transient state, do not retain UI pointers

Recovered global Lua dispatchers:

```text
ReceivePacket(packetID,data)
ReceiveEvent(eventType,data)
```

High-value inbound state:

```text
100007 -> current GameDialog
200034 -> current NPCShop
200063 -> current Revival lifecycle
```

High-value core events:

```text
RemoveItem=2        data site:dbID:position
UpdateItemsList=3
ChatEvent=50
NewCaptcha=57
```

Recommended guest scanner copies semantic transaction fields into versioned snapshots and lets original client handlers continue.

Captcha event is a pause/manual-safety signal only; no bypass logic.

Canonical:

- `analysis/26_LUA_PACKET_EVENT_OBSERVER.md`
- `database/RUNTIME_EVENTS_MOBILE.csv`

---

# 15. Runtime snapshot / generation contract

Recommended per-LD state includes:

```text
AndroidGamePID
ResolverGeneration
WorldGeneration
SnapshotVersion
Role/map/HP/MP/death
movement/AutoPath/target
FreeBagSpace + BagVersion + BagItems
DialogGeneration
ShopGeneration
RevivalGeneration
Captcha/safety
last action/proof
```

Raw managed/native/Lua UI pointers never cross into reusable Windows-host state.

An action should be rejected if the generation it was decided against is stale at execution time.

Canonical: `analysis/18_RUNTIME_ROLE_BAG_SNAPSHOT.md`.

---

# 16. Lua/UI action bridge

Static mobile client exposes:

```text
LuaSystemManager.GetScript(uiName)
LuaSystemManager.ExecuteFunction(...)
LuaSystemAPI_GUI.FindUI / CallUI
LuaSystemAPI_Network.SendPacket
MainThread.Execute(System.Action)
```

Recommended split:

- direct Game APIs for state/movement/simple semantic actions;
- exact Network.SendPacket for proven one-packet actions while preserving stock guards;
- game-owned Lua service/method for stateful engines such as Auto Train.

Exact external closure/object ABI/lifetime remains runtime proof. Do not inject arbitrary Lua text when a loaded game service already exists.

Canonical:

- `analysis/24_LUA_UI_ACTION_BRIDGE_BLUEPRINT.md`
- `contexts/BUILD_MAINTHREAD_BRIDGE.md`

---

# 17. Multi-LD Windows EXE architecture

```text
ThanLongAuto.exe
 -> LD Instance Manager
 -> ADB serial namespace
 -> package com.fgstudio.thanlongmobile
 -> current Android PID
 -> one independent BotSession
 -> guest ARM64 bridge
 -> resolver/scanner/action adapter
```

Each LD owns its own:

```text
profile
snapshots/generations
action gate
current target
bag instance IDs
shop/dialog transactions
last action/proof
```

No mutable state sharing across LDs.

Tool settings should use `database/EXE_PER_LD_PROFILE_SCHEMA.md`, not overload the game's legacy AutoSettings string.

Canonical:

- `analysis/05_LD9_HOST_GUEST_ARCHITECTURE.md`
- `analysis/19_LD9_ACTION_ORCHESTRATION.md`
- `analysis/25_ANDROID_MANIFEST_PROCESS_BINDING.md`

---

# 18. Built-in AutoSettings mobile revision

Mobile snapshot:

```text
AUTOVERSION_DEFINE = "3.5"
```

Current PC frozen repo analyzed separately uses `4.1`.

Core Train/Pick/Recovery structure is related, but FUBEN schema differs materially. This is direct evidence that PC/mobile share much code but are **not the same client revision**.

The mobile source also contains a verified PickItem loader mismatch for `DropItemSettings`.

Canonical:

- `database/BUILTIN_AUTO_SETTINGS_MOBILE.md`
- `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md`

---

# 19. Packaged resource boundary

The base APK does not expose the same full static Config corpus as the PC frozen client.

Decrypted Shared/LoadingResources/Logo are primarily graphics/shared assets. Base `data.unity3d` exposes runtime/resources/type identities, not the full current NPC/Items/Maps tables.

Therefore:

```text
semantic actions -> APK Lua + IL2CPP
static identity donor -> PC data when useful
current truth -> LD9 runtime/downloaded resources
```

Do not silently promote PC NPC/item/map rows to mobile VERIFIED.

Canonical: `analysis/21_PACKAGED_RESOURCE_BOUNDARY.md`.

---

# 20. Known shipped-source defects/stale paths

Do not clone stock code blindly. Confirmed concerns include:

- PICKITEM loader assigns `DropItemSettings` from the same field as `IsAutoDropItem`;
- AutoUsingItem/AutoDrop settings exist but no reliable general execution loop was found in recovered AutoFight source;
- successful Auto HP/MP Use returns before updating the inner timestamp limiter;
- loot free-space comparisons are conservative/asymmetric;
- mobile symbolic Nga My `KIMCHAMDOKIEP=407` conflicts with independently verified PC Config identity, so variable label alone is not skill truth.

Canonical: `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md`.

---

# 21. Canonical state/action/proof matrix

Before coding a mutable feature, read:

`analysis/27_AUTO_STATE_ACTION_PROOF_MATRIX.md`

It defines implementation contracts for:

```text
Train start/yield
Recovery
Sell trigger/route/shop/sell/return
Revive/return
NPC treatment
Chat/ping
Abandon/Destroy
Captcha safety
reconnect/process restart
multi-LD isolation
```

This is the main orchestration blueprint.

---

# 22. Current real gaps

Broad APK reverse is no longer useful for the core Train/Revive/Sell/Chat packet questions. High-value remaining work is runtime proof:

1. stable per-LD read-only resolver/snapshot and generation invalidation;
2. harmless managed Action → MainThread callback;
3. live Lua/UI lookup/action bridge ABI/lifetime;
4. safe Train start/stop/soak on LD9;
5. map-change/return-to-spot proof;
6. one normal revive acceptance/completion using `200063:"1"`;
7. one mobile vendor → current non-guild NPCShop → one `200036` sell → removal proof;
8. HP/MP `Use=3` proof and arbitration with stock recovery;
9. current healer identity + dynamic treatment dialog/result;
10. desired chat channel acceptance/rate-limit behavior;
11. multi-LD isolation soak.

Only decrypt/reverse another resource bundle when one of these concrete implementation gaps cannot be solved from current static knowledge + runtime proof.
