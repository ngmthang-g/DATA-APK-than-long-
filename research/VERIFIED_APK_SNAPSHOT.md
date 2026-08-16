# VERIFIED APK Snapshot — ThanLongMobile_2024

This document records direct/reconfirmed evidence for the frozen APK only.

## Artifact

Status: **VERIFIED**

```text
ThanLongMobile_2024.apk
size    77,318,940 bytes
SHA256  9de719391c29d50816a3762f758abe26896cab4d996706711b30fdc10d9933f0
```

## Runtime architecture

Status: **VERIFIED**

APK contains:

```text
arm64-v8a/libil2cpp.so
arm64-v8a/libunity.so
arm64-v8a/libmain.so
assets/bin/Data/Managed/Metadata/global-metadata.dat
assets/bin/Data/ScriptingAssemblies.json
```

Metadata header version word is `0x27` = **39**.

`ScriptingAssemblies.json` explicitly contains `Assembly-CSharp.dll` and `Assembly-CSharp-firstpass.dll`.

## Standard IL2CPP export surface

Status: **VERIFIED from ELF dynamic exports**

Examples present in `libil2cpp.so`:

```text
il2cpp_domain_get
il2cpp_domain_get_assemblies
il2cpp_assembly_get_image
il2cpp_class_from_name
il2cpp_class_get_field_from_name
il2cpp_class_get_method_from_name
il2cpp_class_get_methods
il2cpp_class_get_fields
il2cpp_runtime_invoke
```

Implication: a guest-side resolver can prefer semantic class/method resolution over hardcoded mobile absolute addresses.

## High-value semantic names in metadata

Status: **VERIFIED name presence**

### Player/world

```text
LuaLeaderData
RoleData
CurrentHP
MaxHP
MapID
PosX
PosY
Position
IsDeath
IsMapReady
GetCurrentMoveDestination
GetLocalMapObjects
GetNearbyObjects
```

### Player/target/combat

```text
GetNearByPeacePlayers
GetNearbySpritesWithPredicate
SelectedTarget
GetNearByEnemies
SelectTarget
ChaseTarget
UseSkill
RequestUsingSkill
RequestUsingSkillWithTarget
RequestUsingSkillWithPos
GetSkillCooldown
GetBuffs
GetSkillLuaData
```

### Movement/path/NPC

```text
MoveTo
MoveToEx
GoTo
GetNPCPosition
ClickNPC
AutoPathManager
StartAutoPath
StopAutoPath
IsAutoPathing
SendAutoPathRequestChangeMap
```

### Inventory/item

```text
GetFreeBagSpace
GetItems
GetItemsAtSite
GetItemData
GetItemAtSite
GetTotalItems
CountItem
IsItemSellable
GetItemBasePrice
GetItemType
GetEquipType
IsItemThrowable
IsItemSellToShopWithBoundMoney
```

Related field/value names:

```text
ID
ItemID
Site
Position
Bound
Quantity
Creator
DurationLeft
Durability
Properties
Attributes
BasePrice
SellPrice
ItemSite
Bag
Body
Storage
Stall
Trade
Pet
```

### Network/revive

```text
LuaSystemManager
LuaSystemAPI_Game
LuaSystemAPI_Network
SendPacketToServer
SendPacket
TCPGame
TCPOutPacket
PacketCmdID
CMD_REVIVE
ProcessObjectDeath
ProcessObjectRevive
```

`CMD_REVIVE_DATA` exact PC naming was not found as an exact standalone mobile metadata string in this pass; therefore PC numeric revive facts are not imported as mobile VERIFIED.

## MainThread

Status: **RECONFIRMED**

`FGStudio.Engine.Utilities.MainThread`/`MainThread` naming is present. Prior metadata parse identified the familiar `Execute(Action)`, `Update`, `DoExecuteWorks` queue-dispatch pattern also documented in the PC KB.

Remaining boundary: live external/guest bridge callback and managed delegate lifetime proof on LD9.

## Asset crypto/update surface

Status: **VERIFIED name presence**

Metadata contains:

```text
AssetBundleUtils
LoadWithCrypto
LoadWithCryptoAsync
```

APK contains multiple `Interface*.unity3d` bundles. Native FG client library exports `FG_Decrypt` and `FG_Encrypt`.

This establishes a viable later route for Lua/UI semantic extraction if a required action cannot be recovered more narrowly from runtime/network producers.

## Emulator/security/native surface

Status: **VERIFIED exports**

`libFGClientTool_Android.so` exports:

```text
FG_EmuDetect
FG_GetEmuScore
FG_GetEmuReason
FG_IsAdbEnabled
FG_IsAdbReallyEnabled
FG_GetEnabledAccessibilityServices
FG_CanDrawOverlays
FG_GetSecurityReport
FG_Input_OnTap
FG_Input_GetMetrics
FG_Input_Reset
FG_Decrypt
FG_Encrypt
```

Metadata contains `SecurityReport` and `LoginData` names.

This proves observability surface, not enforcement/ban policy.

## Version/config

Status: **VERIFIED**

`assets/Version.xml` contains:

```text
Application VerCode="123"
CDN https://cdn.fgstudio.vn/
Android package com.fgstudio.thanlongmobile
```

## Explicit non-facts

The following are **not yet mobile VERIFIED**:

- mobile revive packet numeric ID/payload;
- mobile sell packet numeric ID/payload;
- exact NPC shop state object/payload requirements;
- exact high-level `AutoFight_Main:StartAutoFight` donor on mobile;
- safety of calling mutable APIs from an external guest bridge;
- whether security surface reacts to the proposed bridge/ADB usage.
