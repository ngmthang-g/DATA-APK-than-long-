# IL2CPP Runtime & Metadata — Android ARM64

## Verified artifact model

The APK is Unity IL2CPP for `arm64-v8a`.

High-value pair:

```text
lib/arm64-v8a/libil2cpp.so
assets/bin/Data/Managed/Metadata/global-metadata.dat
```

Metadata header:

```text
sanity  0xFAB11BAF
version 0x00000027 = 39
```

## Why this matters

Game logic compiled from `Assembly-CSharp.dll` is native ARM64, but IL2CPP retains enough metadata/runtime APIs to recover semantic namespace/class/method/field relationships.

The correct mobile resolver should prefer:

```text
il2cpp_domain_get
 -> il2cpp_domain_get_assemblies
 -> il2cpp_assembly_get_image
 -> il2cpp_class_from_name(namespace,class)
 -> il2cpp_class_get_method_from_name / iterate methods
 -> il2cpp_class_get_field_from_name / iterate fields
```

Then cache handles only within the current game process/world generation.

## Verified dynamic exports

Current `libil2cpp.so` exports standard APIs including:

```text
il2cpp_assembly_get_image
il2cpp_class_from_name
il2cpp_class_get_field_from_name
il2cpp_class_get_method_from_name
il2cpp_class_get_methods
il2cpp_class_get_fields
il2cpp_domain_get
il2cpp_domain_get_assemblies
```

The broader export table also includes runtime/type/object helpers. Resolve them by export, not hardcoded offset where possible.

## PC difference

PC research uses Windows x64 `GameAssembly.dll`; mobile uses Android ARM64 ELF `libil2cpp.so`.

Therefore reusable knowledge:

```text
namespace
class name
method name/signature
field semantic
packet semantic
state-machine behavior
```

Non-reusable without revalidation:

```text
RVA
absolute address
x64 calling sequence
Windows injection mechanism
pointer lifetime assumptions across builds
```

## Version resilience

After a future APK update:

1. fingerprint APK and metadata;
2. confirm metadata version;
3. re-resolve semantic classes/methods;
4. run read-only smoke test;
5. only then re-enable mutable actions.

## Call safety boundary

`il2cpp_runtime_invoke` existing is not permission to invoke arbitrary Unity methods from any thread. Object validity, managed exception handling, method ownership and Unity/MainThread constraints still apply.

Production design separates resolver/read-only scanning from one gated mutable action path.
