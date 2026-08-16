# Mobile Auto — State → Guard → One Action → Proof Matrix

Status: **CANONICAL EXTERNAL ORCHESTRATION CONTRACT**

Purpose: turn the recovered client semantics into an implementation-ready contract for one independent LDPlayer BotSession.

The matrix is deliberately stricter than the shipped Lua. Static source tells us **what semantic action exists**; production automation still requires fresh-state guards and concrete result proof.

---

# 1. Global invariants

Every mutable transition follows:

```text
fresh immutable snapshot
 -> check session/generation
 -> check feature guard
 -> acquire one ActionGate owner
 -> issue ONE semantic mutable action
 -> wait for concrete proof / timeout / failure event
 -> release or continue same transaction owner
 -> fresh snapshot
```

Never execute a host decision when its required:

```text
AndroidGamePID
ResolverGeneration
WorldGeneration
DialogGeneration
ShopGeneration
BagVersion
```

no longer matches current state.

Read-only observers may run concurrently. Gameplay mutations inside one BotSession may not compete.

Suggested owner priority:

```text
MANUAL_PAUSE / SAFETY
 > REVIVE
 > MAP_TRANSITION_COMPLETION
 > CRITICAL_RECOVERY
 > HEAL_NPC
 > SELL_TRANSACTION
 > CHAT_REQUEST
 > TRAIN / PICKUP
```

---

# 2. IDLE → TRAINING

### State

```text
alive
map ready
not in fragile dialog/shop transaction
Train.Enabled
```

### Guards

```text
CurrentAutoMode is not incompatible Quest mode
no current higher-priority owner
current map/position acceptable for starting this profile
bridge/MainThread healthy
```

### One action

```text
AutoFight_Main:StartAutoFight(C_AutoModel.Train)
C_AutoModel.Train = 1
```

### Proof

Use fresh semantic state such as:

```text
stock Auto mode becomes Train when observable
and/or target/Train coroutine behavior begins
no immediate client error/refusal
```

### Failure

If stock Quest mode refuses the transition, do not spam Train. Surface current incompatible mode and require the orchestrator/user policy to stop/yield it first.

---

# 3. TRAINING → TRAIN_YIELD

Used before Sell/Heal/explicit external transition.

### Guard

```text
Train currently owns normal combat/pickup
higher-priority transaction needs character control
```

### One action

```text
AutoFight_Main:StartAutoFight(C_AutoModel.None)
```

### Proof

```text
no new stock Train mutation
current target/path state settles or is explicitly superseded
```

Do not use delayed `StopAllCurrentTask()` UI wrapper as the canonical stop primitive.

---

# 4. TRAINING → RECOVERY

### Trigger

```text
HPPercent <= configured HP threshold
or
MPPercent <= configured MP threshold
```

### Guards

```text
ExternalOwnsItemUse=true
alive
not in map loading
configured medicine template present in fresh Bag
Game.IsHPMedicine / IsMPMedicine matches intended type
current live DBItemData.ID resolved
no current item mutation pending
```

### One action

```text
CMD_ITEM_ACTION = 100005
Use = 3
payload = 3:itemInstanceID
```

### Proof

Any consistent combination:

```text
HP/MP increases
medicine quantity/instance changes
RemoveItem / UpdateItemsList event when stack disappears
known item-use/cooldown/action state changes
```

Then **rescan before another Use**.

### Failure

Do not repeat immediately merely because HP remains low. Timeout → fresh bag/vital rescan → decide again.

Configured recovery medicine is automatically RESERVED from Sell/Drop/Destroy classification.

---

# 5. TRAINING → SELL_ROUTE

### Trigger

Canonical semantic trigger:

```text
Game.GetFreeBagSpace() <= Sell.TriggerFreeSlots
```

not a periodic bag-window screen check.

### Guards

```text
Sell.Enabled
alive
saved TrainMap/TrainPosition valid
no current Shop transaction from an older generation
```

### Transition actions

First yield Train/pickup ownership, then route.

For cross-map vendor:

```text
Game.GoTo(vendorMap,-1,-1,callback)
```

then after current map proof:

```text
npcPos = Game.GetNPCPosition(vendorNpcID,...)
Game.GoTo(vendorMap,npcPos.X,npcPos.Y,callback)
Game.ClickNPC(vendorNpcID)
```

These are sequential state-machine actions, never one blind burst.

### Route proof

