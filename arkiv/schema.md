# Arkiv Schema — Bhav

This file is a **required submission artefact**. Keep it updated any time an entity type, attribute, expiry, or query changes.

**Network:** Tiramisu testnet (chain ID 7738577)  
**SDK:** `@arkiv-network/sdk` 0.8.x  
**Last updated:** 2026-10-13

---

## Entity Types

> To be filled in as entity types are designed and deployed.

| Entity type name | Description | Creator | Owner |
|---|---|---|---|
| *(TBD)* | | | |

---

## Attributes

> For each entity type list every attribute that goes into the typed-attributes map (i.e., anything the app filters or ranges on). Anything that is only read, not queried, goes in the payload and is noted here for reference only.

### `PriceReport` (draft)

| Attribute | Type | Queryable | Notes |
|---|---|---|---|
| `crop` | string | yes — equality | Crop name (e.g. "wheat", "maize") |
| `market` | string | yes — equality | Mandi/market name |
| `pricePerQuintal` | number | yes — range | INR per quintal |
| `reportedAt` | number | yes — range | Unix timestamp (ms) of when reporter posted |
| `grade` | string | yes — equality | Crop grade (e.g. "A", "FAQ") |

**Payload (not queryable):**
- `notes` — free-text notes from the reporter (optional)
- `sourceLabel` — reporter's self-described source (trader, visual, etc.)

### `Confirmation` (draft)

| Attribute | Type | Queryable | Notes |
|---|---|---|---|
| `targetEntityId` | string | yes — equality | ID of the PriceReport being confirmed/disputed |
| `confirmedAt` | number | yes — range | Unix timestamp (ms) |
| `verdict` | string | yes — equality | "confirm" or "dispute" |

**Payload:**
- `comment` — optional short note

---

## Expiration per Type

> Entity Expiration controls how long data lives on the network. Use Lifetime Extension (`extendEntity`) to renew an entity; it sets a new expiry and reverts if the new date is not later than the current one.

| Entity type | Default lifetime | Rationale | Lifetime Extension trigger |
|---|---|---|---|
| `PriceReport` | 24 hours | Mandi prices are intraday; stale prices mislead buyers | Confirmations by others extend by +12 h (keeps actively-confirmed prices alive) |
| `Confirmation` | 48 hours | Confirmations should outlive the price to provide evidence | No extension |

---

## Queries the App Runs

> List every query the app makes. No `orderBy` (not supported server-side); all sorting is client-side.

| # | Purpose | Query shape | Sort |
|---|---|---|---|
| 1 | List latest price reports for a crop+market | `select().where({ crop: "wheat", market: "Azadpur" }).limit(50).fetch()` | Client-sort by `reportedAt` desc |
| 2 | All reports by a reporter | `select().createdBy(reporterAddress).limit(200).fetch()` | Client-sort by `reportedAt` desc |
| 3 | Confirmations for a specific report | `select().where({ targetEntityId: "<id>", verdict: "confirm" }).limit(200).fetch()` | Client-sort by `confirmedAt` desc |
| 4 | Live feed (all new entities of a type) | `watchEntityEvents` over WebSocket transport (no `fromBlock`) | Real-time append |

---

*This file is a living document. Add rows when entity types or queries change; never delete historical types (mark as "retired" instead).*
