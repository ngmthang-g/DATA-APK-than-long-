# Auto Runtime Proof Queue — LDPlayer 9

Only collect proofs that unblock production automation. Do not restart broad APK reverse.

## P0 — per-instance read-only bridge

Goal: prove one LD9 guest bridge can repeatedly resolve and read state without destabilizing the game.

Capture:

```text
LD index
ADB serial
Android game PID
character name/RoleID if resolved
HP / MaxHP
MapID / Position
IsDeath
GetFreeBagSpace
snapshot timestamp/version
```

Pass condition: stable repeated reads while moving/map-changing, no cross-instance leakage.

## P0 — MainThread harmless callback

Goal: prove guest code can dispatch one harmless managed action through game-owned MainThread semantics.

Do not begin with sell/revive/combat.

Pass condition:

```text
valid Action/delegate lifetime
 -> MainThread dispatch
 -> callback executes on expected Unity/game thread
 -> no crash/diss
```

## P1 — revive producer/payload

Manual test:

1. character dies;
2. arm network trace at `SendPacketToServer` / `SendPacket` / `TCPGame` producer boundary;
3. press normal Đầu thai once;
4. record command ID and exact payload/argument encoding;
5. record pre/post `IsDeath`, HP, map-ready and position.

Promote only after repeatable capture.

PC donor search hint: PC KB has `200063`, but do not assume equality.

## P1 — shop open + sell producer/payload

Manual test:

1. scan bag and choose one disposable sellable item;
2. go to a known sell NPC manually or semantically;
3. trace dialog/shop opening state;
4. sell exactly one item;
5. record command ID/payload and current identifiers required by request;
6. verify the live item instance disappears/quantity changes and bag rescans consistently.

PC donor search hint: PC KB has `200036` with instance/shop identifiers; mobile requires confirmation.

## P1 — map transition / return-to-spot

Verify:

```text
saved TrainMap/TrainPosition
 -> GoTo/AutoPath
 -> portal/map transition
 -> IsMapReady
 -> fresh MapID/Position
 -> within tolerance
```

No fixed sleep is accepted as proof.

## P1 — multi-LD isolation

Run at least two LD9 sessions and deliberately put them in different states:

```text
A = training
B = dead/reviving
C = bag full/selling (if available)
```

Pass condition: no pointer/profile/action/shop/target state crosses sessions.

## P2 — high-level Auto Train donor

Static metadata already exposes low-level target/chase/skill primitives. Research high-level built-in auto only if using it materially reduces action complexity and runtime risk.

PC donor: `AutoFight_Main:StartAutoFight(C_AutoModel.Train)` exists on PC, but exact mobile source/path remains unverified.

## P2 — asset/Lua extraction

Only attempt encrypted Interface/Lua bundle recovery when a concrete missing action cannot be solved via metadata + runtime producer trace.
