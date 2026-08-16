# Mobile Auto Database

This directory is for compact lookup, not narrative reverse-engineering.

Use:

- `AUTO_TOOL_API_CATALOG.md` — semantic state/query/action names relevant to automation;
- `AUTO_TOOL_ACTION_CATALOG.md` — mutable action readiness and exact-proof requirements;
- `RUNTIME_SNAPSHOT_SCHEMA.md` — host-facing per-LD immutable state contract;
- `APK_SYMBOLS.csv` — current frozen APK symbol-name evidence;
- `PC_MOBILE_SYMBOL_CROSSWALK.csv` — PC donor ↔ mobile evidence status;
- `FACTS.jsonl` — atomic machine-readable facts;
- `FINDING_TO_DOC_MAP.md` — route from finding to canonical detail.

Do not treat a row with evidence `PC-DONOR` as mobile production truth.
