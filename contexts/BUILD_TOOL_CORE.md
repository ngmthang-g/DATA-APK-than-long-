# Context Pack — Build Tool Core / Multi-LD EXE

## Read first

- `AI_BOOTSTRAP.md`
- `AUTO_TOOL_SCOPE.md`
- `analysis/25_ANDROID_MANIFEST_PROCESS_BINDING.md`
- `analysis/05_LD9_HOST_GUEST_ARCHITECTURE.md`
- `analysis/18_RUNTIME_ROLE_BAG_SNAPSHOT.md`
- `analysis/19_LD9_ACTION_ORCHESTRATION.md`
- `database/EXE_PER_LD_PROFILE_SCHEMA.md`

## Goal

Build the Windows host that discovers LDPlayer instances, creates one BotSession per game client and communicates with guest-side semantic scanner/action bridge.

## Static mobile binding facts

```text
package = com.fgstudio.thanlongmobile
main activity = com.unity3d.player.UnityPlayerActivity
```

No custom process attribute was observed on the main app/activity. Scope every Android PID lookup to the corresponding ADB serial because multiple emulators run the same package name independently.

## Required invariants

- one logical BotSession per LD profile / current Android game PID;
- independent settings and action gate;
- guest returns semantic values, not raw pointers for host caching;
- process/map/world generation invalidates guest caches;
- Android PID is reacquired after process restart;
- no action without proof/timeout contract;
- no current item instance/shop/dialog IDs shared across sessions.

## Config ownership

Use `database/EXE_PER_LD_PROFILE_SCHEMA.md`. Do not overload the game's legacy `AutoSettings=3.5` string with external orchestration settings.

Built-in AutoSettings may be imported/read for donor preferences only where useful.

## Do not read unless needed

Inventory/network deep docs are unnecessary for basic instance manager/scanner plumbing. Route feature-specific code through `AI_ROUTER.md`.
