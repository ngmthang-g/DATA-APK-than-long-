# Context pack — Build Auto HP/MP Recovery / item survival

Read:

1. `analysis/20_ITEM_ACTIONS_AND_AUTO_RECOVERY.md`
2. `features/AUTO_RECOVERY.md`
3. `database/ITEM_ACTIONS_MOBILE.csv`
4. `analysis/23_STOCK_SOURCE_DEFECTS_TO_AVOID.md`
5. `database/AUTO_TOOL_ACTION_CATALOG.md`
6. `analysis/19_LD9_ACTION_ORCHESTRATION.md`

Implementation target:

```text
fresh HP/MP + bag snapshot
 -> threshold guard
 -> choose configured medicine template
 -> resolve current live DBItemData.ID
 -> one 100005 / Use=3 action
 -> HP/item/cooldown proof
 -> fresh snapshot
```

Configured survival medicine must be reserved from Auto Sell/Drop/Destroy candidate lists.
