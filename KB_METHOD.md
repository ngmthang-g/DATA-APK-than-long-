# Knowledge Base Method

## Goal

Store compact, reusable engineering facts so future AI/tool versions can answer exact questions without repeating broad reverse engineering.

## Canonical layers

```text
research/VERIFIED*      direct evidence snapshots
analysis/               detailed subsystem reasoning/evidence
features/               production behavior contracts
database/               compact lookup facts/actions/APIs
contexts/               minimal reading packs for implementation tasks
AI_INDEX / AI_ROUTER    navigation
```

## Fact record requirements

A useful fact should identify:

- platform/snapshot;
- evidence level;
- symbol/value/payload if exact;
- why it matters to automation;
- source/provenance;
- remaining boundary if any.

## Evidence discipline

Example of acceptable statement:

> `GetFreeBagSpace` string is present in current mobile metadata — VERIFIED static name. Runtime object acquisition and call behavior still require live proof.

Unacceptable upgrade:

> Mobile sell packet is 200036 because PC uses 200036.

Correct representation:

> PC uses 200036 — PC-DONOR. Search mobile producer/trace for same value, then promote only if confirmed.

## Update discipline

Whenever a runtime test settles a queued proof:

1. update `research/VERIFIED_APK_SNAPSHOT.md` if it changes snapshot-level truth;
2. update affected `analysis/*.md`;
3. update affected `features/*.md`;
4. update `database/AUTO_TOOL_ACTION_CATALOG.md` or API catalog;
5. append/replace relevant atomic fact in `database/FACTS.jsonl`;
6. update `AUTO_FEATURE_READINESS.md` and proof queue.

## Prefer semantic truth

If game exposes HP/map/item/NPC/path/action state semantically, do not replace it with OCR/pixel/coordinate assumptions.

## Mutation proof rule

Every production mutable action should be documented as:

```text
pre-state
 -> guard
 -> exactly one action
 -> expected observable state change
 -> timeout/failure handling
 -> fresh rescan
```
