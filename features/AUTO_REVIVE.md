# Feature Specification — Auto Revive / Đầu thai

Status: **death/revive state surface STATIC-STRONG; exact outbound revive request TARGETED-PROOF**.

## Static evidence

Mobile metadata contains:

```text
IsDeath
ProcessObjectDeath
ProcessObjectRevive
CMD_REVIVE
```

Player/world state also exposes HP/map/position/map-ready concepts.

## State machine

```text
TRAINING
 -> death observed
 -> REVIVE_WAIT_READY
 -> REVIVE_REQUEST (one action)
 -> REVIVE_PROOF
 -> RETURN_TO_SPOT
 -> TRAIN_RESUME
```

## Death proof

Prefer semantic death state (`IsDeath` / HP + role state) rather than looking for a death dialog image.

## Exact request boundary

Do not send a guessed packet.

Required proof:

```text
manual normal revive once
 -> capture outbound producer at SendPacketToServer / SendPacket / TCPGame
 -> exact command ID + payload/type
 -> repeat capture
```

PC donor: PC KB has `CMD_REVIVE_DATA=200063` and revive-type semantics. This is a search hint only until mobile confirms it.

## Completion proof

After one revive action, require a fresh snapshot showing a consistent alive state, e.g.:

```text
IsDeath == false
HP > 0
IsMapReady == true
valid current MapID/Position
```

Then begin return path. Do not equate packet send success with character revival.

## Return-to-spot

Use saved per-session train map/position and semantic `GoTo`/auto-path APIs. Resume Train only after the expected map is ready and position is within tolerance.

## Retry policy

Retry only after timeout/fresh state proves no completion. Do not spam revive commands while a transition is already in progress.

## Multi-LD rule

A death/revive state is local to exactly one BotSession. It must not pause or mutate other LD9 sessions unless the user selected a global pause.
