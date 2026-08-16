# Death, Revive and return-to-train — mobile exact flow

Status: **REQUEST + BUILT-IN COMEBACK FLOW VERIFIED STATIC; runtime acceptance/safety still to prove on LD9**

## Exact revive types

```text
C_RevivalType.Normal         = 1
C_RevivalType.NewbieRevival = 2
C_RevivalType.SkillRevival  = 3
```

## Exact request

Recovered `Revival.lua` sends:

```text
Network.SendPacket(G_TCPPacketDefine.CMD_REVIVE_DATA, String.Format("{0}", reviveType))
```

`CMD_REVIVE_DATA = 200063`.

Therefore normal/Đầu thai is exactly:

```text
packet = 200063
payload = "1"
```

Newbie and skill revive use `"2"` and `"3"`. This is independently mobile-verified, not inherited from PC.

## Revival server/UI state

Inbound `CMD_REVIVE_DATA` supplies `revivalData` and drives the `Revival` UI. Source uses:

```text
Action
TimeLeft
IsEnableReviveNewbie
IsEnableBySkill
```

`TCPCmdHandler` opens/updates/closes the Revival frame based on `Action`. On initial Open it also calls:

```text
AutoFight_Main:DeathActive()
```

This inbound generation is the best server-side transaction state for preventing duplicate revive actions.

## Stock Revival countdown

`Revival:DoCountDown()` initializes from `CurrentReviveData.TimeLeft`, decrements 1000 ms each second, and when the timer becomes negative calls `ButtonGoToInfernalClicked()`. That function sends the same normal request:

```text
200063:"1"
```

So a stock client may generate normal revive from either:

1. Revival countdown expiry;
2. AutoFight `AutoRevival`.

An external tool must avoid racing either stock producer with its own duplicate request. Track one pending normal revive per death/Revival generation.

## Built-in AutoFight death hook

`AutoFight_Main:DeathActive()` only runs its recovery path while current Auto mode is Train or PK.

If current MapID is not 87 it records:

```text
DieMapID = Game.RoleData.MapID
DieMapLocaltion = current position
```

### Stock AutoRevival timing

When `AutoRevival=true`, source does:

```text
wait 5 seconds
send 200063:"1"
wait 0.5 seconds
close Revival UI if still present
```

This fixed 5-second delay is stock policy, **not evidence that protocol 200063 requires five seconds**. External production logic should instead use fresh death/Revival state and concrete completion proof.

## Built-in AutoComeback

When `AutoComeback=true`:

```text
IsComeBackTrain = true
```

The death coroutine attempts `RiderUp()`. If mounting starts it waits five seconds and returns before directly issuing `GoTo`. The comeback state is not lost: the running Train loop sees `IsComeBackTrain` and periodically calls `ComeBackTrainData()`.

In the Train loop this happens when `TotalTick % 10 == 0` while comeback is active, so route correction is periodic rather than event-driven.

## `ComeBackTrainData()` exact donor behavior

### While already moving

If current MapID equals `DieMapID`, it computes distance to the saved death position. When:

```text
distance < 1000
```

it announces successful comeback and clears `IsComeBackTrain`.

That `<1000` threshold is a very loose stock tolerance for a tool that wants a precise configured train center. Treat it as donor behavior, not production truth.

### If currently Map 87

```text
Game.GoTo(2,-1,-1, callback)
 -> Game.GoTo(DieMapID,DieX,DieY, callback)
 -> IsComeBackTrain=false
```

### Otherwise

```text
Game.GoTo(DieMapID,DieX,DieY, callback)
 -> IsComeBackTrain=false
```

## Saved death spot vs configured train spot

Stock donor returns to **the death position**. A larger external auto should distinguish:

```text
DeathMap/DeathPosition = diagnostic / stock donor state
ConfiguredTrainMap/TrainPosition = desired external return target
```

If the character died while chasing far from the ideal center, returning to the exact death coordinate may be the wrong production behavior. The EXE should normally return to its configured train point.

## Recommended production proof

```text
DEAD generation
 -> ensure no normal revive already pending / countdown race
 -> send one 200063:"1" when policy allows
 -> fresh state says alive / Revival lifecycle cleared
 -> wait IsMapReady
 -> Game.GoTo(configured TrainMap,TrainX,TrainY)
 -> fresh MapID + valid position within configured tolerance
 -> resume Train=1
```

Do not treat fixed waits or stock `<1000` distance alone as completion proof.

## Failure / retry discipline

- one pending revive per death generation;
- bounded timeout + fresh state before retry;
- death during return creates a new death generation and invalidates the route action;
- map transition invalidates stale UI/world pointers;
- if stock AutoRevival/AutoComeback remains enabled, do not run an independent external copy without arbitration.
