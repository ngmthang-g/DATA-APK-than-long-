# Runtime Snapshot Schema — Host-facing per LD9

The host should consume immutable semantic snapshots, not guest raw pointers.

```text
BotSnapshot
  SessionId
  LdIndex
  AdbSerial
  GamePid
  ProcessGeneration
  SnapshotVersion
  CapturedAt

  Character
    RoleId?
    Name?
    CurrentHP
    MaxHP
    IsDeath

  World
    MapID
    PosX
    PosY
    IsMapReady
    IsAutoPathing?
    MoveDestination?

  Target
    SelectedRoleId?
    SelectedType?
    SelectedAlive?

  Bag
    FreeSpace
    ItemCount?
    Items[] only when requested/needed
      InstanceID?
      ItemID
      Site
      Position
      Quantity
      Bound
      Sellable?
      Type?
      EquipType?

  Feature
    State
    PendingActionId?
    LastAction
    LastProof
    LastError
```

Question marks indicate fields whose exact mobile runtime acquisition/schema still requires proof.

## Snapshot invariants

- immutable after publish;
- belongs to exactly one ProcessGeneration;
- never carries raw Il2CppObject pointers to Windows host;
- stale snapshot cannot authorize a new item/shop mutation after map/process generation changes.
