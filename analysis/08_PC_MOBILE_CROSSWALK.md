# PC ↔ Mobile Crosswalk

PC knowledge base: `ngmthang-g/clinent-game-than-long-DATA-2222`

Mobile knowledge base: `ngmthang-g/DATA-APK-than-long-`

## Shared semantic evidence

Both research lines expose a very similar FGStudio Unity/IL2CPP vocabulary:

```text
LuaSystemAPI_Game
LuaSystemManager / Lua network bridge
MainThread
RoleData
map/position/path concepts
NPC interaction
bag/item semantic APIs
nearby target/combat APIs
TCP packet layer
```

This strongly supports using PC analysis as a **search map** for mobile.

## Reuse matrix

| Knowledge type | Reuse from PC? | Rule |
|---|---|---|
| namespace/class/method names | Yes, as search hints | confirm in mobile metadata/runtime |
| gameplay state-machine concepts | Yes | adapt to LD9 host/guest boundary |
| item identity distinctions | Yes as semantic model | verify mobile runtime fields/enums |
| packet names | Yes as search hints | confirm mobile producer |
| packet numeric IDs/payloads | No automatic reuse | mobile trace/static producer required |
| Config/NPC IDs | Conditional | mobile assets/server version may differ |
| x64 RVA/offset | No | ARM64 ELF is different |
| Windows injection/action bridge | No | mobile requires guest-side bridge |
| MainThread architecture | Strong donor | live mobile proof still required |

## High-value PC donor facts to test first

PC KB has exact solved equivalents for:

- Train semantic start/stop;
- bag site/item instance semantics;
- normal shop sell packet and payload;
- normal revive packet/type;
- NPC path helper;
- MainThread producer donors;
- runtime snapshot/action-proof architecture.

These facts drastically narrow mobile research but are recorded here as `PC-DONOR` until confirmed.

## Current notable match

Mobile metadata explicitly contains many of the same semantic methods used by PC research, including:

```text
GetNearByPeacePlayers
GetNearbySpritesWithPredicate
SelectedTarget
GetSkillCooldown
GetBuffs
GetFreeBagSpace
GetItemsAtSite
GetNPCPosition
ClickNPC
MoveTo / GoTo
SelectTarget / ChaseTarget
RequestUsingSkillWithTarget
```

This is a stronger relationship than merely “same game UI”.

## Important mismatch already observed

Current exact-string pass did **not** find standalone mobile `AutoFight_Main` or `StartAutoFight`, although PC KB uses these in its built-in Train flow.

Interpretations include asset/Lua relocation/encryption, naming differences, stripping, or different mobile UI layer. Do not claim equivalence until recovered.

Likewise mobile contains `CMD_REVIVE` but current pass did not find exact standalone `CMD_REVIVE_DATA` string used in PC docs.

## Practical research rule

When mobile needs an exact action:

```text
search PC semantic flow
 -> identify likely class/method/packet producer names
 -> confirm names in mobile
 -> trace/recover mobile producer
 -> record mobile exact fact
```

Never reverse the mobile client from zero if the PC KB already tells us where to look.
