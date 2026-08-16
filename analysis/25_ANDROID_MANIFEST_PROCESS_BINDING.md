# Android manifest / process binding for LDPlayer sessions

Status: **VERIFIED STATIC from binary AndroidManifest.xml**

## Main package

```text
package = com.fgstudio.thanlongmobile
```

This matches the package identity seen elsewhere in the APK snapshot.

## Main Unity activity

```text
com.unity3d.player.UnityPlayerActivity
```

Manifest attributes include:

```text
exported = true
enabled = true
launchMode = 2
screenOrientation = 11
resizeableActivity = true
```

No custom `android:process` attribute was observed on the application or main Unity activity. Under normal Android process rules, the game therefore runs in the default application process named for the package unless runtime/vendor behavior changes it.

## SDK/runtime packaging facts

```text
minSdkVersion = 25
targetSdkVersion = 36
compileSdkVersion = 36
extractNativeLibs = true
```

`extractNativeLibs=true` is relevant because ARM64 native libraries are expected to be available to the installed app/runtime in extracted native-lib form rather than requiring the host tool to operate on APK-compressed library bytes.

## Per-LD binding recommendation

The Windows EXE should treat ADB serial as the Android namespace boundary:

```text
LD instance
 -> its ADB serial
 -> package com.fgstudio.thanlongmobile
 -> current Android game PID
 -> guest bridge/session generation
```

Do not identify the Android game process only from the Windows emulator PID or window title.

After game restart/reconnect/crash:

1. retain the logical LD profile;
2. reacquire current Android PID for the package inside that ADB instance;
3. create a new ResolverGeneration/WorldGeneration;
4. discard all guest pointers/script objects from the old PID.

## Multi-instance consequence

Two emulators may each run a process with the same package name. They are still different processes in different emulator/ADB namespaces. All PID lookups and guest commands must be scoped to the correct serial/session.