```text
fresh IsMapReady
fresh MapID == vendorMap
fresh position/proximity state
then inbound current GameDialog or NPCShop generation
```

### Failure

Death → hand to REVIVE.
No NPC/current route → abort Sell transaction and fresh re-plan; do not click hardcoded pixels.

---

# 6. SELL_ROUTE → SHOP_READY

Some vendors may open a GameDialog before NPCShop.

### GameDialog guard

Use the **copied current inbound dialog snapshot**, not a stale UI pointer.

```text
DialogGeneration current
Selections[] present
wanted shop/service text matches one live selection
```

### One action

```text
CMD_SHOW_GAMEDIALOG = 100007
payload = actualSelectionID:-1
```

### Proof

```text
new dialog generation
or
inbound CMD_NPC_SHOP_DATA = 200034
```

### Shop-ready guard

```text
ShopGeneration current
shopData exists
IsGuildShop == false
NpcShopID current
ShopID current
```

Do not persist those transaction IDs as profile constants.

---

# 7. SHOP_READY → SELL_ONE → SHOP_READY

### Candidate guards

Candidate is selected from a **fresh Bag snapshot** using `database/AUTO_SELL_CLASSIFICATION.md`:

```text
Site == Bag(10)
current instance still exists
quest range protected
Game.IsItemSellable(ItemID) == true
not recovery medicine/reserved
explicit Sell policy matches
```

### One action

```text
CMD_NPC_SHOP_SELL_REQUEST = 200036
payload = itemInstanceID:NpcShopID:ShopID
```

where:

```text
itemInstanceID = DBItemData.ID
```

### Proof

Prefer:

```text
RemoveItem(site:dbID:position) for expected live instance
and/or UpdateItemsList
then fresh Bag rescan proves expected quantity/instance change
```

Money/shop state may be supplementary proof.

### Continue/stop

Continue only after fresh candidate selection.

Stop Sell when:

```text
no Sell candidate remains
or
FreeBagSpace >= configured ReturnFreeSlotsTarget
```

Never precompute and fire a 90-item stale list.

---

# 8. SHOP_READY → RETURN_TRAIN_SPOT

### Guards

```text
Sell transaction complete/aborted cleanly
alive
saved TrainMap/TrainPosition valid
```

### One route step

```text
Game.GoTo(savedMap,savedX,savedY,callback)
```

For multi-map routes, let client AutoPath/PathFinder own portal selection.

### Proof

```text
IsMapReady == true
MapID == savedMap
fresh Position within ReturnTolerance
```

Then restore pickup policy and transition to TRAIN start.

A fixed sleep is not proof.

---

# 9. ANY_ALIVE_STATE → DEAD / REVIVE

### Detection

Canonical state combines fresh local death truth with server lifecycle:

```text
LuaLeaderData.IsDeath / GRole.IsDeath / HP
Revival generation when received
```

### Guard

```text
dead
Revive.Enabled
no normal revive action already pending for this death generation
```

### One action

Normal Đầu thai:

```text
CMD_REVIVE_DATA = 200063
payload = "1"
```

### Proof

```text
fresh alive state / HP restored
Revival lifecycle changes/closes
map becomes ready
valid fresh position
```

Do not call `ProcessObjectRevive` or any response handler as the request.

### Failure

Timeout does not justify packet spam. Refresh death/Revival state and retry according to bounded profile policy.

---

# 10. REVIVE → RETURN_TRAIN_SPOT

### Guard

```text
alive
map ready or transition-ready
saved TrainMap/TrainPosition valid
```

### Action

Use semantic `Game.GoTo`.

Stock donor has special Map 87 handling and `AutoComeback`; the external tool may initially reuse the stock comeback behavior or own the route itself, but **only one return engine** should be active.

### Proof

```text
expected MapID
IsMapReady
position within tolerance
```

Then resume Train.

---

# 11. TRAINING / IDLE → HEAL_ROUTE

### Trigger

Tool policy, e.g. HP threshold or manual command, distinct from simple consumable Recovery.

### Guards

```text
HealNPC.Enabled
alive
saved return spot if Train must resume
configured healer MapID/NpcID
```

### Route

Use the same semantic NPC path:

```text
map-only GoTo if needed
GetNPCPosition
GoTo live NPC position
ClickNPC
```

### Proof

Wait for copied current `GameDialog`/service state.

---

