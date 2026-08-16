# LD9 action orchestration for Train / Revive / Sell / Heal / Chat

Status: **DESIGN BASED ON VERIFIED CLIENT ACTION SURFACE**

## Ownership split

```text
Windows EXE
 -> LD instance discovery/profile/UI/logging/orchestration
 -> one BotSession per LD9
 -> guest command channel
 -> ARM64 guest bridge
 -> IL2CPP resolver + Lua/Game semantic action adapter
```

The Windows process does not directly execute ARM64 IL2CPP code.

## Per-session mutable action gate

Only one mutation should own a character at a time. Suggested priority:

```text
manual pause / safety
 > death + revive
 > map-transition completion
 > treatment / survival
 > active shop/sell transaction
 > chat command if urgent
 > Train
 > optional background optimization
```

Read-only snapshots may run concurrently. Mutable gameplay actions may not compete.

## Feature transitions

### Normal Train

```text
TRAIN_ENTER -> StartAutoFight(Train) -> TRAINING
```

### Death recovery

```text
DEAD -> stop/yield conflicting actions -> revive 200063:"1"
 -> alive/map-ready -> return saved Train spot -> resume Train
```

### Bag full

```text
TRAINING -> stop Train -> GO_VENDOR -> OPEN_SHOP
 -> SELL_ONE -> PROVE -> RESCAN loop
 -> RETURN_SPOT -> resume Train
```

### Treatment

```text
need-heal -> stop Train -> GO_HEALER -> DIALOG_MATCH
 -> send actual selection -> prove HP/result -> RETURN_SPOT -> resume Train
```

### Chat / ping

Chat is a short semantic network action. It should not steal Windows focus or keyboard. Rate-limit it and avoid inserting it during a fragile map/shop transition unless required.

## Fresh-world rule

After map change, death/revive, reconnect, shop/dialog open/close, invalidate guest object references and reacquire semantic roots. A `WorldGeneration` counter should prevent stale actions from a previous scene generation.
