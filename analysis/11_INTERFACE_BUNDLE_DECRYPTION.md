# Phase 3 — Interface bundle decryption and Lua/UI recovery

Status: **VERIFIED STATIC EVIDENCE**

## Why this matters

The initial APK pass showed that important UI/gameplay actions were not all visible as dedicated C# methods. `libFGClientTool_Android.so` exports `FG_Encrypt` / `FG_Decrypt`, and the APK includes encrypted `Interface*.unity3d` bundles. Recovering this layer exposes the original Lua/UI action producers used by the mobile client.

## Native crypto evidence

The Android native module contains `FG_Encrypt`, `FG_Decrypt`, `FGStudio::FGCrypto::Encrypt`, `Decrypt`, `LegacyEncrypt`, and `LegacyDecrypt`.

For the current `Interface.unity3d`, the recovered legacy transformation produces a valid `UnityFS` bundle identifying Unity `6000.3.6f1`. Do not infer every bundle uses the same crypto branch; `Shared_2.unity3d` appears to require a different/current path.

## Recovered bundle facts

```text
Decrypted Interface.unity3d SHA256
dac818ce020d60ec04334fec5c91aa1884a534b81dc5620873a53c5ec07594db

Serialized payload SHA256
6eb2a292c52aba4e5ae6ff36a54e75ddb0b951623192943ebd79fd0af3428ffa

UnityFS format 8
Unity 6000.3.6f1
serialized version 22
platform 13
TypeTree enabled
class 49 TextAsset: 631 objects
class 142 AssetBundle: 1 object
```

The knowledge base deliberately stores derived facts and short proof excerpts rather than redistributing the entire proprietary Lua/UI corpus.

## High-value recovered TextAssets

```text
AutoFight_Main.lua
AutoFight.lua
Revival.lua
NPCShop.lua
NPCShop_SellItemTab.lua
NPCShop_BuyItemTab.lua
GameDialog.lua
ChatBox.lua
MiniChat.lua
Global_Constants.txt
Global_Functions.lua
TCPPacketDefine.txt
TCPCmdHandler.lua
TCPCmdEventHandler.lua
```

Representative hashes:

```text
AutoFight_Main.lua          bc963ead343785bf6f3497e9e02b206821014f4b2656bca53592046f5b8f8f7b
TCPPacketDefine.txt         40e65dbd2de776abda0a003dbc90f9e7b71b2d417c6f5c455aebe0df5d79d161
NPCShop_SellItemTab.lua    3352009d305c636e7b86631a32d836cb69604b6f6e0cb711501c132dd2d79b56
Revival.lua                8a8d3ac225a037191281b1d76a31d8d425e290afae63e67699d3d1896c581c35
ChatBox.lua                b91a34bee1da91c5c41eeb1720513eebad190ea9dbac160291a7d8190bee6adc
GameDialog.lua             e24211f056b5c56fdf8fb8dc4d7e1c1b84d2f2e6c6e117a8bf81c98120e16f42
```

## Research consequence

Preferred order for mobile automation questions is now:

```text
compact KB/database
 -> recovered Lua/UI semantic producer
 -> matching C#/IL2CPP API
 -> exact packet/payload where present
 -> runtime proof on LD9 only for remaining behavior/safety
```

For Train, Revive, Sell, GameDialog and Chat, broad binary reverse is no longer the first choice because exact client source semantics are now available.
