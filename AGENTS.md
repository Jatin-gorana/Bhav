# AGENTS.md — Bhav

Rules and context for AI coding agents (Claude, Cursor, Copilot, Codex…) contributing to this project.

---

## Project: Bhav

A live mandi (agricultural market) crop price board where:
- **Reporters** post prices as Arkiv entities.
- **Others** confirm or dispute posted prices (also as Arkiv entities).
- **Prices auto-expire** via Arkiv Entity Expiration — stale data vanishes on its own.
- **The frontend reads Arkiv directly from RPC.** There is no backend of ours — no server, no proxy, no database we operate. The Arkiv network IS the backend.

---

## Chain / Network

| Property | Value |
|---|---|
| Network | Tiramisu testnet |
| Chain ID | `7738577` |
| HTTP RPC | `https://rpc.tiramisu.db-chain.testnet.arkiv.network` |
| WebSocket | `wss://rpc.tiramisu.db-chain.testnet.arkiv.network` |
| SDK | `@arkiv-network/sdk` 0.8.x |
| Docs | https://docs.arkiv.network/networks/tiramisu/ |

---

## SDK Rules (non-negotiable)

> **Never assume the SDK API from memory or older docs.** Older docs and older context mention Kaolin and v0.6 — those are a different network and version. Always verify against the **installed package's TypeScript type definitions** (`.d.ts` files under `node_modules/@arkiv-network/sdk`) and the Tiramisu docs linked above.

- **Queries:** `client.select().where(...).limit(...).fetch()`, plus `ownedBy()`, `createdBy()`, `cursor()`. No server-side ordering (`orderBy` does not exist); sort results on the client, or range-filter on `$createdAt`. Max 200 results per page.
- **Ranges only on numeric attributes.** Store timestamps, prices and scores as numbers.
- **The payload is not queryable.** Anything you filter on (crop, market, reporter address, price, timestamp) goes in typed attributes, not the payload.
- **Everything expires.** Every entity type must have a defined expiry. Use `extendEntity` (called "Lifetime Extension") to renew — it *sets* a new expiry; it reverts if the new date is not later than the current one.
- **Trust comes from `$creator`** (immutable, on-chain, unforgeable). `$owner` is mutable and controls writes. Show `$creator` in the UI as the reporter address.
- **Live updates:** `watchEntityEvents` opens a real WebSocket only with a WebSocket transport and **no `fromBlock`** argument; over HTTP it silently falls back to polling. Always connect via `wss://` for live UI.
- **All Arkiv entities are publicly readable.** Privacy means encrypting or hashing the payload. There is no private-entity toggle.
- **Vocabulary:** say "Entity Expiration" (not TTL), "Lifetime Extension" (not renewal/renew/extend), "Arkiv entity" (not record/row/item), "Web3 database" for Arkiv.

---

## Security & Data Rules

- **Never store private keys, secrets or real personal data in Arkiv.** Use non-personal or synthetic data only. Hashes of emails or phone numbers are also forbidden (they can be reversed).
- **Never commit `.env` files or private keys.** Use environment variables loaded at runtime.
- **Never rewrite git history** (no force-push, no rebase of pushed commits, no `git commit --amend` after push). The hackathon judges record a commit SHA at the deadline; history rewrites disqualify the entry.

---

## Logging and Documentation Obligations

### Friction log (`/arkiv/friction.md`)
Every Arkiv problem or surprise — error, unexpected behaviour, missing feature, confusing doc — gets logged **immediately** in `/arkiv/friction.md` with:
- **Surface** (SDK, RPC, Docs, Explorer, Faucet, etc.)
- **Expected** (what you thought would happen)
- **Actual** (what happened)
- **Versions** (SDK version, Node version, browser/env)
- **Repro steps** (copy-pasteable)
- **Suggested fix** (your best guess at the root cause or fix)

A friction report that only says "everything worked great" scores 0. Log tested paths that worked with the same rigour.

### Schema (`/arkiv/schema.md`)
Keep `/arkiv/schema.md` updated **whenever any entity type, attribute, expiry or query changes.** The rubric checks the file; stale schema = lost points.

---

## Git Discipline

- Prefer small, focused commits with clear messages (conventional commits preferred: `feat:`, `fix:`, `chore:`, `docs:`).
- Do not bundle unrelated changes in a single commit.
- Do not push until intentional. The first push sets the origin baseline.

---

## Questions to resolve before coding a new entity type

1. What does this entity represent, and who is its `$creator`?
2. What attributes does the app filter on? (Put those in typed attributes, not the payload.)
3. What is the correct Entity Expiration for this type, and when should Lifetime Extension be applied?
4. What queries does the app need? (Write them in `/arkiv/schema.md` first.)
5. Is any payload field sensitive? (If so, encrypt or omit it; never store personal data.)
