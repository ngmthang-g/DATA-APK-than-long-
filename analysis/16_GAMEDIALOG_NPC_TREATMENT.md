# NPC service / Trị liệu through GameDialog

Status: **GENERIC DIALOG MECHANISM VERIFIED STATIC; exact healer selection/result is runtime-dependent**

## Exact NPC navigation donor

Recovered `Global_Functions.lua` defines `GoToNPC(mapID,npcID)` with documented intent to automatically find the route and talk to the target NPC.

Flow:

```text
if current map != target map:
    Game.GoTo(mapID,-1,-1, callback)
GetNPCPosition(npcID)
Game.GoTo(mapID,npcX,npcY, callback)
Game.ClickNPC(npcID)
```

This removes the need for hardcoded screen coordinates or user-entered NPC X/Y when NPC ID is known.

## Dynamic GameDialog

`GameDialog.lua` receives `gameDialogData` and builds buttons from:

```text
gameDialogData.Selections[selectionID] = visibleText
```

It also consumes:

```text
gameDialogData.Title
gameDialogData.Message  # Base64-decoded by UI
gameDialogData.Awards
```

Each selection button stores the live `selectionID`. Selection sends:

```text
CMD_SHOW_GAMEDIALOG = 100007
payload = selectionID:SelectedItemID
```

`SelectedItemID` defaults to `-1` for ordinary service choices.

## Strong stock donor: Auto Quest already performs semantic text matching

`AutoFight_Main:ProcessGameDialog(gameDialogData)` proves the shipped auto does **not** need to click a visible GameDialog button when automating a known semantic choice.

It iterates:

```text
for selectionID, selectionName in pairs(gameDialogData.Selections)
```

matches visible text such as:

```text
[NHẬN]
[TRẢ]
Nhận thưởng
Tiếp nhận
```

then sends directly:

```text
CMD_SHOW_GAMEDIALOG = 100007
selectionID:itemPick
```

For a simple acknowledgement it recognizes `selectionID == 99999` or text `Ta biết rồi` and sends `selectionID:-1`.

This is very strong evidence for the same architecture in NPC treatment:

```text
server dialog -> copy live Selections -> semantic text matcher -> exact selection packet
```

rather than screen-coordinate click automation.

## Stock Quest timing is donor policy, not requirement

`ProcessGameDialog()` waits a fixed two seconds after opening the dialog before auto-selecting. That may reduce UI/race issues in the stock Quest coroutine, but it is not evidence that packet 100007 requires two seconds.

An external tool should instead require the **current inbound DialogGeneration** and its copied Selections, then execute one guarded action when the transaction is ready.

## Important observer boundary

`GameDialog.lua` uses the inbound `gameDialogData` during initialization; it does not expose the original server object as a stable public `Selections` field on the UI script.

`AutoFight_Main:PutGameDialog()` only retains/processes the object while current mode is Quest.

Therefore the production scanner should copy the current dialog at inbound `ReceivePacket(100007,data)` time:

```text
DialogGeneration
Title
Message
Selections[] { ID, Text }
Awards summary if needed
ReceivedAt
```

Do not rely on a persistent `GUI.FindUI("GameDialog").Selections` pointer.

Canonical observer: `analysis/26_LUA_PACKET_EVENT_OBSERVER.md`.

## Treatment implication

There is no verified global constant for one fixed `Trị liệu` selection number. Robust action:

```text
GoToNPC
 -> wait current inbound GameDialog generation
 -> enumerate copied live Selections
 -> normalize visible text
 -> uniquely match configured treatment/service matcher
 -> send actual selectionID:-1
 -> observe next dialog / HP / money / close state
```

If multiple selections match, do not guess. Surface the current list in development logs/UI and require the policy to disambiguate by exact/contains/regex-like normalized matcher semantics chosen by the tool design.

Never hardcode a selection ID because one opening happened to use it.

## `Trị liệu` string ambiguity

The recovered APK does contain the Vietnamese string `Trị liệu`, but one confirmed occurrence is the **Auto HP/MP settings tab label**. This does not prove the server's healer GameDialog currently uses exactly that visible wording.

Treat actual healer selection text as runtime data.

## PC donor candidate

PC Config verifies Lâu Lan Map 5 NPC 339 `Đỗ Thanh Đằng`, ResName `LangZhong1`, as a strong healer candidate. Because the mobile APK snapshot does not embed the same full Config rows in the recovered Interface bundle, mobile status remains **PC-DONOR candidate** until LD9 runtime interaction confirms the current identity/service.

## Completion proof

Preferred proof:

```text
HP/MaxHP before
 -> current dialog selection sent
 -> possible next confirmation dialog handled as a new generation
 -> HP increases/restores and/or expected money change
 -> dialog transaction closes/completes
 -> fresh role snapshot
```

For production, `HP restored` is stronger proof than merely seeing the dialog disappear.
