# Windows EXE ↔ LDPlayer 9 Host/Guest Architecture

## Product requirement

The user operates **one Windows EXE**, while every game client is an APK inside a separate LDPlayer 9 instance.

This differs fundamentally from the PC version where a Windows process can directly share the same OS/architecture environment with `GameAssembly.dll`.

## Correct split

```text
ThanLongAuto.exe (Windows host)
  |
  +-- LD Instance Manager
  |    - discover LD index/window/PID
  |    - map ADB serial/endpoint
  |    - start/pause selected BotSessions
  |
  +-- BotSession #1 ---------------------+
  +-- BotSession #2 ------------------+  |
  +-- BotSession #N ---------------+  |  |
                                    |  |  |
                              guest channel
                                    |  |  |
                             Android ARM64 bridge
                                    |
                            game process / IL2CPP
```

## Why a guest bridge exists

A Windows x64 EXE cannot directly call an ARM64 function pointer inside an Android guest as if it were a local DLL. The semantic resolver/action code that touches `libil2cpp.so` must execute in the guest/game environment or through a guest-capable instrumentation channel.

Host remains the product UI/orchestrator; bridge is an implementation component.

## Per-session state

```text
LdIndex
AdbSerial/Endpoint
EmulatorWindowPid
GamePid
SessionGeneration
CharacterIdentity
ResolverState
Snapshot
TrainProfile
SellProfile
FeatureState
ActionGate
LastAction
LastProof
Retry/timeout counters
```

## Isolation rule

Never share between sessions:

- raw pointers/Il2CppObject addresses;
- MainThread instance/delegate;
- selected target object;
- live item instance IDs selected for mutation;
- current shop/dialog object;
- pending action completion state.

Shared read-only static config may exist only when build/snapshot identity matches.

## Host ↔ guest message model

Prefer semantic request/response messages, e.g.:

```text
READ_SNAPSHOT
MOVE_TO(map,x,y)
SELECT_TARGET(roleId)
USE_SKILL(skillId,targetId)
REVIVE(type/payload after verified)
OPEN_NPC(npcId)
SELL_ITEM(instanceId,... after verified)
```

Guest should return structured proof/state, not raw pointers for host to keep.

## Action arbitration

Host orchestrator may observe many sessions concurrently, but each guest/game PID has one mutable action gate.

Example priority:

```text
manual pause/security condition
 > fatal recovery
 > revive
 > map transition/return
 > current sell transaction
 > train combat
 > background metrics
```

## Failure domains

If one LD9 crashes/disconnects, invalidate only that BotSession and rediscover its game process. Other sessions continue independently.
