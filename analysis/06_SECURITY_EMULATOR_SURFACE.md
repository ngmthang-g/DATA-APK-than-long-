# Security / Emulator / ADB Surface

## Verified native exports

Current `libFGClientTool_Android.so` exports:

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
```

Metadata also contains `SecurityReport` and `LoginData` names.

## What this proves

The client has code capable of observing/reporting emulator-related, ADB, accessibility, overlay and input/security signals.

## What this does NOT prove

Static presence does not prove:

- a specific threshold causes a ban;
- LDPlayer 9 is rejected;
- ADB is always reported to server;
- any proposed bridge is detected;
- a given export is called during normal login in this snapshot.

Do not convert observability into enforcement claims without runtime evidence.

## Engineering consequence

Do not make Android Accessibility/overlay clicker services the core automation architecture when semantic IL2CPP state/action APIs are available.

Keep bridge minimal, isolate write actions, log stability, and test read-only first.

## Safety boundary

This KB is for understanding stability/compatibility and avoiding accidental crashes/diss. It does not document bypasses of security, emulator checks or anti-cheat controls.

If a security/captcha/manual-review condition becomes visible to the tool, production orchestration should pause/surface it rather than attempt automatic bypass.
