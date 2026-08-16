# Context Pack — Build MainThread / Lua semantic bridge

## Required

- `analysis/01_IL2CPP_RUNTIME_METADATA.md`
- `analysis/07_MAIN_THREAD_DISPATCHER.md`
- `analysis/24_LUA_UI_ACTION_BRIDGE_BLUEPRINT.md`
- `analysis/05_LD9_HOST_GUEST_ARCHITECTURE.md`
- `research/AUTO_RUNTIME_PROOF_QUEUE.md`

## Goal

Prove a guest-side managed Action/delegate can be safely dispatched through game-owned MainThread semantics, then expose stable adapters to game-owned C#/Lua action surfaces.

## First proof

Use a harmless callback with no gameplay mutation.

Pass only when:

- correct delegate signature/lifetime;
- callback runs on expected Unity/game thread;
- repeated calls remain stable;
- a map transition/reconnect does not leave stale cached dispatcher/script state.

## Second proof — Lua/UI bridge

After MainThread proof, test one harmless live lookup through game-owned APIs such as:

```text
LuaSystemManager.HasScript / GetScript
LuaSystemAPI_GUI.FindUI
```

Do not begin by invoking Revive/Sell/Train.

Then prove the exact lifetime/reacquisition behavior of one UI script across open/close/map changes.

## Action adapter rule

- state/query: direct exact C# Game APIs when possible;
- one-packet semantic actions: game-owned Network.SendPacket path with stock guards;
- stateful shipped engine such as Train: invoke the loaded Lua service/method rather than replaying downstream packets.

## Hard boundary

Metadata proves method existence, not external ABI/closure semantics. Do not assume `ExecuteFunction(functionName,args)` accepts `object:method` syntax until a live reversible proof establishes it.
