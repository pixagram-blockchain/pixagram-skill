---
name: pixagram
description: Use when interacting with the Pixagram blockchain — a Hive fork. Triggers on tasks that mention Pixagram, PIXA / PXS / VESTS tokens, the api.pixagram.com or pixagram.dev endpoints, hived/Hivemind on this fork, or any account/post/witness operation on this chain. Tells the agent which endpoint to hit, which token names to use, which fields/accounts are renamed, and that the standard Hive API surface applies.
---

# Pixagram

A fork of [Hive](https://hive.io). The standard Hive RPC surface (`condenser_api.*`, `database_api.*`, `bridge.*`, `follow_api.*`, `tags_api.*`) works — only the renames below differ.

## Endpoints

| Network | RPC |
|---|---|
| Mainnet | https://api.pixagram.com |
| Testnet | https://pixagram.dev |

Both serve full Hive-compatible JSON-RPC. `bridge.*`, `follow_api.*`, `tags_api.*` and select `condenser_api.*` social methods are routed to a **Hivemind** indexer behind the same endpoint.

## Differences from Hive

### Tokens & addresses
| | |
|---|---|
| Liquid native | **PIXA** (replaces HIVE) |
| Stable | **PXS** (replaces HBD) |
| Staked | **VESTS** (same as Hive) |
| Public-key prefix | **`PIX`** (e.g. `PIX6LLegb…`) |
| Chain ID | derived from ASCII string `"pixagram"` (padded to 32 bytes) |

When building legacy-format asset payloads, send `PIXA` / `PXS` as the on-wire symbol bytes (not `STEEM` / `SBD` that older Hive clients hardcode). HF26 NAI-format assets work as-is.

### API field renames
Jussi rewrites these in responses (and accepts the new names in requests):

| Hive | Pixagram |
|---|---|
| `hbd_balance` | `pxs_balance` |
| `hbd_exchange_rate` | `pxs_exchange_rate` |
| `hbd_interest_rate` | `pxs_interest_rate` |
| `reward_hive` / `reward_hbd` | `reward_pixa` / `reward_pxs` |
| `total_vesting_fund_hive` | `total_vesting_fund_pixa` |
| `current_hbd_supply` | `current_pxs_supply` |
| `dhf_interval_ledger` | `dpf_interval_ledger` |

Symbols inside response strings (`"1.000 HBD"` etc.) are also normalized to PIXA/PXS.

### Governance
- **DPF** (Decentralized Proposal Fund) replaces Hive's DHF. Same mechanics — funded by ~15% of per-block inflation.
- Treasury account is `pixa.omnibus` (replaces `steem.dao` / `hive.fund`). Holds PXS only.

## Init / system accounts

These exist at genesis and you'll encounter them in any query:

| Account | What it is |
|---|---|
| `initminer` | Sole witness on pre-mainnet; receives producer rewards as VESTS |
| `pixa.ico` | ICO pool — 50,000,000 VESTS at genesis |
| `pixa.team` | Team vested allocation — 25,000,000 VESTS at genesis |
| `pixa.omnibus` | Treasury (DPF). Holds 250,000 PXS at genesis. Spendable only via approved proposals. |
| `miners`, `null`, `temp`, `steem` | Hive-inherited system placeholders. Empty key_auths → unspendable. `steem` exists only as a legacy recovery_account fallback target. |

## Quick examples

```bash
# Global chain state
curl -s -X POST https://api.pixagram.com -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"condenser_api.get_dynamic_global_properties","params":[],"id":1}'

# Account (note: pxs_balance not hbd_balance)
curl -s -X POST https://api.pixagram.com -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"condenser_api.get_accounts","params":[["initminer"]],"id":1}'

# Treasury balance
curl -s -X POST https://api.pixagram.com -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"condenser_api.get_accounts","params":[["pixa.omnibus"]],"id":1}'

# Median price feed
curl -s -X POST https://api.pixagram.com -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"condenser_api.get_current_median_history_price","params":[],"id":1}'

# Social feed (Hivemind)
curl -s -X POST https://api.pixagram.com -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"bridge.get_ranked_posts","params":{"sort":"trending","tag":"","limit":20},"id":1}'
```

For anything else, refer to [Hive developer docs](https://developers.hive.io/) — methods and ops are identical.