# 12. HEAL_ROUTE → HEAL_DIALOG_ACTION

### Guard

```text
current DialogGeneration
Selections[] copied from inbound 100007 data
one configured text matcher uniquely matches current visible service text
```

### One action

```text
CMD_SHOW_GAMEDIALOG = 100007
payload = actualSelectionID:-1
```

### Proof

Depending on current server flow:

```text
new dialog generation / confirmation step
HP increases/restores
money changes when applicable
dialog closes/completes
```

If a confirmation dialog appears, it is another dynamic state/action cycle.

### Important unresolved runtime boundary

The generic mechanism is statically solved. Exact current mobile healer identity and server-provided treatment text/confirmation sequence remain runtime proof. PC NPC 339 is a donor candidate, not a mobile static fact.

---

# 13. ANY_STABLE_STATE → SEND_CHAT

### Guards

```text
Chat.Enabled / user or scheduler request
actual channel ID resolved (not UI dropdown index)
text <= practical client limit 200 chars
private channel -> live target RoleID/name provided
per-session conservative rate limiter allows send
not inside fragile map/shop/dialog mutation unless explicitly prioritized
```

### One action

```text
CMD_CLIENT_CHAT = 100008
{ RoleID, Name, Content=Base64(text), Channel=actualChannelID }
```

### Proof

For development/telemetry use available chat/server response/event state. Do **not** require an echo as the only success proof because server/channel behavior may differ.

Server rate limits/permissions are runtime outcomes; do not bypass them.

---

# 14. SEND_LOCATION_PING

### Guards

Same as chat plus fresh current location.

### Build payload text

```text
current Grid = Game.WorldToGridPosition(RoleData.Position)
raw token = @GOTO_<MapID>_<GridX>_<GridY>
```

Append only if total content remains within the practical 200-character client envelope.

### One action

Send ordinary `100008` chat with the token in Base64 content.

Receiver stock logic converts the display link back through `GridToWorldPosition` and `Game.GoTo`.

---

# 15. ITEM_ABANDON

This is separate from Sell.

### Guards

```text
fresh Bag instance
explicit profile Abandon rule
Game.IsItemThrowable(ItemID) == true
not RESERVED/recovery/current transaction item
```

### One action

```text
100005
payload = 4:itemInstanceID
```

### Proof

Expected RemoveItem/UpdateItemsList + fresh bag difference.

---

# 16. ITEM_DESTROY

### Default state

```text
DISABLED
```

### Guards when explicitly enabled

```text
fresh exact candidate list
narrow explicit Destroy rule
not recovery/reserved/unknown/equipment unless rule explicitly covers it
candidate generations still current
```

### One action

```text
100005
payload = 9:id1;id2;...
```

### Proof

Every expected live instance is removed in fresh state.

Because stock UI treats this as irreversible, production defaults must remain conservative.

---

# 17. Captcha / safety

### Detection

```text
G_TCPEventType.NewCaptcha = 57
```

### Action

External orchestrator:

```text
pause mutable automation
surface manual intervention/status
```

No automated captcha solving/bypass belongs in this architecture.

---

# 18. Reconnect / process restart

### Detection

```text
Android PID changed
bridge unhealthy/disconnected
login/world recreation observed
```

### Mandatory action

No gameplay packet is sent.

Instead:

```text
invalidate resolver/script/UI/item/shop/dialog pointers
increment ResolverGeneration / WorldGeneration
reacquire package PID inside the same ADB serial
rebuild read-only snapshot
only then resume orchestration
```

---

# 19. Multi-LD rule

The matrix runs **once per BotSession**.

Example simultaneous state is valid:

```text
LD0 = TRAINING
LD1 = REVIVE
LD2 = SELL_ONE
LD3 = HEAL_DIALOG_ACTION
```

Their action gates, generations, bag instances, shops, targets and profiles are independent. A global `Start selected` button dispatches separate state transitions; it does not merge their state machines.

---

# 20. Implementation order

Recommended first production proofs:

```text
1 read-only snapshot/generation stability
2 harmless MainThread callback
3 Lua/UI lookup bridge
4 Train start/stop
5 map route/return proof
6 Revive
7 bag trigger + one Sell
8 Recovery item Use
9 dynamic GameDialog treatment
10 Chat / @GOTO
11 multi-LD soak
```

This order minimizes the number of simultaneous unknowns when diagnosing diss/crash/state desynchronization.
