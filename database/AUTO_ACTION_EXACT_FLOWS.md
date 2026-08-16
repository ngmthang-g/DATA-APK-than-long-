# Exact action flows — mobile compact lookup

## Train

```text
GUI.FindUI("AutoFight_Main"):StartAutoFight(C_AutoModel.Train)
C_AutoModel.Train = 1
stop/yield -> StartAutoFight(C_AutoModel.None)
```

## Normal revive / Đầu thai

```text
packet 200063
payload "1"
```

## NPC navigation

```text
GetNPCPosition -> GoTo -> ClickNPC
```

## GameDialog selection

```text
packet 100007
payload selectionID:SelectedItemID
default SelectedItemID = -1
```

## Sell one item

```text
packet 200036
payload itemInstanceID:NpcShopID:ShopID
itemInstanceID = DBItemData.ID
```

## Chat

```text
packet 100008
object { RoleID, Name, Content=Base64(text), Channel }
```

## Ping current location in chat

```text
@GOTO_<MapID>_<GridX>_<GridY>
```

## Proof discipline

Every mutable action remains:

```text
fresh state -> guard -> one action -> observed result -> fresh state
```
