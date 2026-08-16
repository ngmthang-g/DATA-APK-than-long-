# Auto Feature Readiness — Mobile APK / LD9

Legend:

- `STATIC-SOLVED` — exact client semantic/source/action is recovered.
- `STATIC-STRONG` — sufficient state/API basis exists; some object/runtime binding remains.
- `RUNTIME-PROOF` — exact static action is known; live safety/acceptance must be proven.
- `DESIGN` — external architecture is defined but not live-tested.
- `PC-DONOR` — useful PC Config/identity hint, not mobile fact by itself.

| Feature | Status | Solved now | Remaining proof |
|---|---|---|---|
| IL2CPP metadata map | STATIC-SOLVED | v39 parser, class/method/field/token mapping | live resolver |
| Interface Lua/UI recovery | STATIC-SOLVED | decrypted UnityFS; 631 TextAssets | only other bundles if a concrete gap requires them |
| Local role snapshot schema | STATIC-STRONG | exact `RoleData`/HP/map/pos/team/etc. members | stable guest object acquisition |
| Bag/item schema | STATIC-STRONG | `Bag=10`, exact DBItemData identity fields | repeated live scan |
| Built-in Train semantic flow | STATIC-SOLVED | `Train=1`, `StartAutoFight`, target/chase/skill/range/return-center source | safe live invoke/soak |
| Loot/pickup engine | STATIC-SOLVED | nearby pack scan, semantic pickup, range/filter source | live pickup/interaction proof |
| Bag-full Sell trigger | STATIC-SOLVED | stock uses `GetFreeBagSpace`; pickup disables itself when insufficient | choose production threshold/tolerance |
| HP/MP medicine classification | STATIC-SOLVED | `IsHPMedicine` / `IsMPMedicine`, configured template -> live bag instance | live item/action proof |
| Item Use | RUNTIME-PROOF | `100005`, `3:itemInstanceID` | HP/item/cooldown proof |
| Item Abandon | RUNTIME-PROOF | `100005`, `4:itemInstanceID`, stock `IsItemThrowable` guard | live removal proof + explicit policy |
| Item Move | RUNTIME-PROOF | `100005`, `5:itemInstanceID:destinationSite` | destination/state proof |
| Bulk Destroy | STATIC-SOLVED but DANGEROUS | `100005`, `9:id1;id2;...` | normally disabled; only explicit narrow policy/live proof |
| Stock AutoDrop/AutoUsing toggles | DEFECT/STALE | UI/settings found but no reliable execution path; deserialize defect for DropItemSettings | do not depend on stock switches |
| Death detection | STATIC-STRONG | Role/Lua/GRole + death/revive lifecycle | live canonical source choice |
| Normal Revive request | RUNTIME-PROOF | exact `200063:"1"`; types 1/2/3 | live action/server completion |
| Return-to-train donor | STATIC-SOLVED | death map/position + `AutoComeback` route source | transition proof/tolerance |
| NPC navigation | STATIC-SOLVED | `GoToNPC` semantic source | chosen NPC availability |
| GameDialog action | STATIC-SOLVED | live Selections -> `100007 selectionID:-1` | chosen service sequence |
| NPC treatment | RUNTIME-PROOF | generic mechanism solved; PC healer candidate available | mobile healer/text/result |
| NPCShop open lifecycle | STATIC-SOLVED | `200034`, current shop data, guild-shop guard | vendor runtime path |
| Sell one item | RUNTIME-PROOF | exact `200036 itemInstanceID:NpcShopID:ShopID` | live removal/money proof |
| Auto Chat semantic send | RUNTIME-PROOF | `100008` object format, channel IDs, 200-char client limit | server acceptance/cooldown |
| Auto Chat scheduler | DESIGN | no stock periodic sender found; EXE must schedule/trigger semantic sends | runtime policy/rate limit |
| Location ping | RUNTIME-PROOF | exact `@GOTO_MapID_GridX_GridY` source | desired channel acceptance |
| MainThread bridge | RUNTIME-PROOF | exact dispatcher member map | harmless external Action callback |
| Multi-LD EXE | DESIGN | independent BotSession/action-gate architecture | discovery/channel/isolation/soak |
| Security compatibility | RUNTIME-PROOF | emulator/ADB/accessibility/report surfaces known | observe actual behavior; no bypass work |

## Facts promoted in deep static analysis

The following are no longer unresolved:

```text
Train high-level entrypoint
semantic loot + bag-full detection
Revive numeric request/payload
Sell numeric request/payload
GameDialog request/payload
Item Use / Abandon / Move / Destroy request formats
Chat numeric request/object
@GOTO location ping format
```

Known shipped-source defects/stale paths are recorded in `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md`. Do not waste future analysis re-solving exact facts above unless the APK version changes.
