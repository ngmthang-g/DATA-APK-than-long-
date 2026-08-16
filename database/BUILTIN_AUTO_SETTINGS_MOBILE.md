# Built-in AutoSettings schema — mobile APK snapshot

Status: **VERIFIED from recovered mobile `AutoFight_Main.lua`**

## Version

```text
AUTOVERSION_DEFINE = "3.5"
```

This differs from the currently analyzed PC frozen client (`4.1`). The two platforms share substantial structure but are not the same Auto revision. Do not copy the full PC serialized settings payload into this mobile client.

## Persistence

Mobile source builds:

```text
version#AUTOTRAIN#PICKITEM#UTILITIES#REGENE#PET#AUTOPK#FUBEN
```

Then sends:

```text
CMD_SHARED_PARAMETER = 200024
C_SharedParameterType.Set = 1
payload = 1:AutoSettings:<AUTOSETINGDATA>
```

and assigns the same string to:

```text
Game.RoleData.AutoSettings
```

## AUTOTRAIN — 9 fields

```text
1 IsAttackMonsterInList
2 AttackMonsterList
3 IsLureModel
4 IsTrainInRanger
5 RangerScan
6 AutoTrainSkillList
7 UsingCombo
8 UsingF1Key
9 GiveUpMonsterOutRanger
```

Defaults include `IsTrainInRanger=true`, `RangerScan=500`, seven `-1` skill slots and `UsingF1Key=true`.

## PICKITEM — saved as 9 fields

```text
1 IsOn
2 PickRanger
3 IsFilterItem
4 FilterItemSettings
5 AutoEatX2
6 AutoUsingItem
7 UsingItemList
8 IsAutoDropItem
9 DropItemSettings
```

Default `PickRanger=500`, `FilterItemSettings="0_1_3_0_1"`, other automation toggles off.

### Verified loader defect

Mobile `LoadAutoConfig()` reads both:

```text
IsAutoDropItem   <- field 8
DropItemSettings <- field 8
```

although Save writes the drop-list to field 9. See `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md`.

## UTILITIES — 14 fields

```text
1  AutoAcceptInviteTeam
2  AutoRejectInviteTeam
3  AutoRejectTrade
4  AutoLevelUp
5  LevelUpSet
6  IsAutoBuff
7  AutoBuffSkillList
8  ChatSelect
9  ChatCostumeChannel
10 ChatSelectSend
11 AutoRejectEmoji
12 AutoRejectMount
13 RejectJoinGuild
14 RejectJoinAllies
```

The three Chat fields are persisted channel/UI preferences; recovered source does **not** contain a stock periodic chat sender.

## REGENE — 14 fields

```text
1  AutoRegenHP
2  AutoRegenHPPercent
3  AutoRegenMP
4  AutoRegenMPPercent
5  AutoComeback
6  AutoRevival
7  HPItemRegen
8  MPItemRegen
9  IsNgaMy
10 NgaMyPercent
11 PhatQuangPhoChieu
12 ThanhTamPhoThienChu
13 KimChamDoKiep
14 CaiTuHoanSinh
```

Defaults: HP/MP thresholds 50%, medicine IDs `-1`, automatic toggles off.

Important: mobile Lua symbolic Nga My naming has a conflict with independently verified PC Config skill identities; do not infer actual skill name from symbolic variable alone.

## PET — 16 fields

```text
IsAutoCallPet | PetIDSelect | BloodSacrifice | BloodSacrificeValue |
Dedication | DedicationValue | AutoSkillPet | AutoCallBackPet |
CallBackPetNumber | AutoEat | HpPercent | AutoInjoy | InjoyValue |
AttackModel | IsAutoCallSprit | SpritIDSelect
```

## AUTOPK — 6 fields

```text
AutoPkAgian | IsLowHpTarget | IsFactionTarget | FactionID | SkillPK | UsingCombo
```

## FUBEN — mobile 3.5 layout

```text
1 AcTac
2 AcTacMapID
3 AcBa
4 LauLanTamBao
5 TranLongKyCuoc
6 ThuyLao
7 TrungAc
8 MainQuest
```

This is materially different from the newer PC schema and is another direct revision marker.

## Tool design recommendation

The EXE should maintain a **separate tool-owned per-LD profile**. Reading/importing built-in AutoSettings can be useful for default Train/recovery preferences, but external orchestration settings (vendor, SellThreshold, chat schedule, healer, state-machine retries) should not be forced into this legacy string.

When the tool intentionally changes built-in settings, update the live stock service through its semantic configuration path rather than manually editing a serialized string unless that exact write path has been proven safe.
