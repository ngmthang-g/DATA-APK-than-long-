# Packaged resource boundary — what the base APK does and does not contain

Status: **VERIFIED STATIC CLASSIFICATION**

## Interface companion bundles

The same recovered legacy native decrypt path produces valid UnityFS for:

```text
Interface/Shared.unity3d
Interface/LoadingResources.unity3d
Interface/Logo.unity3d
```

All identify Unity `6000.3.4f1` and unpack cleanly.

Object-class inspection shows these companion bundles are graphics/shared-resource packages rather than hidden gameplay Config/Lua databases:

```text
Shared:           Sprite-heavy + Texture2D + AssetBundle
LoadingResources: Sprite-heavy + Texture2D + AssetBundle
Logo:             Sprite + Texture2D + AssetBundle
```

No TextAsset corpus comparable to `Interface.unity3d` was found in those three bundles.

`Shared_2.unity3d` does not decrypt with the same recovered legacy branch and should not be treated as solved. Do not spend time on it unless a concrete automation gap requires its contents.

## `data.unity3d`

Base `assets/bin/Data/data.unity3d` is an ordinary UnityFS bundle. It contains engine/player resources such as:

```text
globalgamemanagers
globalgamemanagers.assets
sharedassets0.assets
resources.assets
level0
```

The resources expose runtime/MonoScript type names (`LuaSystemAPI_Game`, `GScene_NPC`, `TCPGameEventProcessor_ItemPack`, `DBItemData`, etc.) and graphics/fonts/materials, but current inspection does **not** reveal the large gameplay Config tables (NPC rows, full Items/Equips/Monsters/Maps) that were recoverable from the PC frozen client data repository.

## Automation consequence

For this APK snapshot:

```text
semantic action logic -> recovered Interface Lua + IL2CPP metadata
static NPC/item/map identity tables -> PC donor or later downloaded/runtime resource proof
live truth -> LD9 runtime state
```

Do not repeatedly search the three solved graphics bundles for vendor/healer Config rows.

## Evidence boundary

The absence claim is scoped to the inspected **base APK packaged resource bundles**. The running game may download/update additional resource/config bundles after installation. Those are not present in this uploaded APK and must not be invented from static APK analysis.
