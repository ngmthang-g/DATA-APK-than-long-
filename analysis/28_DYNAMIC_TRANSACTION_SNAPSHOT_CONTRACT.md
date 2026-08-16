# Dynamic transaction snapshot contract — GameDialog / NPCShop / Revival

Status: **FIELD USAGE VERIFIED FROM RECOVERED MOBILE LUA; live observer wiring remains runtime proof**

Purpose: define exactly what the guest bridge should copy from transient inbound Lua server objects so the Windows host can make decisions without holding Lua/UI object pointers.

---

# 1. General transaction envelope

For every copied transient server object publish:

```text
TransactionType
Generation
ReceivedAt
WorldGeneration
SourcePacketID
IsOpen / IsClosed
```

Generation increments whenever a new inbound object replaces, refreshes or closes the previous transaction.

Never expose the original Lua/C# managed object address to reusable Windows-host state.

---

# 2. GameDialog snapshot

Source packet:

```text
CMD_SHOW_GAMEDIALOG = 100007
```

`TCPCmdHandler` destroys any old GameDialog first. If inbound data is literal `"NULL"`, the dialog is closed and no new UI is created.

Recovered `GameDialog.lua` consumes:

```text
Title
Message                 # Base64 string from server; UI decodes with String.FromBase64
Selections              # map selectionID -> visible selectionName
Awards
```

Awards are optional and can contain at least:

```text
ItemSelectable
EnableSelectItem
Items[]
Exp
# additional money/currency reward fields used by the UI
```

For treatment/shop-service automation the minimum copied structure is:

```text
DialogGeneration
Open
Title
MessageBase64
MessageDecoded
Selections[] {
    SelectionID
    VisibleText
}
HasAwards
ReceivedAt
```

Optional quest automation can additionally copy Award items/selection requirements.

### Close signal

```text
inbound data == "NULL"
```

should create a closed generation or invalidate current dialog immediately.

### Action dependency

A `selectionID` is valid only against the exact current DialogGeneration from which it was copied.

---

# 3. NPCShop snapshot

Source packet:

```text
CMD_NPC_SHOP_DATA = 200034
```

`TCPCmdHandler` either creates `NPCShop` or calls `RefreshData(shopData)`.

Recovered shop UI consumes these fields directly:

```text
shopData.CategoryName
shopData.IsGuildShop
shopData.ID
shopData.NpcShopID
shopData.Items
shopData.TotalSellItem
```

### Meaning in source

```text
CategoryName -> NPCShop frame title
IsGuildShop  -> disables Sell tab when true
ID           -> current ShopID used by buy/sell requests
NpcShopID    -> current NPC shop identity used by requests
Items        -> items for sale by NPC
TotalSellItem-> recently sold/buy-back live item list displayed on Sell tab
```

Minimum Auto Sell copy:

```text
ShopGeneration
Open
CategoryName
IsGuildShop
ShopID = shopData.ID
NpcShopID = shopData.NpcShopID
TotalSellItemCount
ReceivedAt
```

There is no need to copy the entire shop `Items` catalog for basic Auto Sell unless the feature also buys items.

### Exact sell dependency

```text
200036
payload = liveItemInstanceID : NpcShopID : ShopID
```

`NpcShopID` and `ShopID` must come from the current ShopGeneration. Never store them as persistent vendor-profile constants.

### Buy-back observation

The Sell tab displays up to 10 entries from `TotalSellItem`. Stock buy-back request is:

```text
CMD_NPC_SHOP_BUY_REQUEST = 200035
payload = dbItemData.ID:NpcShopID:ShopID:dbItemData.Quantity:1
```

This is useful as diagnostic evidence that `TotalSellItem` contains the live sold-item records, but Auto Sell does not need to invoke buy-back.

---

# 4. Revival snapshot

Source packet:

```text
CMD_REVIVE_DATA = 200063
```

Recovered UI/handler consumes:

```text
Action
TimeLeft
IsEnableReviveNewbie
IsEnableBySkill
```

Recommended copy:

```text
RevivalGeneration
Open
Action
TimeLeftMs
IsEnableReviveNewbie
IsEnableBySkill
ReceivedAt
NormalRevivePendingByTool
```

The last field is tool-owned state, not server data. It prevents duplicate `200063:"1"` actions inside one death generation.

### Stock countdown interaction

The stock Revival UI can automatically send normal revive when `TimeLeft` expires. Therefore external logic must treat countdown state as part of its duplicate-action arbitration.

---

# 5. Bag mutation event snapshot

Central event layer provides:

```text
RemoveItem = 2 -> site:dbID:position
UpdateItemsList = 3
AddItem = 1
```

Recommended event record:

```text
BagEventGeneration
EventType
Site
DBID
Position
ReceivedAt
```

After any mutation event:

```text
increment BagVersion
mark cached BagItems stale
fresh GetItemsAtSite(Bag) before next destructive item action
```

---

# 6. Chat / safety events

Relevant central events:

```text
ChatEvent = 50
NewCaptcha = 57
```

Chat event may be copied for telemetry but is not the only possible proof of outbound send acceptance.

Captcha event must immediately elevate Safety owner and pause mutable automation. No automated solving/bypass.

---

# 7. Host-facing JSON-like shape

Illustrative serialized snapshot:

```text
{
  worldGeneration: 42,
  dialog: {
    generation: 17,
    open: true,
    title: "...",
    message: "...",
    selections: [
      { id: 5, text: "..." },
      { id: 9, text: "..." }
    ]
  },
  shop: {
    generation: 8,
    open: true,
    categoryName: "...",
    isGuildShop: false,
    shopId: 123,
    npcShopId: 456
  },
  revival: {
    generation: 4,
    open: false
  },
  bagVersion: 91
}
```

Exact wire format between guest and Windows host is an implementation choice. The invariant is semantic-copy + generation, not raw pointers.

---

# 8. Action-gate stale-transaction rejection

An action request from the Windows host should declare the generation it was decided against:

```text
SellOne {
    requiredShopGeneration,
    requiredBagVersion,
    itemInstanceID
}

SelectDialog {
    requiredDialogGeneration,
    selectionID
}
```

The guest must reject the request without mutation if the generation/version no longer matches current state.

This prevents a classic race:

```text
host reads selection/shop/item
 -> server refreshes/closes transaction
 -> stale host command arrives later
 -> wrong current transaction is mutated
```

For a multi-LD auto this generation check is one of the most important crash/wrong-action protections.
