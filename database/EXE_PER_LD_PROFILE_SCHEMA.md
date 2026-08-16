# Tool-owned per-LD profile schema — recommended

Status: **DESIGN CONTRACT**

Purpose: keep external automation state/config separate from the game's legacy `AutoSettings=3.5` string. Each LDPlayer instance owns one profile and one runtime session.

## Identity / binding

```text
ProfileId
LDIndex
ADBSerial
ExpectedPackage = com.fgstudio.thanlongmobile
ExpectedRoleID? / RoleName?
Enabled
```

Do not bind permanently to a transient Android PID; reacquire it after restart/reconnect.

## Train

```text
Train.Enabled
Train.MapID
Train.X
Train.Y
Train.PositionTolerance
Train.Radius
Train.UseBuiltInTrain = true
Train.ResumeAfterRecovery = true
Train.ResumeAfterSell = true
Train.ResumeAfterHeal = true
```

Optional donor-import fields:

```text
AttackMonsterList
AutoTrainSkillList
UsingCombo
UsingF1Key
GiveUpMonsterOutRanger
```

## Pickup / bag threshold

```text
Pickup.Enabled
Pickup.RestoreAfterSell
Sell.TriggerFreeSlots        # transition when FreeBagSpace <= N
Sell.ReturnFreeSlotsTarget   # stop sell when FreeBagSpace >= target, if candidates remain optional
```

Avoid wall-clock `check bag every X minutes` as the canonical trigger; free-space is already semantic state.

## Recovery

```text
Recovery.ExternalOwnsItemUse
Recovery.HP.Enabled
Recovery.HP.Percent
Recovery.HP.TemplateItemID
Recovery.MP.Enabled
Recovery.MP.Percent
Recovery.MP.TemplateItemID
Recovery.ActionTimeoutMs
```

If `ExternalOwnsItemUse=true`, disable/arbitrate the stock recovery loop so two engines do not use the same medicine concurrently.

Configured medicine becomes an automatic Keep rule.

## Revive / return

```text
Revive.Enabled
Revive.Type = Normal
Revive.ActionTimeoutMs
ReturnToTrain.Enabled
ReturnToTrain.MapReadyTimeoutMs
ReturnToTrain.PositionTolerance
```

Normal mobile request is statically solved as `200063:"1"`; the profile should not store/redefine packet numbers.

## Sell

```text
Sell.Enabled
Sell.Vendor.MapID
Sell.Vendor.NpcID
Sell.Vendor.ExpectedName?     # display/proof only
Sell.OpenTimeoutMs
Sell.ItemActionTimeoutMs
Sell.KeepRules[]
Sell.SellRules[]
Sell.NeverSellTemplateIDs[]
Sell.NeverSellInstanceIDs[]   # runtime/temporary only; do not persist stale instances
Sell.ProtectQuestRange = true
Sell.RequireIsItemSellable = true
```

Do not persist current `NpcShopID`, `ShopID`, item instance IDs or UI selection IDs as stable config. Acquire them from the **current transaction**.

## Drop / Destroy / Move

```text
ItemPolicy.Abandon.Enabled = false
ItemPolicy.Abandon.Rules[]
ItemPolicy.Destroy.Enabled = false
ItemPolicy.Destroy.Rules[]
ItemPolicy.Move.Enabled = false
ItemPolicy.Move.DestinationSite?
```

Abandon preserves `IsItemThrowable`. Destroy remains opt-in and should display/log the exact fresh candidate list before execution in development builds.

## NPC treatment

```text
HealNPC.Enabled
HealNPC.TriggerHPPercent?
HealNPC.MapID
HealNPC.NpcID
HealNPC.ExpectedName?
HealNPC.SelectionTextMatchers[]
HealNPC.ActionTimeoutMs
```

Do not store one global GameDialog selection ID. Resolve the current live `Selections` each time.

## Chat / Ping

```text
Chat.Enabled
Chat.Channel
Chat.PrivateRoleID?
Chat.PrivateName?
Chat.MinIntervalMs
Chat.MaxLength = 200
Chat.Templates[]
Chat.Schedule[]
Chat.EventRules[]
Chat.PingLocation.Enabled
```

Event examples can include `onDeath`, `onReturnSpot`, `onSellStart`, `onSellComplete`, `onSpotSwitch`, but the user/tool decides which are enabled. Chat should be rate-limited independently per LD session.

## Safety / runtime ownership

```text
Safety.PauseOnCaptcha = true
Safety.MaxConsecutiveActionFailures
Safety.MaxMapTransitionFailures
Safety.ActionTimeoutDefaults
Safety.StopOnBridgeUnhealthy = true
```

No captcha/security bypass fields belong in this schema.

## Runtime-only session state — never persist as reusable pointers

```text
AndroidGamePID
ResolverGeneration
WorldGeneration
SnapshotVersion
CurrentTargetRoleID
CurrentDialog object/reference
CurrentShop object/reference
CurrentShopID / NpcShopID
CurrentBagItemInstanceIDs
LastActionId / LastProof
```

These values are invalidated on reconnect/map/world generation changes as appropriate.

## Profile independence

Two LD9 profiles may point to the same train map/NPC, but they never share mutable session state. Global UI operations (`Start selected`, `Pause selected`) dispatch independent commands to each `BotSession` rather than creating one shared state machine.
