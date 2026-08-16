# Exact action flows — mobile compact lookup

Use this file for quick implementation lookup. For guard/proof/failure semantics read `analysis/27_AUTO_STATE_ACTION_PROOF_MATRIX.md`.

## Train

```text
C_AutoModel.Train = 1
start -> AutoFight_Main:StartAutoFight(C_AutoModel.Train)
stop/yield -> AutoFight_Main:StartAutoFight(C_AutoModel.None)
```

Do not repeatedly start Train while an incompatible stock mode such as Quest is active.

## Map / NPC

```text
cross-map -> Game.GoTo(mapID,-1,-1,callback)
NPC pos -> Game.GetNPCPosition(npcID,...)
NPC route -> Game.GoTo(mapID,npcX,npcY,callback)
interact -> Game.ClickNPC(npcID)
```

Final proof: IsMapReady + expected MapID + fresh position/proximity.

## Bag-full Sell trigger

```text
FreeBagSpace = Game.GetFreeBagSpace()
trigger Sell when FreeBagSpace <= configured threshold
```

No periodic bag-window counting is required.

## Item Use / Recovery

```text
CMD_ITEM_ACTION = 100005
Use = 3
payload = 3:itemInstanceID
```

Use fresh `DBItemData.ID`; proof HP/MP/item state before another action.

## Abandon item

```text
100005
payload = 4:itemInstanceID
```

Require `Game.IsItemThrowable(ItemID)` plus explicit drop policy.

## Move item

```text
100005
payload = 5:itemInstanceID:destinationSite
```

## Destroy items

```text
100005
payload = 9:id1;id2;...
```

Destructive; disabled by default.

## Normal revive / Đầu thai

```text
CMD_REVIVE_DATA = 200063
payload = "1"
```

Newbie=`"2"`, skill=`"3"`.

## GameDialog selection

```text
CMD_SHOW_GAMEDIALOG = 100007
payload = actualSelectionID:SelectedItemID
ordinary service default SelectedItemID = -1
```

Resolve the current selection from inbound dialog data; do not persist one selection ID globally.

## Shop readiness

```text
inbound CMD_NPC_SHOP_DATA = 200034
require current shopData
require IsGuildShop == false
capture current NpcShopID + shop ID for this transaction generation
```

## Sell one item

```text
CMD_NPC_SHOP_SELL_REQUEST = 200036
payload = itemInstanceID:NpcShopID:ShopID
itemInstanceID = current DBItemData.ID
```

Proof with RemoveItem / UpdateItemsList + fresh Bag rescan.

## Chat

```text
CMD_CLIENT_CHAT = 100008
object {
  RoleID,
  Name,
  Content = Base64(text),
  Channel = actual channel ID
}
```

Do not send ChatBox dropdown index as Channel.

## Ping current location in chat

```text
grid = Game.WorldToGridPosition(RoleData.Position)
@GOTO_<MapID>_<GridX>_<GridY>
```

Append to normal chat content, preserve practical 200-character envelope.

## Dynamic inbound proof sources

```text
ReceivePacket(100007,data) -> copy current GameDialog generation/selections
ReceivePacket(200034,data) -> copy current NPCShop generation/state
ReceivePacket(200063,data) -> copy Revival generation/state

ReceiveEvent(RemoveItem=2, "site:dbID:position")
ReceiveEvent(UpdateItemsList=3,...)
ReceiveEvent(NewCaptcha=57,...)
```

## Proof discipline

Every mutable action remains:

```text
fresh state
 -> generation/feature guard
 -> ONE action
 -> observed result or bounded failure
 -> fresh state
```
