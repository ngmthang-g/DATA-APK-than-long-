# Auto Runtime Proof Queue — LDPlayer 9

Static reverse has solved the major Train/Revive/Sell/Chat action producers. Runtime work should now prove safety and state transitions, not rediscover packet IDs.

## P0 — per-instance read-only bridge

For each LD9 capture repeatedly:

```text
LD index / ADB serial / game PID
RoleID / RoleName
HP / MaxHP
MapID / Position
IsDeath / IsMapReady
FreeBagSpace
snapshot version / world generation
```

Pass: stable during movement/map change and no cross-instance leakage.

## P0 — MainThread harmless callback

Prove one valid managed Action/delegate can be dispatched through `MainThread.Execute` with correct lifetime/thread and no crash/diss. Do not begin with sell/revive/combat.

## P1 — built-in Train live proof

Invoke semantic Train start on one disposable test session:

```text
StartAutoFight(Train)
 -> target selected
 -> chase/skill occurs
 -> snapshot remains stable
 -> StartAutoFight(None) stops/yields cleanly
```

No competing external combat loop during this proof.

## P1 — normal revive acceptance

Static request is already solved:

```text
200063:"1"
```

Runtime proof:

```text
fresh dead state -> one request -> alive/HP/revival lifecycle -> map-ready
```

Also compare stock `AutoComeback` with external explicit return policy.

## P1 — one verified mobile sell vendor

Do not rediscover `200036`. Instead prove:

```text
GoToNPC(candidate)
 -> actual GameDialog/Shop
 -> inbound current shop data 200034
 -> IsGuildShop=false
 -> one disposable sellable item
 -> send 200036 with current instance/shop IDs
 -> RemoveItem/UpdateItemsList/fresh bag proof
```

Lâu Lan PC-donor/user candidates: Ba Nhĩ 328, Mã Kiêu Minh 373; confirm on mobile.

## P1 — treatment

For chosen healer:

```text
GoToNPC
 -> capture GameDialog.Selections
 -> record actual visible treatment text and selectionID
 -> send 100007 actualSelectionID:-1
 -> follow any real confirmation
 -> prove HP/result
```

Do not make selectionID a permanent constant until repeated evidence supports it.

## P1 — chat/ping

Prove desired channels using `100008`. Validate normal message and `@GOTO_MapID_GridX_GridY` location ping. Record server cooldown/permission responses; do not bypass them.

## P1 — map transition and return spot

Use state proof:

```text
saved TrainMap/TrainPos
 -> GoTo
 -> map transition
 -> IsMapReady
 -> fresh MapID/Position within tolerance
 -> resume Train
```

## P1 — multi-LD isolation

At least two instances in deliberately different states. Pass only if profiles, snapshots, targets, dialog/shop state and actions never cross sessions.

## P2 — only remaining static reverse

Decrypt/analyze another asset/config bundle only when a concrete feature requires missing static data (for example a mobile-only NPC service table). Broad asset dumping is no longer justified for the current core tool.
