# Auto Tool Action Catalog — Mobile

## Evidence legend

- `STATIC-NAME` — semantic action name exists in APK metadata.
- `TARGETED-PROOF` — exact producer/payload/runtime acceptance still required.
- `PC-DONOR` — PC equivalent solved; search hint only.

| Action | Mobile evidence | Required guard/proof |
|---|---|---|
| Move local player | `MoveTo`, `MoveToEx` STATIC-NAME | alive/map-ready; fresh position proof |
| Cross-map GoTo | `GoTo`, auto-path names STATIC-NAME | expected MapID + map-ready + position tolerance |
| Start/stop auto path | `StartAutoPath`, `StopAutoPath` STATIC-NAME | `IsAutoPathing`/destination/map proof |
| Click NPC | `ClickNPC` STATIC-NAME | correct map/proximity; actual dialog/shop state proof |
| Select target | `SelectTarget` STATIC-NAME | fresh selected target ID/state |
| Chase target | `ChaseTarget` STATIC-NAME | target alive/reachable + movement/target proof |
| Use skill | `UseSkill` / `RequestUsingSkill*` STATIC-NAME | target/range/cooldown/state proof |
| Normal revive | `CMD_REVIVE` concept STATIC-NAME; exact request TARGETED-PROOF | death state -> one request -> alive/map proof |
| Sell one item | inventory/NPC/network primitives present; exact request TARGETED-PROOF | valid current shop + current live item -> removal/update proof |
| Built-in Train start | PC-DONOR only for exact high-level entrypoint | recover mobile entrypoint or use low-level semantic loop |

## PC donor hints

PC KB currently documents exact packet/payloads for revive and sell. Do not place those numeric values in a mobile action implementation until mobile evidence promotes them.

## Mutation rule

All actions use:

```text
fresh state -> guard -> ONE action -> proof -> fresh state
```
