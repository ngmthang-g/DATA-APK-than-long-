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

## Built-in death hook

`AutoFight_Main:DeathActive()` for Train/PK:

1. records death map/position when not already in infernal map;
2. if `AutoRevival=true`, sends normal revive and closes Revival UI if present;
3. if `AutoComeback=true`, sets `IsComeBackTrain=true` and invokes route-back logic.

The stock source uses fixed waits internally. An external orchestrator should prefer state proof over copying those waits.

## Death location and comeback

The donor saves `DieMapID` and `DieMapLocaltion`. Map `87` is specially treated as infernal/death map.

`ComeBackTrainData()` uses current movement/map/distance state. If in Map 87 it routes via `Game.GoTo(2,-1,-1)` and then back to the saved death map/position; otherwise it directly `Game.GoTo(DieMapID,DieX,DieY)`. Arrival clears `IsComeBackTrain`.

## Recommended production proof

```text
DEAD
 -> send one 200063:"1"
 -> fresh state says alive / Revival lifecycle cleared
 -> wait map ready
 -> GoTo saved TrainMap/TrainPos
 -> fresh MapID + valid position within tolerance
 -> resume Train=1
```

Do not treat a fixed sleep as completion proof.
