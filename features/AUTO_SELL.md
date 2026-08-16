# Feature Specification — Auto Sell (Mobile / LD9)

Status: **bag/item/NPC primitives STATIC-STRONG; exact sell request/shop state TARGETED-PROOF**.

## Bag state

Use semantic inventory methods:

```text
GetFreeBagSpace
GetItemsAtSite
GetItems
GetItemData
```

Do not open bag and visually count empty slots as the primary method.

## Live item model

Keep these concepts distinct:

```text
ID       live instance/database identity candidate
ItemID   template identity
Position current slot
Site     current container
Quantity current count
Bound    binding state
```

Exact runtime scanner must validate field meanings before write actions.

## Classification

Static helper names available:

```text
IsItemSellable
GetItemBasePrice
GetItemType
GetEquipType
IsItemThrowable
IsItemSellToShopWithBoundMoney
```

Apply stock semantic guards first, then user's keep/sell lists. Never classify only by icon color/OCR if type/item fields are available.

## Trigger

Example policy:

```text
if GetFreeBagSpace() <= configured threshold
    yield Train
    enter SELL_PREPARE
```

The threshold should be configurable; full bag is not the only possible trigger.

## NPC route

```text
configured vendor map/NPC identity
 -> GetNPCPosition
 -> GoTo/AutoPath
 -> map/proximity proof
 -> ClickNPC
 -> current dialog/shop readiness proof
```

Do not persist one pixel coordinate as the canonical vendor interaction.

## Exact sell action

Currently not mobile VERIFIED.

Required trace:

```text
current bag instance selected
 -> manually sell exactly one disposable item
 -> capture outbound command ID/payload/current shop IDs
 -> capture item removal/update proof
```

PC donor has a solved normal shop sell packet/payload. Use it to narrow mobile search, not as production mobile data.

## Safe inner loop

```text
fresh bag
 -> choose ONE candidate
 -> require current valid shop state
 -> ONE sell request
 -> wait for semantic item/shop update proof
 -> fresh bag
 -> repeat
```

Do not click 90 slots or cache 90 instance IDs.

## Stop condition

Stop when fresh bag scan has zero sell candidates or configured free-space target is reached.

## Return

Close/yield shop as needed, `GoTo(savedTrainMap,savedX,savedY)`, prove map/position/alive, then resume Train.

## Interruptions

Death overrides Sell. Map disconnect/loading invalidates current shop and live item object references.
