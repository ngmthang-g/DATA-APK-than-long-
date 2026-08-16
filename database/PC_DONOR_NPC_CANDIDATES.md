# PC-donor NPC service candidates for mobile runtime verification

Status: **PC-DONOR ONLY until current mobile runtime verifies identity/service**

Purpose: reduce blind NPC discovery on LD9 by seeding likely stable IDs from the separately analyzed PC frozen Config database. These rows are **not** silently promoted to mobile facts.

---

# Lâu Lan — MapID 5 donor candidates

## Auto Sell candidate 1 — Ba Nhĩ

```text
PC MapID   = 5
PC NpcID   = 328
Name       = Ba Nhĩ
ResName    = ALaBoNanRen2
PC identity evidence = VERIFIED STATIC
Service evidence in PC KB = USER-REPORTED selling destination
```

Mobile verification procedure:

```text
GetMapNPCName(5,328)
GetNPCPosition(328)
 -> if consistent, route/interact
 -> require current normal NPCShop 200034 / IsGuildShop=false
 -> one safe sell proof before marking MOBILE_RUNTIME_VERIFIED_VENDOR
```

## Auto Sell candidate 2 — Mã Kiêu Minh

```text
PC MapID   = 5
PC NpcID   = 373
Name       = Mã Kiêu Minh
ResName    = ZhanDouXiaoYaoDiZi
PC identity evidence = VERIFIED STATIC
Service evidence in PC KB = USER-REPORTED selling destination
```

Do not infer merchant service from `ResName`; runtime shop proof is required.

## Auto Sell candidate 3 — Hiệp Hàng

```text
PC MapID   = 5
PC NpcID   = 341
Name       = Hiệp Hàng
ResName    = npcXiYuTuoDuiShangRen
PC identity evidence = VERIFIED STATIC
Service evidence in PC KB = PROBABLE merchant-archetype candidate
```

Good discovery fallback, not a verified mobile sell service.

## Auto Sell candidate 4 — Chu Thập Tam

```text
PC MapID   = 5
PC NpcID   = 398
Name       = Chu Thập Tam
ResName    = TieJiang
PC identity evidence = VERIFIED STATIC
Service evidence in PC KB = probable blacksmith/service candidate
```

Do not assume a blacksmith exposes a normal sell-capable NPCShop.

---

# Lâu Lan healer donor — Đỗ Thanh Đằng

```text
PC MapID = 5
PC NpcID = 339
Name     = Đỗ Thanh Đằng
ResName  = LangZhong1
```

PC frozen Config identity makes this a strong healer/doctor candidate. Mobile promotion requires:

```text
GetMapNPCName(5,339)
GetNPCPosition(339)
 -> live route/interact
 -> current GameDialog copied
 -> actual treatment/service text identified
 -> exact live selectionID used
 -> HP/result proof
```

Do not call it a `MOBILE VERIFIED HEALER` from name/ResName alone.

---

# Candidate testing order for the user's current Lâu Lan use case

Sell:

```text
328 Ba Nhĩ
373 Mã Kiêu Minh
341 Hiệp Hàng
398 Chu Thập Tam
```

Heal:

```text
339 Đỗ Thanh Đằng
```

This order is an evidence-collection priority, not a claim about route distance or current mobile service availability.

---

# Promotion record

When one candidate is verified on LD9, store a separate mobile runtime fact containing:

```text
APK/client/resource version
MapID
NpcID
returned current Name
current grid/world position
observed service type
GameDialog match text when relevant
normal NPCShop proof when relevant
verified timestamp/session
```

Do not overwrite the PC donor label; record the independent mobile runtime evidence alongside it.

Canonical verification design: `analysis/30_NPC_DISCOVERY_IDENTITY_RUNTIME.md`.
