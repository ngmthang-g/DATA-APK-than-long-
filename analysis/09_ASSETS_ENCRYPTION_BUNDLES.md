# Assets / Encryption / Bundles

## APK bundle surface

The frozen APK includes:

```text
assets/Interface.unity3d
assets/Interface/LoadingResources.unity3d
assets/Interface/Logo.unity3d
assets/Interface/Shared.unity3d
assets/Interface/Shared_2.unity3d
assets/bin/Data/data.unity3d
```

## Crypto/loading surface

Metadata contains exact names:

```text
AssetBundleUtils
LoadWithCrypto
LoadWithCryptoAsync
```

`libFGClientTool_Android.so` exports:

```text
FG_Decrypt
FG_Encrypt
```

## Why this matters

PC research gained major value from decoded Lua/UI/config assets. Mobile may also place high-level UI/shop/auto scripts inside encrypted or updated bundles even when the native metadata exposes the underlying Game API.

## Research order

Do **not** start by decrypting every bundle.

Use:

```text
metadata semantic API
 -> runtime producer trace
 -> targeted asset/Lua extraction only if action remains unresolved
```

Good targeted examples:

- exact mobile high-level Train UI/Lua entrypoint;
- exact vendor dialog/shop handler if network trace cannot identify producer cleanly;
- mobile Config enum/ID rows needed for keep/sell policy.

## Update caveat

`Version.xml` points to FGStudio CDN/update services. Runtime assets may therefore differ from the APK's embedded bundle snapshot. Any extracted static asset must be marked with source/version and should not be assumed to represent live-updated content without verification.
