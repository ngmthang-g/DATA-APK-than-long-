# Feature — Auto Trị liệu NPC

Status: **NAVIGATION + DIALOG ACTION MECHANISM SOLVED STATIC; chosen healer/runtime service proof pending**

## Semantic route

```text
GetNPCPosition(npcID)
 -> GoTo(mapID,npcX,npcY)
 -> ClickNPC(npcID)
 -> wait GameDialog
```

or reuse the shipped `GoToNPC(mapID,npcID)` donor.

## Selection

Never click a fixed screen button. Read current `GameDialog.Selections`, match configured visible text (`trị liệu` and any verified server wording), then send:

```text
CMD_SHOW_GAMEDIALOG = 100007
payload = actualSelectionID:-1
```

## Completion

Require fresh HP/money/dialog state. If there is a second confirmation dialog, treat it as another dynamic selection step rather than a fixed sleep/click.

## Candidate

PC data has Lâu Lan Map 5 / NPC 339 Đỗ Thanh Đằng as a strong healer candidate. Mobile runtime must confirm before marking it canonical.
