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

Each button stores the live `selectionID`. Selection sends:

```text
CMD_SHOW_GAMEDIALOG = 100007
payload = selectionID:SelectedItemID
```

`SelectedItemID` defaults to `-1` for ordinary service choices.

## Treatment implication

There is no verified global constant for one fixed `Trị liệu` selection number. Robust action:

```text
GoToNPC
 -> wait current GameDialog
 -> enumerate live Selections
 -> normalize visible text
 -> match configured semantic text such as "trị liệu"
 -> send actual selectionID:-1
 -> observe next dialog / HP / money / close state
```

Never hardcode a selection ID because one opening happened to use it.

## PC donor candidate

PC Config verifies Lâu Lan Map 5 NPC 339 `Đỗ Thanh Đằng`, ResName `LangZhong1`, as a strong healer candidate. Because the mobile APK snapshot does not embed the same full Config rows in the recovered Interface bundle, mobile status remains **PC-DONOR candidate** until LD9 runtime interaction confirms the current identity/service.

## Completion proof

Preferred proof:

```text
HP/MaxHP before
 -> service selection sent
 -> HP increases/restores and/or expected money change
 -> dialog transition/close
 -> fresh role snapshot
```
